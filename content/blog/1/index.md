---
title: "Chapter 0: Prologue"
date: "2021-06-06T22:40:32.169Z"
description: Hi There and Namaste! This is going to be the second blog and first blog related to GSoC where I will be sharing my experience Community Bonding Period Experience with Radis. 
---
Hi There and Namaste! This is going to be the second blog and first blog related to GSoC where I will be sharing my experience Community Bonding Period Experience with <b>Radis</b>. Before moving ahead lets learn about GSoC and my perspective about it.

## Google Summer of Code
GSoC or the way I like to say it (Great Summer Opportunity to Code ;) ) is a program conducted and funded by Google to promote college students around the world to engage with Open Source Community and contribute to the organization for a tenure of 3 months. In the process, code is created and released for the world to see and use. But the main aim of GSoC is to promote students to stick to the organizations and help to grow the Open Source Community. This is a great initiative by Google that brings thousands of students every year and help them get an opportunity to peek into the world of open source development, learn new skills and also get compensated for the work, quite generously.

I remember during second year of my college, it was around end of March and my roommate was applying for GSoC and I was like what is this program? There I got to know about it but since the deadline was near I was afraid of doing all the stuffs in a week of time, so I didn't apply for it. Fast forwarding to next year, I was prepared enough this time and I feel priviledged to be a part of GSoC as part of OpenAstronomy. 

## My GSoC Project 
I'm part of <b>[Radis](https://github.com/radis/radis)</b> organization which is a sub-org of [OpenAstronomy](https://github.com/OpenAstronomy). Radis is a fast line-by-line code used to synthesize high-resolution infrared molecular
spectra and a post-processing library to analyze spectral lines. It can synthesize absorption
and emission spectrum for multiple molecular species at both equilibrium and
non-equilibrium conditions.<br>
Radis computes every spectral line (absorption/emission) from the molecule considering
the effect of parameters like Temperature, Pressure. Due to these parameters, we don't get
a discrete line but rather a shape with a width. This is called line broadening and for any spectral synthesis code, this is the bottleneck step.<br>

Ok let us C what my GSoC project is all about! <br>
Radis has 2 methods to calculate the lineshape of lines.<br>
● Legacy Method<br>
● DLM Method<br>

The goal of this project is to derive an equation comprising all parameters that affect the
performance for calculating Voigt broadening by running several benchmarks for different
parameters involved in the calculation of lineshapes to check their significance in
computation time. Then we need to find the critical value for the derived equation (`Rc`)
which will tell us which optimization technique to select based on the computed `R` value in
<b>calc_spectrum()</b>. An `optimization = "auto"` will be added that will choose the best method based on the parameters provided.

## Community Bonding Period
The first phase of GSoC is the <b>Community Bonding Period</b> which is a 3 weeks long period. It's main aim is allow the student to get familiar with the community and the codebase. It serves as a warm-up period before the coding period. The first thing I did was that I went though the original Radis [paper](doi.org/10.1016/j.jqsrt.2018.09.027) and also the DLM implementation [paper](doi:10.1016/j.jqsrt.2020.107476) because our project objective is based on these 2 implementations. It helped me understand the main purpose of RADIS, its architecture and the science behind different steps of both equilibrium and non-equilibrium spectrum, though I have to accept these papers are way too technical for me :p (Complex Spectroscopy related formulas).<br> I believed inorder to get myself ready for the coding period, I shall focus on solving some related issues to make me more familiar with the codebase.<br>

In order to compute any spectrum we need to determine several parameters like - minimum-maximum wavenumber, molecule, Temperature of gas, mole fraction, wstep, etc.<br>
`wstep` determines the wavenumber grid’s resolution. Smaller the value, higher the resolution and vice-versa. By default radis uses `wstep=0.01`. You can manually set the wstep value in <b>calc_spectrum()</b> and SpectrumFactory. To get more accurate result you can further reduce the value, and to increase the performance you can increase the value.

Based on wstep, it will determine the number of gridpoints per linewidth. To make sure that there are enough gridpoints, Radis will raise an Accuracy Warning. If number of gridpoints are less than `GRIDPOINTS_PER_LINEWIDTH_WARN_THRESHOLD` and raises an Accuracy Error if number of gridpoints are less than `GRIDPOINTS_PER_LINEWIDTH_ERROR_THRESHOLD`.

So inorder to select the optimum value of `wstep`