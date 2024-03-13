# Stochastic Manche-Gascogne CROCO config

![Alt text](https://github.com/quentinjamet/Tuto/blob/main/Figure/comp_domain_MANGA_CMEMS-IBI.png "a title")

## install CROCO

  * (with ssh) requires an access to inria gitlab 
  ```
  git clone git@gitlab.inria.fr:croco-ocean/croco.git
  ```

## Make the config
  * adjust and run ```create_config.bash```

## update couple of files:
  /* ```jobcomp```: update source, compilation and run directory (chnage gfortran to ifort? if done, need to source ifort bashrc from Xavier (/home2/datawork/qjamet/test/bashrc.intel))
  * ```param.h```: define the size of your domain 

  * ```cppdef.h```: 
	* define a CPP key ```MANGA_STO``` in ```ifdef REGIONAL```
	* ```# define USE_CALENDAR```
	* ```# define NC4PAR```
	* ```# define NO_LAND``` -> deal with dry MPI proc (see ./croco/MPI_NOLAND for details)
	* ```# define WET_DRY``` -> deal with drying grid cell with tides/high pressure
	* ```# define PSOURCE``` -> deal with river runoff 

## prepare initial, obcs and foprcing files 
