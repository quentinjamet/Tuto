# Manche-Gascogne CROCO config

![Alt text](https://github.com/quentinjamet/Tuto/blob/main/Figure/comp_domain_MANGA_CMEMS-IBI.png "a title")

Provides the different steps to setup a CROCO config (here MANGA) on Datarmor. Some adjustments are required for other HPC centers.

The (gitlab) CROCO documentation is here (https://croco-ocean.gitlabpages.inria.fr/croco_doc/tutos/tutos.01.download.croco.html), but some other infos can also be found there (https://www.croco-ocean.org/).

The practice I follow:
  * install and compile CROCO on ```$DATAWORK/CROCO/```
  * download CMEMES/Meteo France products on ```$SCRATCH```, then move generated forcing files on ```$DATAWORK/CROCO/data_in/``` (require some free space -- 1TB availalbe ...)
  * run CROCO on ```$SCRATCH``` and move outputs on ```$DATAWORK/CROCO/runs/```

## Install CROCO

  * Git clone CROCO projet from the Gitlab of Inria (assuming you have an Inria GitLab account and you have setup ssh keys ; see https://docs.gitlab.com/16.8/ee/user/ssh.html): 
  ```
  cd $DATAWORK
  mkdir CROCO_Shom
  cd ./CROCO_Shom/
  git clone git@gitlab.inria.fr:croco-ocean/croco.git
  ```

  * In case the clone does not work on frontal, first log on a ftp node
  ```
  qsub -I -q ftp -l mem=32GB -l walltime=06:00:00
  ```
  then clone the project.

  * In case you do not have an gitlab account, use https protocol instead:
  ```
  git clone https://gitlab.inria.fr/croco-ocean/croco.git
  ```

  * Creat a local branch for your configuration
  ```
  git checkout -b my_branch
  ```

## Create the config tree
  * Adjust and run ```create_config.bash```, in particular ```MY_CONFIG_NAME```
  * Then move to the configuration directory ```cd ./your_config_name/```

## Compile the code (with ```gfortran```)

This can be done on several nodes using PBS with the script ```Compile.pbs```:
  ```
  cp /home2/datawork/qjamet/CROCO/croco/SCRIPTS/Compile.pbs .
  cp /home2/datawork/qjamet/CROCO/croco/SCRIPTS/croco_env_qj.sh .
  ```
where ```croco_env_qj.sh``` is used to load proper modules and librairies.

Before submitting it, few adjustments are required. First, add 16 nodes to the make command at the very end of ```jobcomp```:
  ```
  $MAKE -j 16
  ```

Then, with ```gfortran``` there is an arithmetic overflow (i.e. memory issue) which needs to be fixed. This is done by setting the ```max_buff_size``` in ```partit.F``` to an older value, i.e.:
  ```
  cp ../OCEAN/partit.F .
  sed -i 's/3000\*2000\*100/16384/g' partit.F
  ```

(The other option is to use ```ifort``` instead (```FC=ifort``` in ```jobcomp```), and here are the proper modules and librairies: ```source /home2/datawork/qjamet/bashrc.intel```)

Then, adjust ```cppdef.h``` with the following changes: 
	* define a CPP key ```YOUR_CONF_NAME``` in ```ifdef REGIONAL```
	* ```# define  MPI```
	* ```# define USE_CALENDAR```
	* ```# define NC4PAR```
	* ```# define NO_LAND``` -> deal with dry MPI proc (see ./croco/MPI_NOLAND for details)
	* ```# define WET_DRY``` -> deal with drying grid cell with tides/high pressure
	* ```# undef LMD_MIXING```
	* ```# define  GLS_MIXING``` -> change vertical mixing scheme from KPP to GLS, and add
  ```
  # ifdef GLS_MIXING
  #   define GLS_GEN
  # endif
  ```
	* ```# define PSOURCE``` -> deal with river runoff 

  * ```param.h```: define the size of your domain
  ```
  ...
  #elif defined REGIONAL
  # if defined  BENGUELA_LR
      parameter (LLm0=41,   MMm0=42,   N=32)   ! BENGUELA_LR
  # elif defined  YOUR_CONF_NAME
      parameter (LLm0=798,   MMm0=739,   N=75)   ! YOUR_CONF_NAME
  ...
  ``` 

To submit ```Compile.pbs``` to the PBS job scheduler:
  ```
  qsub Compile.pbs
  ```
and use ```qstat -u your_login``` to follow the job status.


## Prepare bathy, initial, obcs and foprcing files 
For this, best option is to use Mathieur Le Corre's python scripts (here on datarmor: home2/datawork/qjamet/Python_tools_export/). Then use these different scripts (ctivate conda env before ```conda activate crocoenv```):

  * ```make_grid.py```: 
	* to generate bathy file
	* ```python3.9 make_grid.py```
  * ```download_glorys_data_copernicus_cli.sh```: 
	* to get CMEMS-GLORYS product
	* But first, install CMEMS API (see https://marine.copernicus.eu/news/access-data-opendap-erddap-api).
	* update the dates and region or provide bathy file ```INPUT_GRD```
  * ```make_ini.py```:
	* update ```Yini,Mini,Dini```
	* update ```sigma_params``` -> deals with vertical mesh, will be used to define obcs files. Should provide these variables also in ```make_bry.py``` and in ```croco.in``` to define S-coordinates.
	* ```python3.9 make_ini.py```
	* files are generated in ```croco_dir```
  * ```make_bry.py```:
	* define ```croco_dir``` and ```input_dir```
	* define ```sigma_params```
	* ```output_file_format``` -> monthy or yearly
	* starting and ending dates
	* ```python3.9 make_bry.py```
