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

[image1]: ./output_images/undistorted_chessboard.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./output_images/Thresholded_image.png "Thresholded image"
[image4]: ./output_images/Thresholded_warped.png "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/final_result.png "Final output image"
[image7]: ./output_images/Original_Undistorted.png "Undistorted test image"
[image8]: ./output_images/histogram.png "Histogram"
[image9]: ./output_images/sliding_window.png "Sliding Window"
[image10]: ./output_images/searchable_area.png "Search region for lane lines"
[video1]: ./videos/project_video_test.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "P4-SDCND.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
The code for this step is contained in the fourth cell of Ipython notebook. We first read the image using `mpimg.imread` and then use `cv2.undistort()` to undistort the image. The undistorted image can be seen as below:
![alt-text][image7]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in cells 5 through 6 in `P4-SDCND.ipynb`). I gray scaled the image using `cv2.cvtColor()` and then calculated its Laplacian. Then I applied gradient thresholds to the image. For color thresholding, I used the s-channel, and created a binary image. Finally, I combined the gradient and color thresholds.
 Here's an example of my output for this step.
![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes functions called `find_perspective_points()` and `get_perspective_transform`, which appears in code cells 5 through 6 in the file `P4-SDCND.ipynb`.  The `find_perspective_points()` function takes as inputs an image (`img`), and generates appropriate source (`src`) and destination (`dst`) points. The function `get_perspective_transform()` takes as input an image (`img`) and source (`src`) and destination (`dst`) points, and generates the M and Minv matrix. These matrices are then used to get the warped image using the function `cv2.warpPerspective()`.

If the function `find_perspective_points()` fails to find source (`src`) and destination (`dst`) points, then I chose to hardcode the source and destination points in the following manner:

```
src = np.array([[585. / 1280. * img_size[1], 455. / 720. * img_size[0]],
                        [705. / 1280. * img_size[1], 455. / 720. * img_size[0]],
                        [1130. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [190. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)

dst = np.array([[300. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [1000. / 1280. * img_size[1], 100. / 720. * img_size[0]],
                        [1000. / 1280. * img_size[1], 720. / 720. * img_size[0]],
                        [300. / 1280. * img_size[1], 720. / 720. * img_size[0]]], np.float32)
```
I verified that my perspective transform was working as expected by plotting the original image and warped image and verifying that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Once i got the binary warped image, I take a histogram along all the columns in the lower half of the image like this:

![alt text][image8].

With this histogram I am adding up the pixel values along each column in the image. In my thresholded binary image, pixels are either 0 or 1, so the two most prominent peaks in this histogram will be good indicators of the x-position of the base of the lane lines. I can use that as a starting point for where to search for the lines. From that point, I can use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame.
The sliding window approach is implemented in the function `sliding_window()`. After implementing sliding window I used the function `searchable_area()` which calculates the area in which we should search lane lines. The functions `sliding_window()` and `searchable_area()` are implemented in the code cells 7 through 8 of the Ipython notebook `P4-SDCND.ipynb`.
My ouptput looks as follows:

![alt text][image9]

![alt text][image10]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Once we fit a second order polynomial to the lane lines, radius of curvature can be easily calculated. The position from center is also calculated along with this by calculating the x values for left fit and right fit, and then taking the mean.
I did this in code cell 9 in the function `compute_rad_curv()` in my code in `P4-SDCND.ipynb`.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code cell 12 in my code in `P4-SDCND.ipynb` in the function `process_image()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](.videos/project_video_test.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Some of the issues that I faced are as follows:
* Applying color and gradient thresholds: Choosing the correct values for thresholds is a challenge as it greatly effects the amount of noise in the binary image.
* While doing perspective transform, I struggled in choosing appropriate source and destination points.
* I faced difficulty in optimizing my search for lane lines in subsequent video frames.

To make it more robust, I could do following things:
* Use a low-pass filter to smooth the lane detection over frames.
* Although I have implemented outlier rejection, I think there can be a better way to do that.
