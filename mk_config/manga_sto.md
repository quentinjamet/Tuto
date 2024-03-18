# Manche-Gascogne CROCO config

![Alt text](https://github.com/quentinjamet/Tuto/blob/main/Figure/comp_domain_MANGA_CMEMS-IBI.png "a title")

Provides the different steps to setup a CROCO config (here MANGA) on Datarmor. Some adjustments are required for other HPC centers.

The (gitlab) CROCO documentation is here (https://croco-ocean.gitlabpages.inria.fr/croco_doc/tutos/tutos.01.download.croco.html), but some other infos can also be found there (https://www.croco-ocean.org/).

The practice I follow:
  * install and compile CROCO on ```$DATAWORK/CROCO_Shom/```
  * download CMEMES/Meteo France products on ```$SCRATCH```, then move generated forcing files on ```$DATAWORK/CROCO_Shom/data_in/``` (require some free space -- 1TB availalbe ...)
  * run CROCO on ```$SCRATCH``` and move outputs on ```$DATAWORK/CROCO_Shom/RUNS/```

## Install CROCO

  * Git clone (on your ```$DATAWORK/CROCO_Shom/```) the last stable release CROCO projet from the Gitlab of Inria: 
  ```
  cd $DATAWORK
  mkdir CROCO_Shom
  cd ./CROCO_Shom/
  git clone https://gitlab.inria.fr/croco-ocean/croco.git 
  ```

  * In case the clone does not work on frontal, first log on a ftp node
  ```
  qsub -I -q ftp -l mem=32GB -l walltime=06:00:00
  ```
  then clone the project

  * For developments, you will need an Inria gitlab access to contribute to the projet, and you should clone the project using ssh instead ()
  ```
  git clone git@gitlab.inria.fr:croco-ocean/croco.git
  ```

## Create the config tree
  * adjust and run ```create_config.bash```

## Compile the code
Need to update the following files:

  * ```jobcomp```: 
	* change ```FC=gfortran``` to ```FC=ifort```, and source additional librairies ```source /home2/datawork/qjamet/bashrc.intel```

  * ```cppdef.h```: 
	* define a CPP key ```MANGA``` in ```ifdef REGIONAL```
	* ```# define USE_CALENDAR```
	* ```# define NC4PAR```
	* ```# define NO_LAND``` -> deal with dry MPI proc (see ./croco/MPI_NOLAND for details)
	* ```# define WET_DRY``` -> deal with drying grid cell with tides/high pressure
	* ```# undef LMD_MIXING```
	* ```# define  GLS_MIXING``` -> change vertical mixing scheme from KPP to GLS
	* ```# define PSOURCE``` -> deal with river runoff 

  * ```param.h```: define the size of your domain 

Then compile the code:  ```./jobcomp```


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
