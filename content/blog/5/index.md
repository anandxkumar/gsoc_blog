---
title: "Chapter 4: The Other Side"
date: "2021-08-01T14:24:32.169Z"
description: A new month has started and I have started to see the light at the end of the tunnel. Good Morning and welcome back. Phase 2 has been rolling and let us look at the new findings.

---
A new month has started and I have started to see the light at the end of the tunnel. Good Morning and welcome back. Phase 2 has been rolling and let us look at the new findings.

Earlier the complexity of Legacy method was determined. The complexity of LDM Voigt and LDM FFT was to be determined using similar approach. Upon executing several benchmarks based on Number of lines, Spectum range, wstep, broadening max width. Previously it was thought the complexity was: <br>

```
time(LDM_fft) ~ c2*Nlines + c3*(N_G*N_L + 1)*N_v*log(N_v) (where N_v =  Spectral Points)
time(LDM_voigt) ~ c2*Nlines + c3'*(N_G*N_L + 1)*N_truncation*log(N_truncation) (where N_truncation = broadening width / wstep)
```
But in actual Re running all benchmarks for **LDM>Voigt** and **LDM>FFT** with a `broadening max width = 300 cm-1`. All benchmarks and visualizations can be found [here](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/) we were able to conclude the followings:<br>

**FFT:**<br>
•  Complexity doesn't depend on Nlines but rather wL x wG ; check this benchmark: [link](https://public.tableau.com/app/profile/anand.kumar4841/viz/LDMLinesvsCalculationTimeUpdatedCO2/Sheet1), it certainly looks like Complexity ∝ Nlines but its actually dependent on wL and wG, and gives same result on (wL x wG+ 1) x Spectral_Points x Log(Spectral_Points).<br>
•  Upon implementing multiple linear regression for **c1 x Nlines + c2 x (wL x wG+ 1)*Spectral_Points x Log(Spectral_Points)** gives `c1=2.65e-07`, `c2=4.48256e-08` but their `p value = 0.648 and 0.00001`, and `p>0.05` are insignificant, thus Nlines is insignificant for determining the complexity.<br>
•  Since FFT is independent of broadening max width; benchmark: [link](https://public.tableau.com/app/profile/anand.kumar4841/viz/LDMVoigtandFFTBMW_NEW/Sheet1), so on comparing it Spectral point gives us same same time. Thus Spectral Point =  (wavenum max - wavenum max)/wstep instead of (wavenum maxcalc - wavenum min calc)/wstep<br>
•  **Overall complexity =  4.48256897e-08 x (wL x wG+ 1) x Spectral_Points(without BMW) x Log(Spectral_Points(without BMW))**  [link](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/LDM/Complexity_FFT_Final/Complexity_FFT_Final.html) (with the help of multple linear regression using sklearn; is almost accurate)<br>

**Voigt:**<br>
•  Similar to 1st point of FFT.<br>
•  Upon doing multiple linear regression for **c1 x N lines + c2 x (wL x wG + 1) x SpectralPoints x BMW xLog(SpectralPoints x (BMW) )** gives `c1=-1.9392e-06, c2=1.28256e-09` but their `p value = 0.848 and 0.00001`, and `p>0.05` are insignificant, thus N_lines is insignificant for determining the complexity.<br>
•  Calculation time is dependent `Broadening_Max_width`, but upon inspections with Spectral Points, we have the exact same plot. So complexity is dependent only on Spectral Points but with broadening_max_width i.e. wavenum_calc, which causes the increase in computational time on increasing broadening_max_width.<br>
•  **Overall complexity = 5.26795e-07 * (wL x wG+ 1)*Spectral Points x Log(Spectral Points)** [link](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/LDM/Complexity_Voigt_Final/Complexity_Voigt_Final.html) (with the help of multple linear regression using sklearn; almost straight)<br>

**Also:** From all the above plots, it really clear if going with broadening_max_width=300cm-1 in wavespace, it will take alot more time than fft in all aspects.

But upon replacing `np.convolve` with `scipy.signal.oaconvolve`, we were able to achieve `2 to 30` times performance boost. So it will be interesting to re run benchmarks with the latest piece of code and see which method performs better. Also some benchmarks will be added to ASV benchmark too to see how its performance changes over time.

Also profiler was modified to a tree like a stucture using `OrderedDict` and `YAML` has been used to print the profiler in a proper structued way using **Spectrum.print\_perf\_profiler()** or **SpectrumFactory.print\_perf\_profiler()**.

**Example:**

```
s = calc_spectrum(1900, 2300,         # cm-1
                  molecule='CO',
                  isotope='1,2,3',
                  pressure=1.01325,   # bar
                  Tvib=1000,          # K
                  Trot=300,           # K
                  mole_fraction=0.1,
                  verbose=3,
                  )
s.print_perf_profile()
```

**Gives the following output:**

```
>>> spectrum_calculation:
>>>   applied_linestrength_cutoff: 0.0024361610412597656
>>>   calc_emission_integral: 0.006468772888183594
>>>   calc_hwhm: 0.006415128707885742
>>>   calc_line_broadening:
>>>     DLM_Distribute_lines: 0.0003898143768310547
>>>     DLM_Initialized_vectors: 9.775161743164062e-06
>>>     DLM_closest_matching_line: 0.0005028247833251953
>>>     DLM_convolve: 0.029767990112304688
>>>     precompute_DLM_lineshapes: 0.013132810592651367
>>>     value: 0.07619166374206543
>>>   calc_lineshift: 0.00074005126953125
>>>   calc_noneq_population:
>>>     part_function: 0.03405046463012695
>>>     population: 0.005669832229614258
>>>     value: 0.03983640670776367
>>>   calc_other_spectral_quan: 0.002928495407104492
>>>   calc_weight_trans: 0.008247852325439453
>>>   check_line_databank: 0.0002810955047607422
>>>   check_non_eq_param: 0.04109525680541992
>>>   fetch_energy_5: 0.014983654022216797
>>>   generate_spectrum_obj: 0.00032138824462890625
>>>   generate_wavenumber_arrays: 0.0010433197021484375
>>>   reinitialize:
>>>     copy_database: 2.1457672119140625e-06
>>>     memory_usage_warning: 0.0018389225006103516
>>>     reset_population: 2.6226043701171875e-05
>>>     value: 0.001964569091796875
>>>   scaled_non_eq_linestrength:
>>>     corrected_population_se: 0.002747774124145508
>>>     map_part_func: 0.0010590553283691406
>>>     value: 0.0038983821868896484
>>>   value: 0.1904621124267578
```

So at the end a productive week! Looking forward to conclude GSoC with a worthy ending :)