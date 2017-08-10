
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

[video1]: ./output_images/proj.mp4 "Project Video"
[image1]: ./output_images/test1_orig.png "Original"
[image2]: ./output_images/test1_undistorted.png "Undistorted"
[image3]: ./output_images/test1_roi.png "ROI"
[image4]: ./output_images/test1_blur.png "Warp and Blur"
[image5]: ./output_images/test1_s_AND_l.png "Bitwise AND of L and S"
[image6]: ./output_images/test1_sobelx.png "Sobelx"
[image7]: ./output_images/test1_sobely.png "Sobely"
[image8]: ./output_images/test1_sobelmag.png "Sobelmag"
[image9]: ./output_images/test1_final.png "Final"

[image10]: ./output_images/chessboardwithcornersidentified.png "chessboard"

[image11]: ./output_images/test3_orig.png "Original"
[image12]: ./output_images/test3_undistorted.png "Undistorted"
[image13]: ./output_images/test3_roi.png "ROI"
[image14]: ./output_images/test3_blur.png "Warp and Blur"
[image15]: ./output_images/test3_s_AND_l.png "Bitwise AND of L and S"
[image16]: ./output_images/test3_sobelx.png "Sobelx"
[image17]: ./output_images/test3_sobely.png "Sobely"
[image18]: ./output_images/test3_sobelmag.png "Sobelmag"
[image19]: ./output_images/test3_final.png "Final"

[image20]: ./output_images/test1_histogram.png "histogram"
[image21]: ./output_images/test3_histogram.png "hist"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

The whole project is captured in a single IPython notebook.
The first half of the IPython notebook contain operations/functions that work on a single image in order to collect and tune various images/thresholds

The second part of the notebook has the definition of the core pipeline for video processing which includes fitting of lines

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The core code for this step is contained in function calibrate_camera  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. The core function, `calibrate_camera`, takes in the list of chess board image filenames and processes them sequentially. image and object points from all images are appended to `imgpoints` and `objpoints` respectively.Here is sample chess board with corners identified

![alt text][image10]

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function in code cell 5.  I applied this distortion correction to the test image using the `cv2.undistort()` function.

### Pipeline (single images)

My pipeline had the following operations in this order:
1) Undistort
2) Apply ROI mask
3) Transform perspective and blur
4) Get sobelx, sobely, sobelmag, s_channel, l_channel
5) Combine all of these in a way as to minimize overall noise 
6) Get the histogram and start detecting candidate points
7) Fit quadratric polynomials to detected (and historic) points
8) Calculate curvature and center offset
9) Draw the lane on warped image and then unwarp
10) Combine with orig image

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to acouple of the test images like these:
![alt text][image1] ![alt text][image11]


Corresponding undistorted images:
![alt text][image2] ![alt text][image12]



#### 2. ROI applied

The ROI points were similar to the ones to used to generate the warp/unwarp matrices

![alt text][image3] ![alt text][image13]


#### 3. Warp and apply Gaussian Blur

The blur helped reduce a lot of interlane noise
![alt text][image4] ![alt text][image14]


#### 4. Bitwise AND of L and S channel (thesholded)
![alt text][image5] ![alt text][image15]


#### 5. Example of Sobelx, Sobely and Sobel_mag

Sobelx:
![alt text][image6] ![alt text][image16]


Sobely:
![alt text][image7] ![alt text][image17]




#### 6. Final cobination of all images
All images were combined in a specific order using bitwise operators. Based on observing various combination, the final combination
used is captured in cell 43,44,45 of the IPython notebook
![alt text][image9] ![alt text][image19]


#### 7. Histogram from above images

![alt text][image20] ![alt text][image21]




#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used a stack of 9 windows (sliding based on mean of previous window). I also maintain circular buffers to hold historic detections.
Both current and historic detections are used while fitting the polynomial. This helped with smoother lane detection in case I encountered windows with no points detected. I also apply some sanity checks based on histogram and checking if peaks fall within a reasonable range, else skip the frame.

Example of windows is captured in final video.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature for both lanes is calculated in `getCurvature`. Final result is average of left and right lane curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This is captured in the video

![alt text][video1]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/proj.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I spent a lot of time playing with thresholds and various ways of combining the various images (sobel, h/s channel etc). My final strategy was to combine in such a way so as to reduce the inter-lane noise...even if it reduces the signal (i.e, the lanes)

This made fitting/detection of pixel better. I could compensate was reduced rate of pixel detection by augmenting with historical data.

The model still has challenges if the there are sharp color gradients in the area between the lines (like in the hard challenge video).
I think dynamically adjusting various thresholds (s-channel,l-channel,grayscale,sobel) based on amount of pixels in resulting image could
be one approach that could help. 

Another approach could be to choose fitting either the left or right lane (based on which has a strong detection on histogram for the frame). And then extrapolate/offset pixels for the other lane.