## Advanced Lane line Detection Project Wirteup

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # "Image References"

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./test_videos_output/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first section (code cell 2-5) of the IPython notebook located in "./Notebook.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

After undistort transformation, the image becomes:

![undistort_test1](examples/undistort_test1.jpg)



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at function `abs_sobel_thresh` and `color_thresh` code cell #10 in `Notebook.ipynb`).  Here's an example of my output for this step, where red pixels are identified by color thresholds, and green pixels are identified by gradient thresholds.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform()`, which appears in code cell #10.  The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points as follow:

|  Source   | Destination |
| :-------: | :---------: |
| 263, 680  |  260, 720   |
| 1044, 680 |  1040, 720  |
| 582, 460  |   260, 0    |
| 699, 460  |   1040,0    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I plot the distribution of the bottom half pixels' x vlaues on histogram and pick the peak of the histogram as the starting position to search for pixels belonging to the lane line. This is performed on the first image of the video. The code is implemented in function `find_lane_points` in code cell #12.

For subsequent images in the video, I use the fitted lines from the previous image as the center of the search window and set search margin to 100 pixels. This code is implemented in function `find_lane_points_with_prior` in code cell #12.

Then I fit my lane lines with a 2nd order polynomial like this:

![alt text][image5]

This is implemented in function `fit` of Class `Line` in code cell #9.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in funciton `calc_curvature` of Class `Line` in code cell #9.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines #30 through #47 in code cell #17.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

One difficulty I had when working on this project was that in some frames where there were heave shades covering part of the road, it was hard to correct identifying the pixels belonging to the lane line. I inspected the bianry pixels pickuped by both color and gradient threshold and find that most of the wrong pixels came from color thresholding, therefore I increased the min color threshold to remove some of the incorrectely included pixels. I also introduced smoothing between frames so that the lane lines does not change radically. 

To make the pipeline more robust, we can further analyze the fitted line in each frame. Depending on the curvature, and whether the two lines are parallel, we can then decide how much we adjust the lane line function from the previsou frame. 