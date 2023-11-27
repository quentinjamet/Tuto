# Run an idealized ocean basin II
Credits: Jonathan Gula (jonathan.gula@univ-brest.fr)

**Barotropic vorticity equation**

  * We will diagnose the barotropic vorticity equation from the simulations. See
    * https://journals.ametsoc.org/view/journals/phoc/31/10/1520-0485_2001_031_2871_wwbcir_2.0.co_2.xml?tab_body=pdf
    * http://jgula.fr/ModNum/diagnostics_croco.pdf

  * Rerun the BASIN test case from Activity 1 with additional diagnostics
      * Add diagnostics in the ```cppdefs.h```
        ```
        # define DIAGNOSTICS_VRT
        ```
      * Modify the ```croco.in``` (you can take this one https://www.jgula.fr/ModNum/croco.in.Basin)
      * Re-compile and rerun the simulation

  * <ins>**QUESTIONS**</ins>:
      * Using your preferred language (python , matlab, julia, etc.) plot together the different terms of the barotropic vorticity budget averaged over the last 5 years of the simulation for the BASIN test case from Activity 1 [available in the ```basin_diags_vrt_avg.nc file```].
      * What is the first order balance over the interior of the gyres?
      * What about the western boundary current?
