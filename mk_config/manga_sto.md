# Stochastic Manche-Gascogne CROCO config

![Alt text](https://github.com/quentinjamet/Tuto/blob/main/Figure/comp_domain_MANGA_CMEMS-IBI.png "a title")

## Install CROCO

  * (with ssh) requires an access to inria gitlab 
  ```
  git clone git@gitlab.inria.fr:croco-ocean/croco.git
  ```

## Make the config
  * adjust and run ```create_config.bash```

## Update couple of files:
  /* ```jobcomp```: update source, compilation and run directory (chnage gfortran to ifort? if done, need to source ifort bashrc from Xavier (/home2/datawork/qjamet/test/bashrc.intel))
  * ```param.h```: define the size of your domain 

  * ```cppdef.h```: 
	* define a CPP key ```MANGA_STO``` in ```ifdef REGIONAL```
	* ```# define USE_CALENDAR```
	* ```# define NC4PAR```
	* ```# define NO_LAND``` -> deal with dry MPI proc (see ./croco/MPI_NOLAND for details)
	* ```# define WET_DRY``` -> deal with drying grid cell with tides/high pressure
	* ```# define PSOURCE``` -> deal with river runoff 

## Prepare bathy, initial, obcs and foprcing files 
For this, best option is to use Mathieur Le Corre's python scripts (here for me on datarmor: home2/datawork/qjamet/Python_tools_export/). Then use these different scripts to generate bathymetry.

  * ```make_grid.py```: to generate bathy file
	* ```python3.9 make_grid.py```
  * ```download_glorys_data_copernicus_cli.sh```: to get CMEMS-GLORYS product
	* But first, install CMEMS API (see https://marine.copernicus.eu/news/access-data-opendap-erddap-api).
	* update the dates and region or provide bathy file ```INPUT_GRD```.
