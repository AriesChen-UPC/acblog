---

title: "Using MATLAB for AprilTag detecting"
description: ""
tags: [
    "MATLAB",
    "AprilTag",
]
date: "2022-07-01"
categories: [
    "Robotics",
    "Perception",
    "Software"
]
author: "AriesChen"
menu:
  main:
    parent: "docs"
    weight: 3

---

> AprilTags are conceptually similar to QR Codes, in that they are a type of two-dimensional bar code. However, they are designed to encode far smaller data payloads (between 4 and 12 bits), allowing them to be detected more robustly and from longer ranges.

![](https://miro.medium.com/v2/resize:fit:1400/1*IX1otmM33h_mec4islurfA.jpeg)

Image from University of Michigan

<u>AprilTag is a visual fiducial system popular in robotics research.</u> When we do some work related to ROS or something needed to be labeled, there is AprilTag. Here we use MATLAB to do some research about AprilTag.

## Create AprilTag

Here we use OpenMV to create AprilTags in the **TAG36H11** family. We create six tag IDs from family 0 to family 5 for a test.

Here is an example: TAG36H11 0

> The OpenMV project is about creating low-cost, extensible, Python-powered, machine vision modules and aims at becoming the “Arduino of Machine Vision**”**.

![](https://miro.medium.com/v2/resize:fit:1400/1*sGFg3EfYnlyqGhXJauwVVw.png)

TAG36H11 created by OpenMV

## Place the AprilTag

We use the AprilTag to label documents, and stapler e.g., the tags are placed irregularly with different angles. Our aim is to test the accuracy of the recognition.

![](https://miro.medium.com/v2/resize:fit:1400/1*LZEAWk5F9MkosIxIkZcF5Q.png)

Our office desk

## Detect the AprilTag

First, we load the picture including AprilTags in it, and store it in the MATLAB workspace for later use.

```matlab
[file,path] = uigetfile({'*.jpg;*.bmp;*.png','All Image Files';'*.*','All Files'});
if isequal(file,0)
   disp('User selected Cancel');
else
   disp(['User selected ', fullfile(path,file)]);
end
imagePath = fullfile(path,file);
I = imread(imagePath)
```

Then using the **readAprilTag** function to detect the AprilTags.

The AprilTag formats are supported within “tag36h11”, “tagCircle21h7”, “tagCircle49h12”, “tagCustom48h12”, “tagStandard41h12”.

> readAprilTag: Detect and estimate pose for AprilTag in image

```matlab
[id,loc,detectedFamily] = readAprilTag(I,tagFamily);
for idx = 1:length(id)
        disp("Detected Tag ID, Family: " + id(idx) + ", " + detectedFamily(idx));
        markerRadius = 8;
        numCorners = size(loc,1);
        markerPosition = [loc(:,:,idx),repmat(markerRadius,numCorners,1)];
        I = insertShape(I,"FilledCircle",markerPosition,Color="red",Opacity=1);
end
```

Here we can see, some incorrect corners are detected which are labeled with red circles. Some are detected as tagCircle21h7 family.

```matlab
Detected Tag ID, Family: 0, tag36h11
Detected Tag ID, Family: 1, tag36h11
Detected Tag ID, Family: 2, tag36h11
Detected Tag ID, Family: 3, tagCircle21h7
Detected Tag ID, Family: 3, tag36h11
Detected Tag ID, Family: 4, tag36h11
Detected Tag ID, Family: 5, tag36h11
Detected Tag ID, Family: 11, tagCircle21h7
Detected Tag ID, Family: 17, tagCircle21h7
```

![](https://miro.medium.com/v2/resize:fit:1400/1*IvQORGkpDPYCiJ5wbUEVTg.png)

RGB image detection

So, we change the RGB picture to binary and do the detection again. The result of the detection is correct. Only the AprilTag we placed was detected, which was labeled by the four corners.

```matlab
[I, imageTransformName]= image2binary(imagePath);
```



![](https://miro.medium.com/v2/resize:fit:1400/1*jQCRCa8bC5k6mYDoQzJ1ZA.png)

Binary image detection

# Estimate AprilTag Poses in Image

We use the camera’s intrinsic parameters to estimate AprilTag poses.

First, we load the information from the camera.

Second, specify the tag size in meters and undistort the input image using the camera’s intrinsic parameters.

Third, detect a specific family of AprilTags and estimate the tag poses, and set the origin for the axes vectors and for the tag frames.

Finally, we can add the tag frames and IDs to the image.

```matlab
info = imfinfo(imagePath);
focalLength = info.DigitalCamera.FocalLength;
widthPixel = info.Width;
lengthPixel = info.Height;
focalLength = [focalLength * lengthPixel,focalLength * widthPixel];
principalPoint = [lengthPixel/2, widthPixel/2];  % principalPoint
imageSize = [lengthPixel, widthPixel];
intrinsics = cameraIntrinsics(focalLength,principalPoint,imageSize);
tagSize = 0.04;
I = undistortImage(I,intrinsics,"OutputView","same");
[id,loc,pose] = readAprilTag(I,"tag36h11",intrinsics,tagSize);
worldPoints = [0 0 0; tagSize/2 0 0; 0 tagSize/2 0; 0 0 tagSize/2];
for i = 1:length(pose)
    imagePoints = worldToImage(intrinsics,pose(i).Rotation, ...
                  pose(i).Translation,worldPoints);
    I = insertShape(I,"Line",[imagePoints(1,:) imagePoints(2,:); ...
        imagePoints(1,:) imagePoints(3,:); imagePoints(1,:) imagePoints(4,:)], ...
        "Color",["red","green","blue"],"LineWidth",7);
    I = insertText(I,loc(1,:,i),id(i),"BoxOpacity",1,"FontSize",60);
end
```

![](https://miro.medium.com/v2/resize:fit:1400/1*CbhR6LZF3xGI2JRjnKrU-A.png)

AprilTag detection

AprilTags are widely used as visual markers for applications in object detection, and localization, and as a target for camera calibration.

So, take a try!

