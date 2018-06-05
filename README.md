[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[image_1]: ./calibration_images/example_grid1.jpg
[image_2]: ./output/warp-mask_example.png
[image_3]: ./output/terrain-example.png
[image_4]: ./output/gold-example.png
[image_5]: ./output/Image_Reduction.png

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
- Terrain maping 
- Objects of interest mapping

#### Terrain Mapping

Using the color Thresholding technoques demonstrated in the course, we can extract the navigable terrain but thresholding the wapred image. This yields a binary image where 1 equals a pixel along the path, and 0 equals a pixel that corresponds to an obstacle or the out of bounds of the image. Once we computed our naviable terrain, Using the mask generated earlier and the path map, we can commute the obstacle map, which is the inverse of the path minus the mask. The Range used for thresholding is [R Value > 165] & [G Value > 165] & [Blue Value > 165].

See Below for an example.

![alt text][image_3] 

#### Objects of Interest Maping (aka Gold mapping)

Using images provided and pictures from trial run recordings, the RGB values where extracted and used to create a "gold" specific threshold function. Using this function any pixels correspding to a range of RGB values that conicide with the color gold, will be converted to 1 in a binary image (rest will be zero). This is used to map the "gold rocks" locations on the world map. The Range used for thresholding is [R Value > 110] & [G Value > 110] & [Blue Value < 50].

See an example of the color thresholding on an image with a gold rock. 

![alt text][image_4] 

### 3 - Coordinate Transformation

Using the code and techniques presented in the class, the terrain map pixels to transformed to rover centric coordinates. Using the path pixels in rover-coordinates, a direction vector can be calculated using polar coordinates and the mean angle of all the path pixels. See Below for an example.

![alt text][image_5]

### 4 - Process Image

In the Process_image function, the folowing steps are applied to map the rover's enviroment.
1 Warp the Image to get the Bird's eye view.
2 Use selective Color Thresholding and tecniques discussed to extract 3 maps 
- Navigable Terrain Math
- Obstacle Map
- Gold "rock" Map
3 Covert the maps to Rover-centric Coordinates
4 Convert the Rover-centric maps to their respective World coordiantes
5 Update World map with the local data gathered.
- +5 value to the Blue Channel for Navigable terrain
- +1 value to the Red Channel for the Obstacle terrain and set to zero if the Blue channel is greater than zero. This says that the rover say a path there, so there should be no obstacels. 
- set to 255 to the all the channels if gold is found in the world map. Marks the spot "white"

Using this Procedure the images from the dash camera can be decomposed to map the Rover's Enviroment in 3 distict colours, red for obstacles, blue for navigable terrain, and white for gold.

See Videos Below for 2 trials:

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/KbYnQKkQlVE/0.jpg)](http://www.youtube.com/watch?v=KbYnQKkQlVE)

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/o9TpkYqn87k/0.jpg)](http://www.youtube.com/watch?v=o9TpkYqn87k)

## Autonomous Navigation and Mapping



