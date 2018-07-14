## Writeup

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

[image_UndistortCalibration]: ./output_images/UndistortCalibration.PNG "Undistorted calibration image"
[image_UndistortTest]: ./output_images/UndistortTest.PNG "Undistorted test image"
[image_WarpTest1]: ./output_images/WarpTest1.PNG "Warped test image"
[image_WarpTest2]: ./output_images/WarpTest2.PNG "Warped test image"
[image_Threshold]: ./output_images/Threshold.PNG "Applied thresholds"
[image_PolyFit]: ./output_images/PolyFit.PNG "Fitted polynomial"
[image_PlotBack]: ./output_images/PlotBack.PNG "Found lane plotted back to original image"

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the jupyter notebook.
I loaded the 20 calibration images from "./camera_cal" (lines 22). Then, I searched for chessboard corners (line 26) and added all found corners to a list (line 31).
I finally calibrated the camera with the data from all 20 calibration images (line 41).

The following image shows an original calibration image (left) and the corresponding undistorted image (right).
![alt text][image_UndistortCalibration]

The following image shows an original test image from the "test_images"-folder (left) and the corresponding undistorted image (right).
![alt text][image_UndistortTest]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The distortion-correction is performed in line 342.

The following image shows an original test image from the "test_images"-folder (left) and the corresponding undistorted image (right).
![alt text][image_UndistortTest]

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The offline part of the perspective transform that computes the transformation matrix is done in the second code cell of the jupyter notebook.
The source an destination points were chosen as follows (lines 11-19):

| Source        | Destination   |
|:-------------:|:-------------:|
| 280, 670      | 250, 720        |
| 580, 460      | 250, 0          |
| 700, 460      | 1030, 0      |
| 1030, 670     | 1030, 720        |

The following two images show an original test image with straight road with the source points as red lines (left) and the corresponding warped image with destination points as red lines:
![alt text][image_WarpTest1]
![alt text][image_WarpTest2]

In the pipeline (3rd code cell), I did the perspective correction before the thresholding (line 343).


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The pipeline performs the color thresholding in line 344.
I used color thresholds in the helper function "threshold" (3rd code cell, line 122-149) to find the lane lines.
I used lower and upper thresholds on the R, G, and B channel to identify white and yellow lanes (lines 132-138).
Additionally, I used lower and upper thresholds on the H, L and S channel to find yellow lines (lines 140-142).
The thresholds to find the lane lines are listed in the table below.

|| Lower threshold        | Upper threshold   |
|:--:|:-------------:|:-------------:|
|RGB white| (100, 100, 100)      | (255, 255, 255)        |
|RGB yellow| (225, 180, 0)      | (255, 255, 170)          |
|HLS yellow| (20, 120, 80)      | (45, 200, 255)      |

The following image shows an original test image (top) and the applied color thresholds (bottom):
![alt text][image_Threshold]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The pipeline performs the polynomial fitting in line 349.
The helper function "fitpolynom" (line 158 onwards) performs this task.
To find the lane lines I used the convolution approach.
The beginning of the lane line that is closest to the car and thus at the bottom of the image is found by convolution of the window with the lower quarter part of the image (lines 169-186).
Starting from this initial lane part, the algorithm goes from all layers, bottom to top (line 190).
If pixels in the thresholded image are found, I take the mean value of the maximum of the result of the convolution and add it to the data, that will be used to fit the polynomial (lines 190-218).
Next I fit a quadratic polynomial into the found data (lines 252-282).

The following images shows the thresholded image (left) and the corresponding fitted line (right):
![alt text][image_PolyFit]
The green squares show the convolution window.
The red dots show the data that is used to fit the polynomial.
The polynomial itself is shown by a yellow line.


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In order to calculate the radius of the curvature of the lane, I first transformed the data from pixel space to meter space with fixed ration (lines 285, 286).
I then did the polynomial fit again (lines 288, 289) and calculated the radius (lines 291, 292).
The final radius is the mean value of the left and right lane radius (line 293).

The calculation of the vehicle position with respect to center is performed in lines 297-300.
I extracted the lane position at the bottom part of image, calculated the mean value and compared that to the center of the image.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The plotting back of the found lane is performed in lines 350,351 of the pipeline through the helper function "unwarp" (lines 304-337).
The following image shows an original test image (left) and an image with the identified lane lines (right).
![alt text][image_PlotBack]



---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I spent most time of this project on finding a reasonable thresholding to find the lane lines.
I also did not test these thresholds on the challenge videos that have a lots of glare.
I also did not use information from the identified lane of the previous frame.
This measure should improve the robustness of the lane finding, e.g. in situation where there are only small pieces of lane lines.
