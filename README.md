﻿﻿﻿﻿﻿﻿﻿﻿﻿Advanced Lane Finding Project

---
### The goals of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/chessboard_corners.png "Finding Chessboard Corners"
[image2]: ./output_images/chessboard_distortion_correction.png "Checking Distortion Correction with Chessboard"
[image3]: ./output_images/undistorted_images.png "Distortion Correction Images"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

----
### Here I will consider the points individually and describe how I addressed each point in my implementation:

---

### 1. Camera Calibration
(cell 4 in the notebook)

In this first step we need to calibrate our camera using the calibration chessboard images provided in the repository. The correct camera matrix and distortion coefficients can be calculated using OpenCV functions 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Drawing and displaying the corners resulted in:

![][image1]

### 2. Distortion correction
(cell 5 + 6 in the notebook)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

`    ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, test_image.shape[0:2], None, None)

     undist = cv2.undistort(test_image, mtx, dist, None, mtx)`

![][image2]

After applying the undistort fuction to the test image I undistorted the images that a car on the road took:

![][image3]

### 3. Perspective transformation
(cell 7 - 9 in the notebook)

After all the given images are undistorted we apply a perspective transformation. We will again use OpenCV functions to rectify each image to a "birds-eye view".  
We first want to identify four source points for our perspective transform. I picked four points in a trapezoidal shape that would represent a rectangle when looking down from above. After defining my source and destination points I warped the image using this code:

`   M = cv2.getPerspectiveTransform(src, dst)

    #Also calculate reverse transform matrix
    
    Mrev = cv2.getPerspectiveTransform(dst, src)
    
    #Apply the matrix
    
    warped = cv2.warpPerspective(img, M, (width, height), 
                                 flags=cv2.INTER_LINEAR)`


I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image:





### 4. Color Transforms + Gradients
(cell 11-13 in the notebook)

I decided to look at different color spaces here; RGB and HLS. I splitted the image into the different channels. These are the results:



You'll want to try out various combinations of color and gradiënt thresholds to generate a binary image where the lane lines are clearly visible. When we look at the images we can conclude that a combination of the S and L channel will probably give us the desired result. So the next step is to combine those. This is the combined binary image:




### 5. Locate the Lane Lines
(cell 15-17 in the notebook)

After applying calibration, thresholding, and a perspective transform to a road image, we have a binary image where the lane lines stand out clearly. However, we still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.

Plotting a histogram of where the binary activations occur across the image is one potential solution for this:

`    histogram = np.sum(combined[combined.shape[0]//2:,:], axis=0)
    plt.plot(histogram)`

This is the histogram:



We can use the two highest peaks from our histogram as a starting point for determining where the lane lines are, and then use sliding windows moving upward in the image (further along the road) to determine where the lane lines go. The first ste[ is to split the histogram into two sides, one for each lane line. Our next step is to set a few hyperparameters related to our sliding windows, and set them up to iterate across the binary activations in the image. Now that we've set up what the windows look like and have a starting point, we'll want to loop for nwindows, with the given window sliding left or right if it finds the mean position of activated pixels within the window to have shifted. When we have found all our pixels belonging to each line through the sliding window method, it's time to fit a polynomial to the line.




### 6. Measuring curvature and the position of the vehicle
(cell 19 in the notebook)

Here the idea is to take the measurements of where the lane lines are and estimate how much the road is curving and where the vehicle is located with respect to the center of the lane. I defined two functions to measure the curvature and the position of the vehicle.



### 7. Final step: drawing the lines on the original image
(cell 21 + 22 in the notebook)

The fit from the rectified image has been warped back onto the original image and plotted to identify the lane boundaries. This is the final result:




---
## Pipeline (video)
(cell 23-26 in the notebook)



Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  









