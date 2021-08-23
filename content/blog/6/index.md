---
title: "Chapter 5: Birds of a Feather"
date: "2021-08-23T14:24:32.169Z"
description: So GSoC 2021 has officially ended and I can say without a doubt that what a journey it was. I recently concluded with my GSoC project, the final PR got merged and I'm quite satisfied with the outcome.

---
<p align="center">
<img src="Radis.png" alt="Radis.png" width="200" /><br>
</p>

So `GSoC 2021` has officially ended and I can say without a doubt that what a journey it was. I recently concluded with my GSoC project, the final PR got merged and I'm quite satisfied with the outcome. 

Earlier we were able to find the time complexity of **LBL>Voigt**, **DIT>Voigt** and **DIT>FFT** (Formely known as LDM>FFT). On a small test replacing `np.convolve` with `scipy.signal.oaconvolve`, we were able to achieve 2 to 30 times performance boost. So we re-ran the benchmarks and were able to confirm this fact. 
You can see the result at [Benchmark Visualization GSoC 2021](https://anandxkumar.github.io/Benchmark_Visualization_GSoC_2021/).

The above results proved that `DIT>Voigt` performs better than `DIT>FFT` in almost every case. So we decided to use `DIT>Voigt` as the default setting in `Radis`. 

A **predict_time()** function was added, which computes the predicted time for **LBL>Voigt**, **DIT>Voigt** and **DIT>FFT** using the derived time complexity, and on `verbose>=2` shows the user the predicted time.

Also we Bifurcated `broadening_max_width` into 2 parameters:<br>
•  **Truncation:** Used in truncation of Voigt method.<br>
•  **neighbour_lines:** Increases Spectral range<br>

So now users have a lot of flexibility. Based on Physics, the default value of **truncation** is set to **50cm-1** and the default value of **neighbour_lines** is set to **0 cm-1**. Apart from this, some minor improvements were done in the `Profiler class` such as an improved algorithm is used to store data and now calculation time gets appended to the same key rather than overwriting it, which useful when we use `chunksize` or DIT optimization for `Non_equilibrium` conditions. 

So overall the code has been optimized and a user can expect a performance boost upto **40x** in worst scenarios. 

You can find all my work during the GSoC period [here](https://github.com/radis/radis/projects/5).

It was a great experience contributing to **Radis** and I definitely have learned alot along the way. And a big thanks to the great mentors at Radis especially [Erwan Pannier](https://github.com/erwanp) who guided me at every stage of the program. The road doesn't end here as I will stick around the organisation and will always find ways to contribute to Radis. One last thanks to **GSoC** for providing such a wonderful opportunity.




<p align="center">
Till we meet again, keep <b>Swinging for the fences.</b>
<br>
<img src="spidermanMM_traversal.gif" alt="spidermanMM_traversal.gif" width="500" /><br>
</p>