# Activity 1 – Run an idealized ocean gyre 
Credits: Jonathan Gula (jonathan.gula@univ-brest.fr)

**Install and compile CROCO**
  * Open a terminal (e.g. ```Ctrl+Alt+T```)
  * Create a directory (e.g. ```mkdir ~/ModNum/```) for the project, then go there (e.g. ```cd ~/ModNum/```)
  * ```git clone``` the CROCO code from the official website:

   ```git clone https://gitlab.inria.fr/croco-ocean/croco.git```

  * You should see the following files (using ```ls ./croco/```), and the source code (i.e. the FORTRAN files ```*.F```) will be in the folder ```./croco/OCEAN/```
    
![Alt text](https://github.com/quentinjamet/Tuto/blob/main/Figure/CROCO_content.png "a title")

  * Create a folder where you will run the model
    
```mkdir ./case1``` or ```mkdir -p ```~/ModNum/case1```

  * We need to edit the following files: ```jobcomp```, ```cppdefs.h```, ```param.h```, ```croco.in``` so copy them into the folder you just created:
```
cp ~/ModNum/croco/OCEAN/jobcomp ~/ModNum/case1/
cp ~/ModNum/croco/OCEAN/cppdefs.h ~/ModNum/case1/
cp ~/ModNum/croco/OCEAN/param.h ~/ModNum/case1/
cp ~/ModNum/croco/TEST_CASES/croco.in.Basin ~/ModNum/case1/croco.in
```

