---
title: "Chapter 2: Survey Corps"
date: "2021-07-05T23:40:32.169Z"
description: So its been around 4 weeks of coding period, a lot of progress so far!
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

Several Spectrums were benchmarked against various parameters to see it's correlation and derive its complexity. We used Profiler class with [init_database()](https://radis.readthedocs.io/en/latest/source/radis.lbl.loader.html#radis.lbl.loader.DatabankLoader.init_database) which stores all parameters of Spectrum along the Profiler in a `csv` generated file; all spectrum info got added into the csv file  which could be used to do create visualizations to analyze the data. We used Xexplorer library and Tableau(a visual analytics platform) to create visualizations. A [github repository](https://github.com/anandxkumar/Benchmark_Visualization_GSoC_2021) was created to store the Visualization along the CSV data file of each benchmark.

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

Similar technique was used to benchmark LDM method. Now LDM uses 2 types of broadening method that are `voigt` and `fft`. `voigt` uses truncation for calculating spectrum  in wavenmber space where as `fft` uses 
## Digging in whiting_jit

Based on several benchmarks, it is estimated that around **70-80%** time is spent on calculating the broadening. The broadening part has the following hierarchy:<br>
<b>
```
_calc_broadening()
-> _calc_lineshape()
   -> _voigt_broadening()
      -> _voigt_lineshape()
         -> whiting_jit()
```
</b>

On close inspection we observed that **80-90%** time is spent on `whiting_jit` process. Going further down in `whiting_jit`, **60-80%** time is spent on **lineshape calculation.** Below is the formula:<br>
```
lineshape = (
    (1 - wl_wv) * exp(-2.772 * w_wv_2)
    + wl_wv * 1 / (1 + 4 * w_wv_2)
    # ... 2nd order correction
    + 0.016 * (1 - wl_wv) * wl_wv * (exp(-0.4 * w_wv_225) - 10 / (10 + w_wv_225))
)
```

The whole process can be divided into 4 parts:<br>
```
    part_1 =   (1 - wl_wv) * exp(-2.772 * w_wv_2)

    part_2 =    wl_wv * 1 / (1 + 4 * w_wv_2)

    # ... 2nd order correction
    part_3 =  0.016 * (1 - wl_wv) * wl_wv * exp(-0.4 * w_wv_225) 

    part_4 =  - 10 / (10 + w_wv_225)
```

The complexity of each part comes out: <br>
<b>
```
    o1 = broadening__max_width * n_lines / wstep

    O(part_1) = n_lines * o1
    O(part_2) = n_lines * 4 * o1
    O(part_3) = (n_lines)**2 * o1
    O(part_4) = o1 
```
</b>

Running several benchmark showed us that **part_3** takes the most time out of all steps. So clearly we can see that the complexity of Legacy method is not dependent on
Spectral Range but rather `Number of Calculated Lines`,`broadening__max_width` and `wstep`. It may seem that the complexity of Legacy method is:<br>

<p align="center"><b> n_lines^2 * broadening__max_width * n_lines / wstep</b></p> <br>

But inorder to prove this we need more benchmarks and evidence to verify this and it may involve normalization of all steps in lineshape calculation!<br> 

So the goal for the next 2 weeks is clear:<br> 
<b>i)</b> Refactor the entire codebase with Profiler.<br>
<b>ii)</b> Find the complexity of **Legacy Method** with the help of more benchmark and analysis.<br>
<b>iii)</b> Do the same for **LDM Method**!<br>

Ok I guess time's up! See you after 2 weeks :)