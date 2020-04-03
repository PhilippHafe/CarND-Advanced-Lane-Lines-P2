## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---
**Advanced Lane Finding Project**

The goals / steps of this project are the following:

1. Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
2. Apply a distortion correction to raw images.
3. Use color transforms, gradients, etc., to create a thresholded binary image.
4. Apply a perspective transform to rectify binary image ("birds-eye view").
5. Detect lane pixels and fit to find the lane boundary.
6. Determine the curvature of the lane and vehicle position with respect to center.
7. Warp the detected lane boundaries back onto the original image.
8. Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/Undistorted_Calibration_Image.jpg "Undistorted Cal Image"
[image2]: ./output_images/Undistorted_Image.jpg "Undistorted Road Image"
[image3]: ./output_images/Color_Gradient_Threshold_1.jpg "Combined Binary"
[image4]: ./output_images/warp_1.jpg "Warp Example"
[image5]: ./output_images/lanepixels_visualized_1.jpg "Lane Pixels Visual"
[image6]: ./output_images/result.jpg "Output Frame"
[video1]: ./project_video_out.mp4 "Output Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The computation of the camera matrix and distortion coefficients is done by determining the image and object points from a set of calibration (chessboard) images. In my program the function is called `processCalibrationImages`
To identify the chessboard corners openCVs function `findChessboardcorners` is used. If all the inner corners in a single calibration image have been found as (x,y)-coordinates, these are added to an image point list.
At the same time, an object point list holding object points in the format (x, y, z) is created and appended. 
The image and object point list are then used to undistort the raw images from the video, which is done in `undistort_image`.
Here you can see an example of an undistorted calibration image:
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The object and imagepoint list from the calibration images are used on every single image within the function `undistort_image`. This function takes in an image, the image and object point list from the calibration images and returns an undistorted image.
The steps necessary to accomplish this are
* Determine the camera matrix and distortion coefficients using `cv2.calibrateCamera`
* Undistort the image by applying the camera matrix and distortion coefficients to the image with the help of `cv2.undistort`
Here you can see an example of an undistorted test image.
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of gradient and color thresholds to create a binary thresholded image.
I first defined subfunctions for creating binarys from one single gradient or color:
* `abs_sobel_thresh`: applies a Sobel in x or y direction with specified kernel size and threshold for creating a binary based on either of the directions
* `mag_threshold`: applies a Sobel in x and y direction with specified kernel size and threshold for creating a binary based on x and y direction gradients
* `dir_threshold`: applies a thresholding based on the angle between the gradients in x and y direction using an $arctan$ and creating a binary based on specified thresholds
* `hls_select`: converts the image to HLS color space and creates a binary from a threshold of the S channel

In a second step, these functions are combined in the function `create_binary` to create one binary ouptut image. This functions takes all the inputs required for the above mentioned subfunctions. In the code, the thresholds are found in the **Main**-Section of the [Project Notebook](./P2.ipynb), where the `create_binary`-function is called (line 16 of that cell).
The thresholds are as follows:
* kernel_size = 9
* gradient_thresh = (60, 200)
* mag_thresh = (60, 200)
* dir_thresh = (0.7, 1.3)
* color_thres = (205, 255)
Below is an example image that shows the applied gradients and the combined binary.
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The perspective transform is implemented in the function `warp_image` in the **Step4**-Section of the [Project Notebook](./P2.ipynb)  This function takes as input an image as well as source (`src`) and destination (`dst`) points. The function is called within the **Main**-section of the project notebook with the following source and destination points:

`bottomleft=(192, 720)
topleft=(576, 460)  
topright=(707, 460) 
bottomright=(1135, 720)
inside = 300

src = np.float32([bottomleft, topleft, topright, bottomright])
dst = np.float32([[inside, imshape[1]], [inside, 0], [imshape[0]-inside, 0], [imshape [0]-inside, imshape[1]]])`

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I did this in the cell in the **Step 5**-Section of the [Project Notebook](./P2.ipynb).
In this cell, all the necessary functions are defined:
* `get_activated_pixels`: takes in an image and returns the indices of the nonzero pixels
* `get_lane_pixels_from_indices`: takes in the indices of the left and right lane pixels and returns the arrays that only contain the pixel indices / coordinates
* `fit_poly`: takes in pixel coordinates for left and right lane and performs a polynomial fit. It returns the coefficients for the left and right lane polynom as well as arrays containing the coordinates of the pixels on the polynom.
* `get_indices_from_poly`: takes in the coefficients of a polynom as well as all the activated pixels and a margin. It then finds indices / coordinates of pixels in the image that are within the margin (in x-direction) from the polynom specified by the coefficients.
* `get_indices_from_window`: applies a "window search method" to find the lanes by looking at the activated pixels within a "windowed area" of the picture. It takes in different parameters for the shape and number of the windows and outputs the indices / coordinates of pixels that are within the windows.
* `draw_pixels`: takes in the pixel indices and outputs an image where the pixels of the left lane are coloured in red and the ones of the right lane are coloured in blue
* `draw_poly`: a function that takes in x and y values of the left and right lane lines and plots them as a yellow line on the current figure
* `visualize_lane2`: This function visualizes the found lane lines by coloring the left lane line pixels in red, the right lane line pixels in blue and filling the area between the two lane lines with green color
![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the cell in the **Step 6**-Section of the [Project Notebook](./P2.ipynb).
The function is called `measure_curvature_and_laneposition` and takes in the pixel coordinates of the polynomial function as well as two scaling factors that specifiy the distance in meters per pixel in x and y-direction.
The pixel coordinates are then transformed to meters and fittet with a polynomial. The coefficients of this polynomial are then used to calculate the radius in meters for the left and right lane line. The function outputs the average of both.
The second part of the function calculates the x coordinate of the lane at the bottom of the picture and substracts it from the center coordinate (i.e. the bottom middle of the picture). This pixel distance is then converted back to meters to give the position relative to the lane center.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the main function, that can be found in the cell "Main Section" in the  [Project Notebook](./P2.ipynb). The detected lane lines that were visualized in the warped image (see point 4 of this writeup_template) have no to be "unwarped" / warped back to the original coordinates. Therefore the `warp_image` function is called with exchanged `src` and `dst` arrays. After that the resulting unwarped visualization is overlayed with the undistorted image using the `cv2.addWeighted` function. Last but not least, the calculated curvature and lane position are put onto the image to retrieve the final result. Here is an example of my result on a test image:
![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

I faced an issue with detecting lane lines in seconds 20-25 of the project video since the street surface is very light and the yellow line to the left side is not clearly detectable (compared to the other parts of the video). The detected lane line jumped to the roadway edge of the concrete barrier. In this time interval, the gradient images are not really helpful, but the S color channel is. However, the lower threshold value for S thresholding needs to be adapted. In my pipeline it is now 120, and this setting works fine.

The pipeline is likely to fail on difficult lighting conditions, e.g. through bridges and tunnels or very light street surface, since the parameters for creating the binary are fixed values. You could account for that by preprocessing the image and determining an overall brightness that is the input for a function to adopt the thresholds.
Also, my pipeline has difficulties if lane markings disappear for some reason, since it is not performing any moving average . of previously detected lanes. This could be implemented in order to make the pipeline more robust.

I would like to improve the following things:
* Implement a line class 
   * to reduce the number of in and ouputs from my functions
   * to store the coefficients of the line polynomials from previous matches in order to perform best fit and sanity checks
* Improve the window search method for turns with small curvature values ("tight turns")
* Introduce masking of the binary depending on the previous determined curvature. This would help to "shift" the focus of the lane finding algorithm depending on the curvature
