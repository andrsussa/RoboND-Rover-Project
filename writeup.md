## Project: Search and Sample Return

---

**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/TODO1N2.jpg
[image2]: ./misc/TODO3obs.jpg
[image3]: ./misc/TODO3rocks.jpg
[image4]: ./misc/TODO3.jpg
[image5]: ./misc/TODO4.jpg
[image6]: ./misc/TODO5.jpg
[image7]: ./misc/TODO6.jpg
[image8]: ./misc/PERS1.jpg
[image9]: ./misc/PERS2.jpg
[image10]: ./misc/PERS3.jpg
[image11]: ./misc/PERS4.jpg
[image11]: ./misc/DESC1.jpg

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

I followed the instructions by running the notebook as it was, taking as input the test images. Afterwards, I recorded my own data making a not so extensive but thorough route (including rocks, end of path and returning to the starting point) and changed the path variable to include this new data and also the path to the csv file.

Then I started to modify the "TODO" section in the `process_image()` function. First, I added the variables to hold the coordinates for the perspective transform and then added the function itself feeding the input image to it, as seen in the next image.

![alt text][image1]

Second, I added a line to take the output of the previous step and then apply a threshold to it to obtain the corresponding navigable terrain. I knew that I had to add also a way to obtain the obstacles and rocks on sight. To perform the obstacles' detection I created a new function called `obstacle_thresh()`, which would do the same as the previous threshold function but considering the pixels below the value 160, rather than above. This code can be seen in the image below.

![alt text][image2]

Later, I took one of the first snippets used in the course to show an image that included a rock. By passing the pointer over the section of the rock, I noticed that the Red and Green values stayed mostly above 160 and the Blue value showed values below 80, then I picked those values to apply a threshold to discriminate the rock pixels. I added a new function called `rock_thresh()` solely for this purpose as seen below.

![alt text][image3]

#### 2. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

I used the thresholding functions mentioned in the previous section as seen in the following image, creating a binary matrix for each of the differents purposes of detection.

![alt text][image4]

Subsequently, I used the `rover_coords()` function to convert the previously obtained pixel values into rover-centric coordinates, as seen next.

![alt text][image5]

Then, I used the `pix_to_world()` function to map the X and Y pixels of the obtained navigable, obstacles and rock coordinates into world coordinates. This function also took into account the rover's position and yaw values, and the worldmap dimensions and scale.

![alt text][image6]

Finally, in order to update the map with the coordinates calculated, depending on the type of terrain, a pixel was painted a different color. If it was navigable terrain, it would be blue, obstacles would be red, and rocks would be painted green. This was done with the following lines of code.

![alt text][image7]


### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

Filling the `perception_step()` function started by referencing the img property
of the Rover object into a `image` variable. I used this to calculate the
destination coordinates for the perspective transformation. The source and
destination for this action were calculated exactly as done in the jupyter
notebook; this can be seen in the following image.

![alt text][image8]

Then I used the warped image to use it as input in the different types of thresholding
functions to calculate their respective binary images. These images were then
used to draw the `vision_image` attribute of the rover, using a different color
for each type of threshold. At first try, this was not working but after some
research, I found out that I had to multiply the images by 255 for it to show
properly in the vision image.

Afterwards, I followed including the conversion from binary pixel values into
rover-centric coordinates, and then into world coordinates, as done before in
the jupyter notebook. Likewise, I mapped these coordinates into the world map.
These three last steps can be seen in the next image.

![alt text][image9]

After watching how it was drawing the navigable terrain and obstacles over the world map, I found
out that there was a way to ignore the not mappable section of the images from
ending up in the map. I could first perform the thresholding step and then pass
it through separate perspective transforms for each type of perception. I
decided to modify these steps in the following way.

![alt text][image10]

Finally, all that was left was to perform the conversion of the navigable
coordinates into polar coordinates, to obtain the navigable distance and angles.
I did this in the following way:

![alt text][image11]


#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

The simulator was launched at a windowed 1024x768 resolution at the fastest
graphics quality and it ran at around 30 FPS.

The first thing I noticed when launching the simulator after finishing the
perception step was that the rover was wobbling a lot and that the fidelity
percentage remained low throughout the navigation.

I followed the recommendation of limiting the instances in which a perception
process would be valid by taking into account the proper ranges of yaw and pitch
values of the rover. After trying several combinations of values for these, I
settled with the following.

![alt text][image12]

After this, the fidelity percentage improved but it affected the ability of the
rover to turn effectively. This is an issue that could be improved.

I also noticed that the rover often fell in a loop in which it kept a constant
turn in one direction and couldn't find a proper path, or would collide into an
obstacle like a big rock, but at the same time, the camera could not detect this
so it would throttle forward hopelessly. These are issues that are work in
progress and that could be solved trying to lean the rover into following a path
that preferred to follow walls, eventually mapping the whole map.

There is also left to add the decision process to pick up the rocks when they
are detected. This could be done by stopping and scanning the surrounding when a
rock is found, and going forward when the detected rock is located in a center
angle with respect to the current position of the rover.

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

