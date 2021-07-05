---
title: "Chapter 2: Survey Corps"
date: "2021-07-05T23:40:32.169Z"
description: So its been around 4 weeks into the coding period, a lot of insights and progress so far!
---
So its been around 4 weeks into the coding period, a lot of insights and progress so far!

## Profiler Class

The good news is that the Profiler class has been successfully implemented in the develop branch and will be available to users by version `0.9.30` .<br>
Link : [Profiler PR](https://github.com/radis/radis/pull/286)<br>

Below is a simple example how all steps are printed based on the verbose level:<br>
```
wmin = 2000
wmax = 3300
wstep = 0.01
T = 3000.0 #K
p = 0.1 #bar
broadening_max_width=10

sf = SpectrumFactory(wavenum_min=wmin, wavenum_max=wmax, 
                    pressure=p,
                    wstep=wstep,
                    broadening_max_width=broadening_max_width, 
                    molecule="CO",
                    cutoff=0, # 1e-27,
                    verbose=3,
                    )
sf.load_databank('HITEMP-CO')
s = sf.eq_spectrum(T)
```
Output:
```
... Scaling equilibrium linestrength
... 0.01s - Scaled equilibrium linestrength
... 0.00s - Calculated lineshift
... 0.00s - Calculate broadening HWHM
... Calculating line broadening (60869 lines: expect ~ 6.09s on 1 CPU)
...... 0.16s - Precomputed DLM lineshapes (30)
...... 0.00s - Initialized vectors
...... 0.00s - Get closest matching line & fraction
...... 0.02s - Distribute lines over DLM
...... 1.95s - Convolve and sum on spectral range
... 2.14s - Calculated line broadening
... 0.01s - Calculated other spectral quantities
... 2.21s - Spectrum calculated (before object generation)
... 0.01s - Generated Spectrum object
2.22s - Spectrum calculated
```
Also we can access these steps and the time taken by them using `Spectrum.get_conditions()['profiler']`. Also there is a parameter `SpectrumFactory.profiler.relative_time_percentage` that stores the percentage of time taken by each steps at a particular verbose level, helpful seeing the most expensive steps in Spectrum calculation.<br>


## Legacy Method Complexity

Several Spectrums were benchmarked against various parameters to see it's correlation and derive its complexity. We used Profiler class with [init_database()](https://radis.readthedocs.io/en/latest/source/radis.lbl.loader.html#radis.lbl.loader.DatabankLoader.init_database) which stores all parameters of Spectrum along the Profiler in a `csv` generated file; all spectrum info got added into the csv file  which could be used to do create visualizations to analyze the data. We used `Xexplorer` library and `Tableau`(a visual analytics platform) to create visualizations. A [github repository](https://github.com/anandxkumar/Benchmark_Visualization_GSoC_2021) was created to store the Visualization along the CSV data file of each benchmark.

Following are the inference of the benchmarks for Legacy Method:

<b>
•  Calculation Time ∝ Number of lines<br>
•  Calculation Time ∝ Broadening max width<br>
•  Calculation Time ∝ 1/wstep<br>
•  Calculation Time not dependent on Spectral Range<br>
</b><br>


So complexity of Legacy method can be derived as: <br>
 **`complexity = constant * Number of lines * Broadening Max Width / Wstep`** <br>


## LDM Method Complexity

Similar technique was used to benchmark LDM method. Now LDM uses 2 types of broadening method that are `voigt` and `fft`. `voigt` uses truncation for calculating spectrum  in wavenmber space where as `fft` calculates spectrum on entire spectral range in fourier space. So benchmarks were done on both methods to compare their performance against various parameters.

Spectrum were benchmarked against parameters like Spectral Range, Wstep, Spectral Points, Number of Lines and Broadening Max Width. Following are the inferences.

For `fft`:<br>
<b>
• Calculation Time ∝ Spectral Points<br>
• Calculation Time ∝ Number of Lines<br>
</b>

For `voigt`:<br>
<b>
• Calculation Time ∝ Spectral Points<br>
• Calculation Time ∝ Number of Lines<br>
• Calculation Time ∝ Broadening Max Width<br>
</b>

For LDM we are expecting the following complexity:<br>
 **`t_LDM_fft ~ c2*N_lines + c3*(N_G*N_L + 1)*N_v*log(N_v)`**<br>
 **`t_LDM_voigt ~ c2*N_lines + c3'*(N_G*N_L + 1)*N_truncation*log(N_truncation)`**<br>

 So the goal for the next 2 weeks will be to get the complexity of both `voigt` and `fft` method and see places for improving both methods and quite possibily create a `Hybrid` method taking the best of both worlds. 