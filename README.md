[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[image_1]: ./calibration_images/example_grid1.jpg'
[image_2]: ./output/warp-mask_example.jpg'
[image_3]: ./output/terrain-example.png'
[image_4]: ./output/gold-example.png'

[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/robotics)
# Term 1: Project 1: Search and Sample Return Project
![alt text][image_0] 

The goals of this project are to develope the three main parts of an autonomous Rover's "brain"
- Preception: Using Machine vision decompse images into naviable terrain, osbtacles, and objects of interest "Gold Rocks"
- Decision: Using the data provide by the perception module, decide on the robot's next course of action
- Actuation: Compute & send the actuations to drive the robot in the desired course.

Please note that the majority (or all) of Decision & Automation has been provided in the RoverSim Repo.

## Notebook Analysis

The Following is a summary of the preception module, developed and tested on the jupyter Notebook, [Rover Project Test Notebook](https://github.com/ExtEng/T1-P0-RoverSim/blob/master/code/Rover_Project_Test_Notebook.ipynb).

### 1 - Perspective Transform
Using the OpenCV warpPerspective function we can attain a bird's eye view of the terrain from the front view camera. As can be seen in the images below. In the same function, we create a mask of all the pixels that represent the image in the orginal image. This will be used later to create a map of all the obstacles in the map. 

![alt text][image_1] 
![alt text][image_2] 

### 2 - Color Thresholding
This part has 2 parts:
-Terrain maping 
-Objects of interest mapping

#### Terrain Mapping

Using the color Thresholding technoques demonstrated in the course, we can extract the navigable terrain but thresholding the wapred image. This yields a binary image where 1 equals a pixel along the path, and 0 equals a pixel that corresponds to an obstacle or the out of bounds of the image. Once we computed our naviable terrain, Using the mask generated earlier and the path map, we can commute the obstacle map, which is the inverse of the path minus the mask. See Below for an example.

![alt text][image_3] 

#### Objects of Interest Maping (aka Gold mapping)


![alt text][image_4] 





### 3 - Coordinate Transformation

### 4 - Process Image

## Autonomous Navigation and Mapping



