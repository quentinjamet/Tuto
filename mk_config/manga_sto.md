# Manche-Gascogne CROCO config
<p align="center">
  <img src="https://github.com/quentinjamet/Tuto/blob/main/Figure/comp_domain_MANGA_CMEMS-IBI.png?raw=true" alt="Sublime's custom image"/>
</p>

Provides the different steps to setup a CROCO config (here MANGA) on Datarmor. Some adjustments are required for other HPC centers.

The (gitlab) CROCO documentation is here (https://croco-ocean.gitlabpages.inria.fr/croco_doc/tutos/tutos.01.download.croco.html), but some other infos can also be found there (https://www.croco-ocean.org/).

The practice I follow:
  * install and compile CROCO on ```${DATAWORK}/CROCO/``` with the following tree:
	* ```${DATAWORK}/CROCO/croco/CONFIGS/MY_CONF_NAME/```: where the code is compiled
	* ```${DATAWORK}/CROCO/data_in/```: all model input files
	* ```${DATAWORK}/CROCO/runs/```: model outputs
  * download CMEMES/Meteo France products and run the model on ```$SCRATCH```, then move generated forcing files in the appropriate storage (i.e. ```$DATAWORK/CROCO/data_in/``` and ```$DATAWORK/CROCO/runs/```, respectively). There might be a question of where to store these large dataset to be addressed at some point, only 1TB available on ```$DATAWORK```.
  * If needed, force a bash environment with a ```~/.login```:
  ```
  if (-x /bin/bash) then
    exec /bin/bash -l
  endif

  ```

<!--- /////////////////////////////////////////// -->
## Install CROCO
<!--- /////////////////////////////////////////// -->

  * Git clone CROCO projet from the Gitlab of Inria (assuming you have an Inria GitLab account and you have setup ssh keys ; see https://docs.gitlab.com/16.8/ee/user/ssh.html -- **for devs**): 
  ```
  cd $DATAWORK
  mkdir CROCO
  cd ./CROCO/
  git clone git@gitlab.inria.fr:croco-ocean/croco.git
  ```

  * In case the clone does not work on frontal, first log on a ftp node
  ```
  qsub -I -q ftp -l mem=32GB -l walltime=06:00:00
  ```
  then clone the project.

  * In case you do not have an gitlab account, use https protocol instead (but you will not be able to ```git push``` changes to the project -- **for use-only**):
  ```
  git clone https://gitlab.inria.fr/croco-ocean/croco.git
  ```

  * Creat a local branch for your configuration
  ```
  git checkout -b my_branch
  ```


<!--- /////////////////////////////////////////// -->
## Create the config tree
<!--- /////////////////////////////////////////// -->

  * Adjust and run ```create_config.bash```, in particular ```MY_CONFIG_NAME```
  * Creat other directories:
  ```
  cd ${DATAWORK}/CROCO/
  mkdir runs 
  mkdir -p data_in/atm  data_in/grd  data_in/ini  data_in/obcs  data_in/runoff  data_in/tide
  ```





<!--- /////////////////////////////////////////// -->
## Compile the code (with ```gfortran```)
<!--- /////////////////////////////////////////// -->

Move to your configuration directory 
  ```
  cd ./your_config_name/
  ```

To compile on sevseral nodes using PBS, use the script ```Compile.pbs```:
  ```
  cp ${DATAWORK}/CROCO/croco/SCRIPTS/Compile.pbs .
  ln -sf ${DATAWORK}/CROCO/croco/SCRIPTS/croco_env_datarmor_gnu_hdf5_netcdf.sh croco_env.sh
  ```
where ```croco_env.sh``` is used to load proper modules and librairies, in particular NetCDF I/O with parallel capabilities (through CPP key ```NC4PAR```). IMPORTANT: This should be also used at run time to insure identical environment between compiling and running CROCO.

Before submitting it, few adjustments are required. First, add 16 nodes to the make command at the very end of ```jobcomp```:
  ```
  $MAKE -j 16
  ```

With ```gfortran``` (default compiler in ```jobcomp```) there is an arithmetic overflow (i.e. memory issue) associated with ```max_buff_size``` in ```partit.F```. This file (in collaboration with ```ncjoin.F```) is used to split large input files into smaller files following the MPI decomposition. In recent version of CROCO the CPP key ```NC4PAR``` also deals with this issue, such that we will not rely on ```partit.F```. An easy fix is then to  set ```max_buff_size``` to an older value, i.e.:
  ```
  cp ${DATAWORK}/CROCO/croco/OCEAN/partit.F .
  sed -i 's/3000\*2000\*100/16384/g' partit.F
  ```

(The other option is to use ```ifort``` instead (```FC=ifort``` in ```jobcomp```), and here are the proper modules and librairies: ```source /home2/datawork/qjamet/bashrc.intel```)

Then, adjust the following files:
  * ```cppdef.h```: 
	* in basic options, replace ```BENGUELA_LR``` by ```YOUR_CONF_NAME``` 
	* ```# define  MPI```
	* ```# define USE_CALENDAR```
	* ```# define NC4PAR```       -> deal with parallel I/O
	* ```# define MPI_NOLAND```   -> deal with dry MPI proc (see ./croco/MPI_NOLAND for details)
	* ```# define WET_DRY```      -> deal with drying grid cell with tides/high pressure
	* ```# undef LMD_MIXING```
	* ```# define  GLS_MIXING```  -> change vertical mixing scheme from KPP to GLS-KEPSILON
	* ```# define PSOURCE``` -> deal with river runoff 

  * ```param.h```: define the size of your domain
	```
	...
	#elif defined REGIONAL
	# if defined  BENGUELA_LR
	    parameter (LLm0=41,   MMm0=42,   N=32)   ! BENGUELA_LR
	# elif defined  YOUR_CONF_NAME
	    parameter (LLm0=xxx,   MMm0=xxx,   N=xxx)   ! YOUR_CONF_NAME
	...
	``` 

To submit ```Compile.pbs``` to the PBS job scheduler:
  ```
  qsub Compile.pbs
  ```
and use ```qstat -u your_login``` to follow the job status. If the code compiled succesfully, you should have ```croco``` executable in your current directory. If this is not the case, errors will be reported in ```COMPILE.oXXXXXXX```.




<!--- /////////////////////////////////////////// -->
## Install python croco tools
<!--- /////////////////////////////////////////// -->

Mathieu Le Corre developped python tools to generate bathymetry, initial conditions and obcs and surface forcing files. To use them, first make a copy to your workdir:
  ```
  cd ${DATAWORK}
  cp -r /home2/datawork/qjamet/Python_tools_export/ .
  ```
The instructions in ```README.INSTALL``` are not up-to-date, especially on the use of conda on datarmor (see: https://domicile.ifremer.fr/intraric/Mon-informatique/Calcul-et-donnees-scientifiques/Datarmor-Calcul-et-Donnees/Datarmor-calcul-et-programmes/Pour-aller-plus-loin/,DanaInfo=w3z.ifremer.fr,SSL+Conda-sur-Datarmor). To use the latest version of conda, run (or even better, add in your ```~/.bashrc``` and source it)
  ```
  . /appli/anaconda/latest/etc/profile.d/conda.sh
  ```
and update your ```~/.condarc``` as:
  ```
  envs_dirs:
    - $DATAWORK/conda-env
    - /appli/conda-env
    - /appli/conda-env/2.7
    - /appli/conda-env/3.6
  pkgs_dirs:
    - $DATAWORK/conda/pkgs
  ```
(Other versions of conda are available here: ```/appli/anaconda/version/```).

Creating the python environmemnt will some take time (~1h) and it is safer to run on a dedicated ftp node:
  ```
  qsub -I -q ftp -l walltime=02:59:00 -l mem=32G
  ```

Switch from ```csh``` (default SHELL on datarmor) to ```bash```
  ```
  bash
  ```
and source the following file (assuming you are on ```${DATAWORK}/Python_tools_export/```):
  ```
  source bashrc.datarmor
  ```
Then, create the conda environment
  ```
  conda env create -f requirements.yaml
  ```
(If correctly installed, you should see it with ```conda info --envs```).




<!--- /////////////////////////////////////////// -->
## Install Copernicus API and download necessary data
<!--- /////////////////////////////////////////// -->

This will be done by creating an additional conda env, but (I think) there is no sensitivity to loaded modules. It is also much faster than for ```crocoenv``` and can be done directly on the frontal node.

Get the copernicus *.yml* file from here ```/home2/datawork/qjamet/Python_tools_export/copernicusmarine_env.yml``` (or from here: https://help.marine.copernicus.eu/en/articles/7970514-copernicus-marine-toolbox-installation), and create the environement:
  ```
  cd ${DATAWORK}/Python_tools_export/
  cp /home2/datawork/qjamet/Python_tools_export/copernicusmarine_env.yml .
  conda env create -f copernicusmarine_env.yml
  ```
(If correctly installed, you should see it (```cmt_1.0```) with ```conda info --envs```).

Download necessary data with ```download_glorys_data_copernicus_cli.sh```:
  * (I had to update ```command_line``` on line 90 to fit what was setup in ```copernicusmarine_env.yml```, i.e. ```"copernicus-marine subset"``` to ```"copernicusmarine subset"```)
  * (A better option is to first generate the grid (see below), then use it to extract the region of interest)
  * update the dates, the region of interest (or set ```READ_GRD=1``` and provide bathy file ```INPUT_GRD```), and the location where data will be downloaded (```OUTDIR```)
  * run it on a ftp node to avoid any troubles:
  ```
  qsub -I -q ftp -l walltime=02:59:00 -l mem=32G
  ```
  * (```conda activate mt_1.0; cd ${DATAWORK}/Python_tools_export/$```)
  * uncomment (an update with correct export names) the following lines and provide your Copernicus username and pw:
	* ```export COPERNICUSMARINE_CACHE_DIRECTORY=/tmp/${USER}```
	* ```export COPERNICUS_MARINE_SERVICE_USERNAME=xxx```
	* ```export COPERNICUS_MARINE_SERVICE_PASSWORD=xxx```
  * 09/04/2024: seems that the new version of ```copernicusmarine``` now requires an updqted nimenclature for ID products; check here: https://data.marine.copernicus.eu/product/GLOBAL_MULTIYEAR_PHY_001_030/services.
  * ```./download_glorys_data_copernicus_cli.sh```
  * (in case of troubles: https://help.marine.copernicus.eu/en/articles/8632322-copernicus-marine-toolbox-troubleshoots)



<!--- /////////////////////////////////////////// -->
## Prepare bathy, initial, obcs and foprcing files
<!--- /////////////////////////////////////////// -->

In case you run away at the end of the previous section and start again from a fresh shell (and mind), first things first:
  ```
  qsub -I -q ftp -l walltime=02:59:00 -l mem=32G
  cd ${DATAWORK}/Python_tools_export/
  . /appli/anaconda/latest/etc/profile.d/conda.sh #(if not included in your .bashrc)
  source bashrc.datarmor
  conda activate crocoenv
  ```
Then you should be fine for next steps ...

As their names indicate (or not), ```make_*.py``` are used to generate grid, initial conditions, and forcing files (atm, obcs, tides, runoff). We will follow them step by step (always use ```python3.9```):

  * ```make_grid.py```: 
	* generate bathy file
	* launch it  
	```
	python3.9 make_grid.py
	```
	* and follow the instructions (configure grid, apply smoothing and save)
	* move the grid file to ```data_in```
	```
	mv your_grid_name.nc ${DATAWORK}/CROCO/data_in/grd/
	```
	* Note that following CROCO standards, the *_rho points will be +2 grid points numbers (+1 at each side), and the *_u, *_v and *_psi will be +1 grid point numbers (+1 and east/north side). Bellow is an illustration:

<p align="center">
  <img src="https://github.com/quentinjamet/Tuto/blob/main/Figure/grid_croco_stag_points.png?raw=true" alt="Sublime's custom image"/>
</p>


  * ```make_ini.py```:
	* update ```input_dir```, ``Yini,Mini,Dini``` in agreement with what was provided in ```download_glorys_data_copernicus_cli.sh```
	* update ```sigma_params``` -> deals with vertical mesh, will be used to define obcs files. Should provide these variables also in ```make_bry.py``` and in ```croco.in``` to define S-coordinates.
	* ```python3.9 make_ini.py```
	* files are generated in ```croco_dir```

  * ```make_bry.py```:
	* define ```croco_dir``` and ```input_dir```
	* define ```sigma_params```
	* ```output_file_format``` -> monthy or yearly
	* ```Yorig``` is in Copernicus input file
	* starting and ending dates
	* ```python3.9 make_bry.py```

  * ```make_tides.py```
	* updated version from Alain Serpette (alain.serpette@shom.fr), using FES2014 tidal dataset, available on DATARMOR at ```/home/shom_simuref/USERS/nducouss/TIDES/FES_2014/```. This has required an update of ```Readers/tides_reader.py``` and ```Modules/tides_class.py``` (included in ```/home2/datawork/qjamet/Python_tools_export/```).
	* Update ```croco_dir```, ```croco_grd```, ```croco_filename``` and ```Yini,Mini,Dini```
