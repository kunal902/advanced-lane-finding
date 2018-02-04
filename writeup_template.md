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

[//]: # (Image References)

[image1]: ./output_images/chessboard_undistorted.png " Chessboard Undistorted"
[image2]: ./output_images/chessboard.png "Chessboard"
[image3]: ./output_images/undistorted.png "Undistorted"
[image4]: ./output_images/thresholded.png "Thresholded"
[image5]: ./output_images/warped.png "Warped"
[image6]: ./output_images/sliding_window_search.png "Sliding Window Search"
[image7]: ./output_images/polyfit_based_on_previous_frame.png "Polyfit Based On Previous Frame"
[image8]: ./output_images/histogram.png "Histogram"
[image9]: ./output_images/final_output.png "Final Output"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image3]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. For gradient threshold first I used Sobelx operator along with thresholding which emphasizes the edges closer to vertical and was giving good result with the images then I used directional gradient as we are interested in finding edges in a particular orientation. After that I applied color threshold also on R and G channel as they are high for white and yellow lane lines but their values change under varying levels of brightness so alone it is not sufficient, then I applied L channel threshold so that edges generated decause of shadows can be ignored, then S channel threshold was also applied as it stays fairly consistent even in shadows or excessive brightness and detects both the white and yellow lane lines. After applying all these steps I also applied a region of interest mask to avoid the top parts of the image. All these steps are clearly written with comments in `retrieve_thresholded_image()` function in IPython notebook. Here's an example of my output for all these steps.

![alt text][image4]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `calculate_perspective_transorm()` function in IPython notebook. After manual examination of the images the source and destination points for perspective transform are choosen. The destination points are choosen manually so that the lane lines appear more or less parallel in the warped image.

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 570, 470      | 320, 1        |
| 220, 720      | 320, 720      |
| 1110, 720     | 920, 720      |
| 722, 470      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I calculated the histogram of the warped image in `calculate_histogram()` function in IPython notebook due to which I got two peaks for left and right lane lines. The result is attached below:

![alt text][image8]

 In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in the above histogram are good indicators of the x-position of the base of the lane lines so the points can be used as a starting point for where to search for the lines. From that point a sliding window is placed around the line centers to find and follow the lines up to the top of the frame. I have used 10 windows of width 100 pixels and all the steps are described clearly along with the comments in `sliding_window_search()` function in IPython notebook. The result is attached below:

 ![alt text][image6]

 I also applied some optimization which is described clearly in `polyfit_based_on_previous_frame()` function in IPython notebook. After all these steps I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I computed the radius of curvature of the fit after converting x and y values to the real world space. For this project, I have assumed that if you're projecting a section of the lane, the lane is about 30 meters long and 3.7 meters wide. All these steps are described in `compute_radius_of_curvature_in_mtrs()` function in IPython notebook.

To calculate the position of the vehicle with respect to the center, I have assumed that the camera is mounted at the center of the car, such that the lane center is the midpoint at the bottom of the image between the two lines detected earlier. The offset of the lane center from the center of the image (converted from pixels to meters) is the distance from the center of the lane. All these steps are described in `compute_centre_in_mtrs()` function in IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `final_pipeline()` function in IPython notebook.  Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

## [Youtube Link](https://youtu.be/mYoZWbPRtBs)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project I followed the techniques and recommendations given by Udacity in the classroom lectures. The pipeline worked well on the project video but not so well on the challenge videos.

#### Potential Shortcomings

One potenatial shortcoming is that the current pipeline seems to fail when the road has sharper turns and at very short intervals.

Also it may fail when the separation between the lane lines are smaller.

The current pipeline may fail when the shape and lane direction changes rapidly.

The current pipeline will likely fail in the situations when the road has cracks coming alongside the lane lines

#### Possible Improvements

Sanity checks as discussed in the classroom lectures can also be implemented to obtain better results

We can choose a better perspective transform technique. As in case of sharper turns we can choose a smaller section to take the transform

Averaging over smaller number of frames when the shape and direction of the lane changes quite fast
