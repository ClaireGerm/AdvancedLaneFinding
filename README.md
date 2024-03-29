### Advanced Lane Finding Project

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

[image1]: ./output_images/findcorners.png "Find Chessboard Corners"
[image2]: ./output_images/chessboarddistortion.png "Distortion Correction with Chessboard"
[image3]: ./output_images/roaddistortion.png "Distortion Correction on the Road"
[image4]: ./output_images/warped.png "Warp Image"
[image5]: ./output_images/channels.png "Color Channels"
[image5a]: ./output_images/hls.png "Color Channels 2"
[image6]: ./output_images/binary.png "Binary Output"
[image7]: ./output_images/histogram.png "Histogram"
[image8]: ./output_images/lines.png "Sliding Window Result"
[image9]: ./output_images/finalfinalroad.png "Final Result"


----
### Here I will consider the points individually and describe how I addressed each point in my implementation:

---

### 1. Camera Calibration
(cell 4 in the notebook)

In this first step we need to calibrate our camera using the calibration chessboard images provided in the repository. The correct camera matrix and distortion coefficients can be calculated using OpenCV functions 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  Drawing and displaying the corners resulted in:

![alttekst][image1]

### 2. Distortion correction
(cell 5 + 6 in the notebook)

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

`ret, mtx, dist, rvecs, tvecs = cv2.calibrateCamera(objpoints, imgpoints, test_image.shape[0:2], None, None) undist = cv2.undistort(test_image, mtx, dist, None, mtx)`

![alttekst][image2]

After applying the undistort fuction to the test image I undistorted the images that a car on the road took:

![alttekst][image3]

### 3. Perspective transformation
(cell 7 - 9 in the notebook)

After all the given images are undistorted we apply a perspective transformation. We will again use OpenCV functions to rectify each image to a "birds-eye view".  
We first want to identify four source points for our perspective transform. I picked four points in a trapezoidal shape that would represent a rectangle when looking down from above. After defining my source and destination points I warped the image using this code:

`M = cv2.getPerspectiveTransform(src, dst)

 #Also calculate reverse transform matrix
    
 Mrev = cv2.getPerspectiveTransform(dst, src)
    
 #Apply the matrix
 warped = cv2.warpPerspective(img, M, (width, height), 
                                 flags=cv2.INTER_LINEAR) `

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image:

![alttekst][image4]



### 4. Color Transforms + Gradients
(cell 11-13 in the notebook)

I decided to look at different color spaces here; RGB and HLS. I splitted the image into the different channels. These are the results:

![alttekst][image5]
![alttekst][image5a]

You'll want to try out various combinations of color and gradiënt thresholds to generate a binary image where the lane lines are clearly visible. When we look at the images we can conclude that a combination of the S and L channel will probably give us the desired result. So the next step is to combine those. This is the combined binary image:

![alttekst][image6]


### 5. Locate the Lane Lines
(cell 15-17 in the notebook)

After applying calibration, thresholding, and a perspective transform to a road image, we have a binary image where the lane lines stand out clearly. However, we still need to decide explicitly which pixels are part of the lines and which belong to the left line and which belong to the right line.

Plotting a histogram of where the binary activations occur across the image is one potential solution for this:

`    histogram = np.sum(combined[combined.shape[0]//2:,:], axis=0)
    plt.plot(histogram)`

This is the histogram:

![alttekst][image7]

We can use the two highest peaks from our histogram as a starting point for determining where the lane lines are, and then use sliding windows moving upward in the image (further along the road) to determine where the lane lines go. The first step is to split the histogram into two sides, one for each lane line. Our next step is to set a few hyperparameters related to our sliding windows, and set them up to iterate across the binary activations in the image. Now that we've set up what the windows look like and have a starting point, we'll want to loop for nwindows, with the given window sliding left or right if it finds the mean position of activated pixels within the window to have shifted. When we have found all our pixels belonging to each line through the sliding window method, it's time to fit a polynomial to the line.

![alttekst][image8]

### 6. Measuring curvature and the position of the vehicle
(cell 19 in the notebook)

Here the idea is to take the measurements of where the lane lines are and estimate how much the road is curving and where the vehicle is located with respect to the center of the lane. I defined two functions to measure the curvature and the position of the vehicle.



### 7. Final step: drawing the lines on the original image
(cell 21 + 22 in the notebook)

The fit from the rectified image has been warped back onto the original image and plotted to identify the lane boundaries. This is the final result:

![alttekst][image9]


---
## Pipeline (video)
(cell 23-26 in the notebook)

Here's a [link to my video result](./output_images/project_video_output.mp4)

---

### Discussion

Working oh this project went pretty good. Sometimes looking at all the code was a bit overwhelming so I tried to solve everything step by step. It was a very educational project!

When we compare this project to the first project "Finding Lane Lines" we can conclude that this method is far more effective because we can locate turns now. We don't know if this will work on very sharp turns though. Another potential shortcoming is the detection in other weather conditions. The given video is made in perfect sunny weather. What are we going to do with snow or rain? Maybe we can solve those problems by combining channels from different color spaces. 
 









