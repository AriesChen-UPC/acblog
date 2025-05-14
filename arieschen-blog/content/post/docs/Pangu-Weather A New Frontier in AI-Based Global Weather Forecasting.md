---
title: "Pangu-Weather: A New Frontier in AI-Based Global Weather Forecasting"
description: ""
tags: [
    "AI",
    "Transformers",
    "Weather",
]
date: "2024-04-25"
categories: [
    "AI Applications",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3
---

The advent of artificial intelligence (AI) in weather forecasting marks a pivotal shift in meteorological sciences, combining traditional numerical weather prediction (NWP) methods with advanced machine learning techniques. **Pangu-Weather**, a large AI model, enhances medium-range global weather forecasting through its innovative 3D neural network architecture, highlighting significant improvements over existing methods in accuracy and computational efficiency.

![](https://raw.githubusercontent.com/AriesChen-UPC/AriesChen-UPC/main/Blog/DALL%C2%B7E%202024-04-24%2014.56.24%20-%20A%20conceptual%20visualization%20of%20Pangu-Weather%2C%20an%20advanced%20AI%20model%20for%20weather%20forecasting%2C%20showcasing%20its%203D%20neural%20network%20architecture.%20The%20image%20de.webp)

### 1. Introduction

Historically, weather forecasting has relied on NWP, which, despite its precision, suffers from high computational demands. Recent advancements in AI have opened new avenues for enhancing forecast speed and efficiency. Pangu-Weather integrates these advancements, employing a three-dimensional deep network equipped with Earth-specific priors and a hierarchical temporal aggregation strategy, achieving unprecedented forecasting accuracy and speed.

<u>Pangu-Weather utilizes a 3D Earth-specific transformer (3DEST) architecture that conceptualizes weather data in three dimensions—longitude, latitude, and altitude. This structure allows for a better capture of the intricate dynamics of atmospheric variables across different layers of the atmosphere. The model's encoder-decoder framework, derived from the Swin Transformer, efficiently processes large datasets, significantly reducing computation time compared to traditional NWP models.</u>

### 2. Forecasting Performance

Trained on 39 years of comprehensive global weather data, Pangu-Weather leverages reanalysis data to simulate and predict weather conditions accurately. Various data augmentation techniques were employed to enhance the model's generalizability and prevent overfitting, ensuring robust performance across diverse atmospheric phenomena.

Pangu-Weather has demonstrated superior performance in deterministic forecasts, especially in tracking tropical cyclones and predicting extreme weather events. Its ability to deliver high-accuracy forecasts significantly faster than the best available NWP systems marks a substantial advancement in meteorological forecasting.

#### 2.1 Data Prepare

It's crucial to gather the right datasets. Here's a brief guide to help you get started:

1. **Near-Surface Data (2D)**:
   This dataset is essential for capturing surface-level atmospheric phenomena. It includes four key variables:
   - **Mean Sea Level Pressure (MSLP)**: This measures the atmospheric pressure adjusted to sea level, crucial for understanding weather patterns.
   - **10-meter U-Wind Component (U10)**: Represents the east-west component of wind speed at 10 meters above the ground.
   - **10-meter V-Wind Component (V10)**: Captures the north-south wind component at the same elevation.
   - **2-meter Temperature (T2M)**: This is the air temperature recorded at two meters above the ground, giving insights into ground-level thermal conditions.

2. **Upper-Air Data (3D)**:
   For a comprehensive analysis of the atmospheric profile, you'll need data from multiple altitudes. This dataset consists of five variables across thirteen pressure levels ranging from 1000hPa to 50hPa:
   - **Geopotential Height (Z)**: This helps determine the height of pressure surfaces, which is crucial for understanding large-scale motion in the atmosphere.
   - **Specific Humidity (Q)**: Indicates the water vapor content in the air.
   - **Temperature (T)**: Air temperature at various altitudes, essential for modeling weather dynamics.
   - **U-Wind Component (U)** and **V-Wind Component (V)**: These represent the east-west and north-south components of wind speed across different atmospheric levels.

#### 2.2 Model Inference

In setting up a weather forecasting system, you need to organize your workspace into four key segments: an input data directory (`input_data`), an output data directory (`output_data`), the model files, and the model inference scripts.

The core of our system is a suite of predictive models, each tailored to different forecasting needs:

- **Hourly Forecast Model**: For those who need up-to-the-hour predictions, our 1-hour model (`pangu_weather_1.onnx`) provides timely updates.
- **Three-Hour Forecast Model**: Perfect for short-range planning, the 3-hour model (`pangu_weather_3.onnx`) offers a broader outlook.
- **Six-Hour Forecast Model**: Extending our foresight, the 6-hour model (`pangu_weather_6.onnx`) helps in anticipating changes throughout the day.
- **Daily Forecast Model**: Our most expansive, the 24-hour model (`pangu_weather_24.onnx`), is ideal for day-ahead planning and activities.

```
├── root
│   ├── input_data
│   │   ├── input_surface.npy
│   │   ├── input_upper.npy
│   ├── output_data
│   ├── pangu_weather_1.onnx
│   ├── pangu_weather_3.onnx
│   ├── pangu_weather_6.onnx
│   ├── pangu_weather_24.onnx
│   ├── inference_cpu.py
│   ├── inference_gpu.py
│   ├── inference_iterative.py
```

##### Exploring Weather Forecasting: The 24-Hour Model

Weather forecasting has greatly advanced with technologies like the 24-hour model, which improves the prediction of imminent weather patterns. This model focuses on two key types of data: **surface** and **upper-level**. Surface data covers essential information such as temperature and wind speed at Earth's surface, crucial for daily decisions like travel and outdoor activities. Upper-level data, meanwhile, helps in understanding atmospheric phenomena higher up, which influence broader weather systems.

In our recent analysis, we zeroed in on a specific region, selecting crucial variables to demonstrate how effectively this model predicts both ground-level and atmospheric conditions. This approach not only provides a snapshot of anticipated weather changes but also showcases the model's accuracy and utility in practical scenarios. This streamlined forecasting technique empowers us to make informed decisions, enhancing both daily life and long-term planning.

![](https://github.com/AriesChen-UPC/AriesChen-UPC/blob/main/Blog/T2M.png?raw=true)

To evaluate the predictive accuracy of the model, we compared its forecasts against actual observational data recorded 24 hours later. <u>The results revealed an average error of 0.04 degrees Celsius, with a standard deviation of 1.28 degrees Celsius.</u> The minimum deviation recorded was -8.13 degrees Celsius, while the maximum deviation was 29.19 degrees Celsius. From these comparisons, it is evident that the model demonstrates high predictive accuracy, making it highly valuable for practical applications.

![](https://github.com/AriesChen-UPC/AriesChen-UPC/blob/main/Blog/CleanShot%202024-04-25%20at%2009.37.21@2x.png?raw=true)

### 3. Conclusion

One of the standout features of Pangu-Weather is its computational efficiency. The model performs forecasts more than 10,000 times faster than traditional NWP systems, enabling real-time weather forecasting and rapid updates, which are crucial during severe weather events.

While Pangu-Weather represents a significant step forward, ongoing work aims to expand its capabilities, including refining its ensemble forecasting techniques and extending its predictive accuracy over longer time horizons. Future iterations will also explore the integration of additional atmospheric variables and the potential for real-time data assimilation.

Pangu-Weather epitomizes the potential of AI in revolutionizing weather forecasting. By integrating advanced AI techniques with traditional meteorological methods, it offers a faster, more accurate alternative to existing forecasting models, promising substantial benefits for disaster preparedness, agriculture, and daily weather prediction.
