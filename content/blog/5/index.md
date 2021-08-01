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
But in actual Re running all benchmarks for LDM>Voigt and LDM>Voigt with a `broadening max width = 300 cm-1`. All benchmarks and visualizations can be found [here](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/) we were able to conclude the followings:<br>

**FFT:**<br>
•  Complexity doesn't depend on Nlines but rather wL x wG ; check this benchmark: [link](https://public.tableau.com/app/profile/anand.kumar4841/viz/LDMLinesvsCalculationTimeUpdatedCO2/Sheet1), it certainly looks like Complexity ∝ Nlines but its actually dependent on wL and wG, and gives same result on (wL x wG+ 1) x Spectral_Points x Log(Spectral_Points).<br>
•  Upon implementing multiple linear regression for **c1 x Nlines + c2 x (wL x wG+ 1)*Spectral_Points x Log(Spectral_Points)** gives `c1=2.65e-07`, `c2=4.48256e-08` but their `p value = 0.648 and 0.00001`, and `p>0.05` are insignificant, thus Nlines is insignificant for determining the complexity.<br>
•  Since FFT is independent of broadening max width; benchmark: [link](https://public.tableau.com/app/profile/anand.kumar4841/viz/LDMVoigtandFFTBMW_NEW/Sheet1), so on comparing it Spectral point gives us same same time. Thus Spectral Point =  (wavenum max - wavenum max)/wstep instead of (wavenum maxcalc - wavenum min calc)/wstep<br>
•  **Overall complexity =  4.48256897e-08 x (wL x wG+ 1) x Spectral_Points(without BMW) x Log(Spectral_Points(without BMW))**  [link](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/LDM/Complexity_FFT_Final/Complexity_FFT_Final.html) (with the help of multple linear regression using sklearn; is almost accurate)<br>

**Voigt:**<br>
Similar to 1st point of FFT.
Upon doing multiple linear regression for c1*N_lines + c2*(wL*wG+ 1)*Spectral_Points*B_M_W*Log(Spectral_Points*(B_M_W) ) gives c1=-1.9392e-06, c2=1.28256e-09 but their p value = 0.848 and 0.00001, and p>0.05 are insignificant, thus N_lines is insignificant for determining the complexity.
Calculation time is dependent Broadening_Max_width, but upon inspections with Spectral Points: link, we have the exact same plot. So complexity is dependent only on Spectral Points but with broadening_max_width i.e. wavenum_calc, which causes the increase in computational time on increasing broadening_max_width.
Overall complexity = 5.26795e-07 * (wL*wG+ 1)*Spectral Points*Log(Spectral Points) [link] (with the help of multple linear regression using sklearn; almost straight)

Also: From all the above plots, it really clear if going with broadening_max_width=300cm-1 in wavespace, it will take alot more time than fft in all aspects.


