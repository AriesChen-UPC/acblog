---

title: "Extract Lidar and Camera Data from Rosbag File with MATLAB"
description: ""
tags: [
    "MATLAB",
    "Lidar",
    "Camera",
    "Rosbag",
]
date: "2022-10-01"
categories: [
    "Robotics & Perception",
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3

---

> ROS (Robot Operating System) provides libraries and tools to help software developers create robot applications. It provides hardware abstraction, device drivers, libraries, visualizers, message-passing, package management, and more. ROS is licensed under an open source, BSD license.

**Requirement:**

1.  Lidar Toolbox
2.  ROS Toolbox

**Optional:**

1.  Example data: The data is collected from ssl\_slam, which relates to the article _<_[_Lightweight 3-D Localization and Mapping for Solid-State LiDAR_](https://github.com/wh200720041/ssl_slam) _(Intel Realsense L515 as an example)\>_.

## Load a rosbag

First, we load the rosbag file using the function **rosbag**. The file was load as a **BagSelection** in MATLAB. The BagSelection object is an index of the messages within a rosbag. You can use it to extract message data from a rosbag, select messages based on specific criteria, or create a timeseries of the message properties.

> [**MATLAB rosbag Structure**](https://www.mathworks.com/help/ros/ug/ros-log-files-rosbags.html)  
> The BagSelection object has the following properties related to the rosbag:  
> **• FilePath:** a character vector of the absolute path to the rosbag file.  
> **• StartTime:** a scalar indicating the time the first message was recorded  
> **• EndTime:** a scalar indicating the time the last message was recorded  
> **• NumMessages:** a scalar indicating how many messages are contained in the file  
> **• AvailableTopics:** a list of what topic and message types were recorded in the bag. This is stored as table data that lists the number of messages, message type, and message definition for each topic. For more information on table data types, see Access Data in Tables. Here is an example output of this table.  
> **• MessageList:** a list of every message in the bag with rows sorted by time stamp of when the message was recorded. This list can be indexed and you can select a portion of the list this way. Calling select allows you to select subsets based on time stamp, topic or message type.

```matlab
[file,path] = uigetfile('*.bag');
if isequal(file,0)
   disp('User selected Cancel');
else
   disp(['User selected ', fullfile(path,file)]);
end
fileName = fullfile(path,file);
bag = rosbag(fileName);
bag.AvailableTopics
```

![](https://miro.medium.com/v2/resize:fit:1400/1*8fVwI-AJtCj7ySAMv5L8TA.jpeg)

<center>Available Topics in rosbag</center>

## Select the subsets

Second, we call the **select** function to select _“/camera/color/camera\_info”, “/camera/color/image\_raw”, “/camera/depth/camera\_info”, and “/camera/depth/color/points”_. The **image\_raw** and **points** subsets are the data we process later.

```matlab
bag.MessageList
```

![](https://miro.medium.com/v2/resize:fit:944/1*gJwwgtNBp2HJSd1yfPISzw.jpeg)

<center>Message List in rosbag</center>

```matlab
bagColorCamera = select(bag,"Topic","/camera/color/camera_info"); 
bagImage = select(bag,"Topic","/camera/color/image_raw"); 
bagDepthCamera = select(bag,"Topic","/camera/depth/camera_info");  
bagPoints = select(bag,"Topic","/camera/depth/color/points");
```

## Extract the Lidar and Camera Data

Third, we use the **readMessages** function to read all the messages in “/camera/color/image\_raw”. The **readImage** function converts the data into an appropriate MATLAB image and returns it in img.

```matlab
imageMsgs = readMessages(bagImage);
imageData = readImage(imageMsgs{1});  % the first frame
imshow(imageData)
```

![](https://github.com/AriesChen-UPC/AriesChen-UPC/blob/main/Blog/CleanShot%202024-03-27%20at%2009.11.21@2x.png?raw=true)

<center>Camera data</center>

The **pointCloud** function creates point cloud data from a set of points in 3-D coordinate system.

```matlab
pcMsgs = readMessages(bagPoints);
pcData = pointCloud(readXYZ(pcMsgs{1}));  % the first frame
pcshow(pcData,"BackgroundColor",'w')
view([15.00 50.00])
```

![](https://github.com/AriesChen-UPC/AriesChen-UPC/blob/main/Blog/CleanShot%202024-03-27%20at%2009.11.33@2x.png?raw=true)

<center>Lidar data</center>