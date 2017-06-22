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

[//]: # (Image References)

[calibration1]: ./output_images/calibration1.png "Calibration Image"
[calibration2]: ./output_images/calibration2.png "Undistorted Chessboard"
[undistorted_test]: ./output_images/undistorted_test.png "Undistorted test Image"
[thresholded_binary]: ./output_images/threshold.png "Thresholded Binary Image"
[warped_normal]: ./output_images/warped_normal.png "Warped Normal Image"
[warped_thresholded]: ./output_images/warped_thresholded.png "Warped Thresholded Image"
[warped_histogram]: ./output_images/warped_histogram.png "Warped Thresholded Histogram Image"
[find_lane_lines]: ./output_images/find_lane_lines.png "Found lane lines Image"
[composited_image]: ./output_images/composited_image.png "Composited Image"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the section of the IPython notebook labeled "Finding and correcting for distortion".

Building off of the sample code from the class, I built the object points based on the 9x6 shape of the chessboard corners.  I then enumerated through all of the calibration images, converted to grayscale and used the cv2 function findChessboardCorners to update the object point coordinates.

Next, I used the cv2 calibrateCamera function to compute the calibration and distortion coefficients.

In the final cell of this section I tested the undistortion on each of the test images included for the project.

![alt text][calibration1]
![alt text][calibration2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Here is the distortion correction applied to a test image:
![alt text][undistorted_test]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I created a function, apply_thresholding, to perform the methods to create a thresholded binary image.  I tested combining a few different methods and ultimately ended up on using a similar one to what is done in module 30 of the Advanced Lane Finding portion of the course, combining Sobel Threshold and Threshold color channel into a binary gradient.

![alt text][thresholded_binary]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I created a function called warp_image which took two parameters, the image itself and the option to reverse the warp.  I hard-coded my src and destination matrices into this function by taking the shape of the input image and moving from the edges of those points.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 582, 460      | 300, 0        | 
| 700, 460      | 980, 0      |
| 1050, 705     | 980, 720      |
| 250, 405      | 300, 720        |

From here I applied the cv2.warpPerspective function to warp to a "birds-eye" view of the lane lines and returned the warped image.

An example on the original image:
![alt text][warped_normal]

And a thresholded binary of the same image that has been warped:
![alt text][warped_thresholded]

And a calculated histogram on the above threshold, showing the likely lane line positions:
![alt text][warped_histogram]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In my find_lane_lines function is where I determine the positions of the lane lines.  This function takes the warped thresholded binary image and a left and right Lane() varaibles as parameters.

The function calculated the historgram of the warped binary image, finds the peak positions and calculates the positions of nonzero pixels.  If a lane line hasn't been previously detected a sliding window search is done to find the lane line and update the corrseponding left and right varaibles.  If a lane line was previously detected then the sliding window search is skipped and this is set based on the previous fit.

Finally, an output image containing what was found is returned.  A few other variables are also returned (but only used for testing purposes).

![alt text][find_lane_lines]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In my function "find_curve_and_carlane_pos" I found the radius of the curvature of the lane and position of the vehicle against center.  The warped binary image and left and right Lane() variables are parameters.

The fit stored in the Lane() variables is used to fit a polynomial against the pixel positions (multiplied by the assumed y and x meters per pixel to get real world meters).  That is then used to calculate the radius of the curvature for the left and right lane lines.

For the car position, earlier the fitted shape of the left and right lines is sampled against the y space..  The mean of the left and right lane lines is taken and assumed to be the car's position.  The lane "middle" is assumted to be halfway inside of the warped binary image.  This is subtracted from the previous car position and set to the distance that the car is off center.

The average curve, car off center distance and a "color warped" image are returned by this function.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

From the previous function I return this image:

![alt text][composited_image]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

All of the previous code was written into functions so I combined them in a main_pipeline function which performs them all and is fed into fl_image to generate the following video clip:

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I compiled most of the approaches outlined in the class to make this end result.  It performs reasonably well on the first example but not on the advanced ones.

Although I implemented the Lane() class as suggested it's missing portions.  Most critically, I don't have any code to determine when the line has been "lost" and to have the program do another sliding window search.  I believe if I find a robust way to do this my program would perform much better on the other sample cases.
