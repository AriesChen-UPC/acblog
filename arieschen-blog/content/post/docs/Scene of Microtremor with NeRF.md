---

title: "Scene of Microtremor with NeRF"
description: ""
tags: [
    "NeRF",
    "AI",
    "Microtremor",
]
date: "2023-06-15"
categories: [
    "Computer Vision",
    "Image Processing",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 2

---

![](https://miro.medium.com/v2/resize:fit:1400/1*hDMkE8-tA4qDKhamoqqqpw.jpeg)

Scene of Microtremor

# Microtremor

<u>The microtremor method is a geophysical technique used to measure the natural vibrations of the ground, known as microtremors.</u> These vibrations are typically caused by the movement of the earth’s crust, such as earthquakes, and can be used to study the subsurface structure and properties of the earth.

![](https://miro.medium.com/v2/resize:fit:830/1*gpfsBT4o15NgRd8PWXnLPw.png)

   <div style="text-align: center;">Signal of Microtremor</div>

To measure microtremors, a seismograph or accelerometer is typically placed on the ground at a specific location. The device records the ground motion as a function of time and frequency, and this data can be used to calculate the spectral characteristics of the microtremors. These spectral characteristics can be used to infer the geotechnical properties of the subsurface, such as the shear wave velocity, which is an important parameter in the analysis of earthquake hazards and site response.

The microtremor method is a relatively simple and non-invasive technique that can be used to study a wide range of geotechnical and geological problems. It is often used in conjunction with other geophysical techniques, such as surface wave analysis, to provide a more complete understanding of the subsurface.

![](https://miro.medium.com/v2/resize:fit:882/1*i8rU35MtsWxzd4YSRfxlzQ.png)

   <div style="text-align: center;">Microtremor in the Airport</div>

## SPAC

<u>Spatial autocorrelation (SPAC), introduced by Aki , is one of the more reliable and more promising of the ambient vibration array techniques currently available.</u> The basic assumption of this technique is that surface waves resulting from ambient noise can be regarded as the sum of waves propagating in a horizontal plane in different directions with different powers, but with the same phase velocity for a given frequency. Therefore, the phase velocity of these waves can be computed by estimating the spatial-correlation function for a fixed frequency without knowledge of the directionality of the waves. By varying the frequency, the phase velocity can be determined as a function of frequency. To capture the surface waves from all directions, as originally proposed, the microtremor SPAC method for phase-velocity determination requires measurement of noise data using sensors placed in a circular array, with an additional one located in the center of the array. In many cases in urban areas, such a sensor layout is difficult to achieve. This constraint has hence led to a number of studies that have modified the original SPAC method.

![](https://miro.medium.com/v2/resize:fit:1224/1*IkvLTyjzqLIHYCrB5qeUjA.jpeg)

   <div style="text-align: center;">Dispersion curve from SPAC</div>

## HVSR

<u>Nakamura, (1989) developed the well-known Horizontal to Vertical Spectral Ratio (HVSR or H/V), which proved to be an extremely cost-effective and rapid tool for seismological investigations, and particularly well suited to study extensive areas (e.g. Nakamura 1989; Nakamura 2019).</u>

Recent introduction of new low-cost and portable sensors and advancement in numerical modeling and inversion techniques, largely popularized the HVSR method and improved our capability of investigating large areas and gain a better understanding of their seismic behaviour. Since its introduction, HVSR has been applied in an increasing number of studies worldwide, to determine the fundamental vibration frequency of the sedimentary cover. For example, in the studies of the Grenoble basin (Cornou et al. 2008), Thessaloniki basin (Raptakis and Makra 2010), Trabzon-Arsin Basin (Akin and Sayil 2016), the horizontal to vertical spectral ratio, as computed in the HVSR method, was leveraged to explore the seismic response of the sites in frequency domain and assess the resonance frequency(ies) of the sediments. The application of HVSR from ambient noises has also been employed in geotechnical applications to obtain 1D (i.e. layered) shear wave velocity profiles (Raptakis and Makra 2010).

![](https://miro.medium.com/v2/resize:fit:1242/1*dhCcAS8R7UZ0s_4vl81NBQ.png)

   <div style="text-align: center;">HVSR curve</div>

# Neural Radiance Fields (NeRF)

<u>A neural radiance field (NeRF) is a fully-connected neural network that can generate novel views of complex 3D scenes, based on a partial set of 2D images.</u> It is trained to use a rendering loss to reproduce input views of a scene. It works by taking input images representing a scene and interpolating between them to render one complete scene. NeRF is a highly effective way to generate images for synthetic data.

A NeRF network is trained to map directly from viewing direction and spatial location (5D input) to opacity and color (4D output), using volume rendering to render new views. NeRF is a computationally-intensive algorithm, and processing of complex scenes can take hours or days. However, new algorithms are available that dramatically improve performance.

![](https://miro.medium.com/v2/resize:fit:704/1*_986fTHFE-es7rb2ZJSW_A.png)

![](https://miro.medium.com/v2/resize:fit:1400/1*xdu4X53kiV5lnZxK09HodA.png)

   <div style="text-align: center;">NeRF</div>