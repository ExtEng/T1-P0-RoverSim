[//]: # (Image References)
[image_0]: ./misc/rover_image.jpg
[image_1]: ./calibration_images/example_grid1.jpg
[image_2]: ./output/warp-mask_example.png
[image_3]: ./output/terrain-example.png
[image_4]: ./output/gold-example.png
[image_5]: ./output/Image_Reduction.png
[image_6]: ./output/Final_1.png
[image_7]: ./output/Final_2.png
[image_8]: ./output/Final_3.png

[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/robotics)
# Term 1: Project 1: Search and Sample Return Project
![alt text][image_0] 

The goals of this project are to develop the three main parts of an autonomous Rover's "brain"
- Perception: Using Machine vision decompose images into navigable terrain, obstacles, and objects of interest "Gold Rocks"
- Decision: Using the data provide by the perception module, decide on the robot's next course of action
- Actuation: Compute & send the actuations to drive the robot in the desired course.

Please note that the majority (or all) of Decision & Automation has been provided in the RoverSim Repo.

## Notebook Analysis

The Following is a summary of the perception module, developed and tested on the jupyter Notebook, [Rover Project Test Notebook](https://github.com/ExtEng/T1-P0-RoverSim/blob/master/code/Rover_Project_Test_Notebook.ipynb).

### 1 - Perspective Transform
Using the OpenCV warpPerspective function we can attain a bird's eye view of the terrain from the front view camera. See Images below. In the same function, we create a mask of all the pixels that represent the image in the original image. This will be used later to create a map of all the obstacles in the map. 

![alt text][image_1]

![alt text][image_2] 

### 2 - Color Thresholding
This part has 2 parts:
- Terrain mapping 
- Objects of interest mapping

#### Terrain Mapping

Using the color Thresholding techniques demonstrated in the course, we can extract the navigable terrain but thresholding the warped image. This yields a binary image where 1 equals a pixel along the path, and 0 equals a pixel that corresponds to an obstacle or the out of bounds of the image. Once we computed our navigable terrain, Using the mask generated earlier and the path map, we can commute the obstacle map, which is the inverse of the path minus the mask. The Range used for thresholding is [R Value > 165] & [G Value > 165] & [Blue Value > 165].

See Below for an example.

![alt text][image_3] 

#### Objects of Interest Mapping (aka Gold mapping)

Using images provided and pictures from trial run recordings, the RGB values where extracted and used to create a "gold" specific threshold function. Using this function any pixels corresponding to a range of RGB values that coincide with the color gold, will be converted to 1 in a binary image (rest will be zero). This is used to map the "gold rocks" locations on the world map. The Range used for thresholding is [R Value > 110] & [G Value > 110] & [Blue Value < 50].

See an example of the color thresholding on an image with a gold rock. 

![alt text][image_4] 

### 3 - Coordinate Transformation

Using the code and techniques presented in the class, the terrain map pixels to transformed to rover centric coordinates. Using the path pixels in rover-coordinates, a direction vector can be calculated using polar coordinates and the mean angle of all the path pixels. See Below for an example.

![alt text][image_5]

### 4 - Process Image

In the Process_image function, the following steps are applied to map the rover's environment.
1 Warp the Image to get the Bird's eye view.
2 Use selective Color Thresholding and techniques discussed to extract 3 maps 
- Navigable Terrain Map
- Obstacle Map
- Gold "rock" Map
3 Covert the maps to Rover-centric Coordinates
4 Convert the Rover-centric maps to their respective World coordinates
5 Update World map with the local data gathered.
- +5 value to the Blue Channel for Navigable terrain
- +1 value to the Red Channel for the Obstacle terrain and set to zero if the Blue channel is greater than zero. Meaning that the rover observed a path in that particular area, so there should be no obstacles. 
- set to 255 to the all the channels if gold is found in the world map. Marks the spot "white"

Using this Procedure, the images from the dash camera can be decomposed to map the Rover's Environment in 3 distinct colours, red for obstacles, blue for navigable terrain, and white for gold.

See Videos Below for 2 trials:

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/KbYnQKkQlVE/0.jpg)](http://www.youtube.com/watch?v=KbYnQKkQlVE)

[![IMAGE ALT TEXT HERE](http://img.youtube.com/vi/o9TpkYqn87k/0.jpg)](http://www.youtube.com/watch?v=o9TpkYqn87k)

## Autonomous Navigation and Mapping

Using the provided python scripts, and the code/techniques tested in the Rover Test Notebook, we can develop the perception and decision steps to autonomously map and navigate the environment simultaneously (cough SLAM) in the Udacity Rover Simulator.

### Perception Step

Most of the techniques used to perceive and map the environment, were discussed previously. I added an extra function to selective threshold images to find the pixel locations of the gold samples, which was also discussed.

The following steps are: 

1) Transform the front camera image to bird's eye view by using the perspective transform function.
2) Using the warped image, create 3 binary images that correspond to

- Navigable Terrain (also know as path map)
- Obstacles (also known as obstacle map)
- Gold Samples (also know as gold map)

using the thresholding and mask functions discussed

3) Store obstacle map, gold map, and path map in the R , G, and B Channels of Rover.vision_image respectively.

4) Convert the path map and obstacle map pixel coordinates to their rover-centric coordinates, using the provided rover_coord function

5) Convert the rover-centric to their world-map coordinates using the provided pix_to_world function

6) The next part was not tested in the Jupyter Notebook, but it was noticed and when the Rover traversed bumpy terrain or the side of an obstacle, the mapping error grew, which was due to the camera axis not being level, thus leading to incorrect mapping. To prevent this error, the world map is only updated when the rover is level, i.e. Pitch and Roll are within +-0.5 degrees of zero.

When updating the map, greater preference is given to the path map pixels than the obstacle pixels (5:1 ratio), when updating the world map. 

7) Calculate the polar distance and angles of the path map's rover-centric coordinates. This is will me used in the desision.py to set the rover's heading. The angles are updated to Rover.nav_angles

8) If the gold map is non-zero, (the rover saw a gold sample) then go through steps 4-6 with the gold map to map the rock sample's coordinates in the world map.

9) An additional step, (not used in the current decision step) was that if more than 4 pixels were found to coincide with a rock, to navigate the rover to the sample, the distances and angles of the rock sample pixels were stored into the Rover object to be used in future modification in the decision step.

### Decision Step 

The majority of the code was unaltered, and utilized the 2 mode state machine "forward" and "stop". Using this routine, the rover could map more than 40% of the map. The semi-brute force method uses the mean angle of the paths to set a steering heading for the Rover and utilizes the size of the nav_angles to determine the stop or forward behavior. This often gets trap in repetitive loops of stuck when an obstacle is dead center of the path (the mean angle is still straight). 

The main modification I made was to add some randomness to the Rover's behavior once it reaches a speed greater than 0.6, which was to alter the rover's heading by a random angle of (0 to +8 Degrees). This was my simple attempt to follow the left side of the wall as in a maze and add randomness to prevent getting stuck in a loop. From multiple runs, It was found qualitatively to improve the mapping time of the rover. 

Also, the Parameters stop_forward, and go_forward that controlled the stop and forward machine, were tuned to 80 and 900. This helped keep the rover away from the walls and turn more after stopping, preventing the rover getting stuck on the edges of the wall for a majority of the cases (it still hits the edges when turning wide).

See Images below for some the results. 

![alt text][image_6]

![alt text][image_7]

![alt text][image_8] 

## Future Work

My next steps for would be:

1) Instead of using the mean nav_angle, create a wall following algorithm that utilizes the Nav_distances to avoid small obstacles in the Rover 's path and the Nav_angles to follow the left side of the obstacles wall. This should prevent any repetitive navigation loops and prevent collisions.

2) Use a rock pixel limit (4 or greater) to prompt a "Search" state, which will use the sample’s angles and distance to navigate to the sample and trigger the pick up once in range. Then go back to either forward or stop state depending on conditions.

3) Create a Navigation plan that returns the rover to start position, using difference of the Rover's current position and the start position to create a navigation plan, while maintaining the current behavior of the rover following the path. i.e. combination of global and local planning.

4) Make 2 maps of navigable terrain, one close range, one both close and far. The close range will be used to update the map (closer means high accuracy) and the combined map can be used for navigation because the farther pixels then to be more useful in deciding the Rover's path.
