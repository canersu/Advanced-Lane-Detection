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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test6.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[image7]: ./output_images/undistorted6.png "Undistorted"
[image8]: ./output_images/final_image6.png "Final"
[image9]: ./output_images/search_around6.png "Search Around"
[image10]: ./output_images/sliding_windows6.png "Sliding Windows"
[image11]: ./output_images/thresh_binary6.png "Binary Threshold"
[image12]: ./output_images/warped_binary6.png "Warped Binary"
[video1]: ./project_video.mp4 "Video"


### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the function `camera_calibration()`IPython notebook located in "./Advanced_Lane.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Distortion Correction

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

To correct image, I used the matrix and distortion coefficients which is a result from camera calibration function. Camera calibration step is needed to be used only one time, so we will undistord each images with those coefficients of `correct_distortion()` function which is located at 3rd code cell inside IPython notebook. After correct the distortion, the output image looks like as following:

![alt text][image7]

Distortion effects can easily be seen from the rear bumper of the white car and the left corner of mountain.

#### 2. Binary Threshold

I used a combination of color and gradient thresholds to generate a binary image. First I needed to smooth the image using Gaussian blurring to remove the high frequencies on image. Then I used sobel gradient(with thresholds 20-100) and HLS color convertion to extract the S channel(with thresholds 170-255). This process is done in `binary_threshold()` function which is located at 4th code block inside IPython cell. After I combined the outputs of binary thresholded white pixels and the result is as following:

![alt text][image11]

#### 3. Perspective Transform

The code for my perspective transform includes a function called `perspective()`, which appears in 5th code block of IPython cell. The `perspective()` function takes as inputs an binary thresholded image (`src_img`). Source and destination of the image is written hardcoded by inspecting the coordinates of test images and the varibles are selected as following:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 570, 470      | 200, 0        | 
| 260, 680      | 200, 680      |
| 720, 470      | 1000, 0       |
| 1050, 680     | 1000, 680     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image12]

#### 4. Find Lane Pixels and Fit Polynomial

Then I used histograms on a binary thresholded and warped image as it can be seen from IPyhton notebook `find_lane_pixels()` function which is located at 6th cell. I divided it 12 pieces horizontally and each window has a image.shape[0]/12 height. Width of window is 2xmargin size which specified 100 pixels. Each window is looking for white pixels on binary image both for left and right half of the images. When white pixels found greater than `minpix`(which is 50), then a window is drawn w.r.t height and width values, which is shown below:

![alt text][image10]

After sliding windows and related pixels found, I applied curve fitting algorithm which is a second order function passes through white pixels, as shown below:

![alt text][image5]

Once the curve is found, it is not necessery to repeat finding process. Better solution is searching the cluster of white pixels +-margin range. In notebook's 7th cell, there is `seach_around()` function which is responsible for that operation and output of the funtion looks like:

![alt text][image9]


#### 5. Radius of Curvature and Center of Vehicle

There are pixels to real world coefficients which are 30/720 for Y-dimension and 3.7/700 for X-dimension. After finding real world dimensions for specified curve, radius is calculated stated as in function `radius_curvature()` located at 9th cell of notebook file. After that, I calculated the horizontal center of image at bottom(image.shape[1]/2) which denotes the center of the car. And I calculated the center of lanes by again taking the bottom horizontal line left lane and right lane X-coordinates, with the formula (right_lane-left_lane)/2. The difference between the lane and car center determines the car's current position.

#### 6. Vizualization on Undistorted Image

I implemented this step as a function `final_output()` at 16th cell in notebook. The area between curvatures filled with green. Vehicle position, left curvature and right curvatures are printed top left on image in meters. The output of my pipeline is as follows:

![alt text][image8]

### 7. Pipeline (video)

To make this compact, I created a function name `video_pipeline()`(at 20th cell) which includes the steps above. Each video frame is processed and a final output image is drawn on original image.

Here's a [link to my video result](./output_video/project_video.mp4)

---

### Discussion

This pipeline works acceptable however it can fail at sharp turns, when a car bumps or too many shadows on road. Vertex must be selected very precisely, and sobel direction mask could be apply to get a better result. Visualization slows down the pipeline, and python is a heavyweight language. With using different moitoring system and C++ is chosen and also speed the process up with GPU would significantly improve the speed of the pipeline.
