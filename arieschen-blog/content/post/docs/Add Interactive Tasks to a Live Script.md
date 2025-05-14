---

title: "Add Interactive Tasks to a Live Script in MATLAB"
description: ""
tags: [
    "MATLAB",
]
date: "2022-09-15"
categories: [
    "Robotics & Perception",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3

---

> _Live Editor tasks are simple point-and-click interfaces that can be added to a live script to perform a specific set of operations. You can add tasks to live scripts to explore parameters and automatically generate code. Use tasks to reduce development time, errors, and time spent plotting._

We are familiar with Jupyter notebook in Python. In MATLAB, there are **live scripts**. MATLAB live scripts and live functions are interactive documents that combine MATLAB code with formatted text, equations, and images in a single environment called the Live Editor. In addition, live scripts store and display output alongside the code that creates it.

The live scripts are better than Jupyter notebook in some ways. Here is an example.

## Add Interactive Tasks to a Live Script

We use the “Reduce data dimensionality using PCA “ task as an example.

![](https://miro.medium.com/v2/resize:fit:1400/1*J89vhBroxRRzQ_WuzJU9lA.png)

<div style="text-align: center;">PCA task output</div>

> _Principal component analysis (PCA) is a popular technique for analyzing large datasets containing a high number of dimensions/features per observation, increasing the interpretability of data while preserving the maximum amount of information, and enabling the visualization of multidimensional data._

First, we must create a live script. Then load the data of HVSR, which is a microtremor method.

![](https://miro.medium.com/v2/resize:fit:292/1*LFG3urvTS5YFVd3PcPyWIw.jpeg)

<div style="text-align: center;">Create a new live script</div>

Next, we add the PCA task from the menu list “Task”.

![](https://miro.medium.com/v2/resize:fit:1364/1*2FhPABpJqib5MUiNYAqlFg.jpeg)

<div style="text-align: center;">Add the PCA task</div>

The PCA task is as below, which contains the **Select data**, **Specity dimensionality reduction criterion** , and **Display results**. All functions in an interactive interface. You can also see the source code by clicking the small triangle.

![](https://miro.medium.com/v2/resize:fit:1400/1*Lc22UUXxa5pG-HAqT2j8gA.jpeg)

<div style="text-align: center;">PCA task</div>

Finally, we can set the output style. The outputs can display on the right or inline.

![](https://miro.medium.com/v2/resize:fit:1400/1*XEogSEyQzIUNM9-FUP5x7Q.jpeg)

<div style="text-align: center;">Output style</div>

In summary, the task in the live script is very convenient. Hope this helps you!