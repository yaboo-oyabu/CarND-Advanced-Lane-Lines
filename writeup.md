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

[imageC]: ./output_images/undistort_output_checkboard.png "Camera Calibration"
[image1]: ./output_images/undistort_output_0.png "Undistortion"
[image2]: ./output_images/binary_output_0.png "Binary Threshold"
[image3]: ./output_images/ptransform_output_0.png "PTransform"
[image4]: ./output_images/lane_output_0.png "Warp Example"
[image6]: ./output_images/overlay_output_0.png "Fit Visual"

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first section of "Camera Calibration" of the IPython notebook located in "./P2.ipynb". There are two functions: `compute_camera_calibration()` and `apply_undistortion()`. `compute_camera_calibration()` estimates `camera_matrix` and `dist_coeffs` from chessboard images, and `apply_undistortion` undistors an input image with `cv2.undistort` which requires `camera_matrix` and `dist_coeffs`.

In the above process, the estimatiion of `camera_matrix` and `dist_coeffs` is the most important because it determines the performance of undistortion. To estimate these parameters, `objpoints` and `imgpoints` are needed. `objpoints` is a list of `objp`, which is array of the (x, y, z) coordinates of the chessboard corners in the calibration pattern coordinate space. Here, z is set to 0 because I am assuming the chessboard is fixed on the (x, y) plane. `imgpoints` is a list of `imgp`, which is array of (x, y) coordinates of chessboard corners in chessboard images. Here, chessboard corners are detected by `cv2.findChessboardCorners()`. After preparing `objpoints` and `imgpoints`, I estimate `camera_matrix` and `dist_coeffs` with `cv2.calibrateCamera()`.

The following image shows how a chessboard image is undistorted by `apply_undistortion()` with estimated `camera_matrix` and `dist_coeffs`:

![alt text][imageC]

### Pipeline (single images)
#### 1. Provide an example of a distortion-corrected image.

This step is implemented in the first sub-section of "Pipeline" section of the IPython notebook located in "./CarND_Advanced_Lane_Lines.ipynb". To demonstrate the distortion-correction step, I correct the distortion of the images which located in `test_images/straight_lines2.jpg`. I applied `apply_undistortion()` to this image and get the following result. In the follwing result, the left image is the input to `apply_undistortion()` and the right image is the output. You can see the red car in the right image is streched horizontally compared to the left.

![alt_text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

This step is implemented in the second sub-section of "Pipeline" section of the IPython notebook located in "./CarND_Advanced_Lane_Lines.ipynb". In this step, `apply_binary_transformation_with_thresholds()` is used to create a thresholded binary image. I used a combination of gradient and color thresholds to generate a binary image. The gradient threshold is applied to the gray-scale image which is the output of `cv2.Sobel()`. This calculates x derivatives of an input image. The color threshold is applied to the HLS color image which is converted from an input image by `cv2.cvtColor()`. Here is an example of a thresholded binary image converted by `apply_binary_transformation_with_thresholds()`.

![alt text][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

This step is implemented in the third sub-section of "Pipeline" section of the IPython notebook located in "./CarND_Advanced_Lane_Lines.ipynb". In this step, `apply_perspective_transform()` is used to apply a perspective transform to an input image. This function takes a transform matrix which is calculated by `calculate_perspective_transform_matrix()`. To obtain a transform matrix, I used `cv2.getPerspectiveTransform()` and input source (`src`) and destination (`dst`) points to that function. I chose the hardcode the source and destination points as follows.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 203, 720      | 320, 0        | 
| 581, 460      | 320, 720      |
| 703, 460      | 960, 720      |
| 1110, 720     | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This step is implemented in the forth sub-section of "Pipeline" section of the IPython notebook located in "./CarND_Advanced_Lane_Lines.ipynb". In this step, `fit_polynomial` takes left points (`leftx` and `lefty`) and right points (`rightx` and `righty`) to estimate second order polynomials for left and right lane. These points are extracted by `find_lane_pixels()` which has important steps to separate left and right points without outliars. In the first step, I calculate `midpoint` of the lane based on left and right peaks of the `histogram`, and use it to split points to left and right groups. Then, outliars are removed from each group by using sliding window technique. Since this function is slow due to it's computational complexity, I also define `search_around_poly()` which reuse previously calcuated coefficients of second order polynomials to reduce the computational complexity for the extraction of left and right points. 

![alt text][image4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This step is implemented in the fifth sub-section of "Pipeline" section of the IPython notebook located in "./CarND_Advanced_Lane_Lines.ipynb". In this step, `measure_curvature_and_position()` returns the radius of curvature of the lane with `measure_curvature()`, and the position of the vehicle with respect to center with `measure_position()`. 

`measure_curvature()` uses the following formula to calcute the radius of curvature ![$R_{curve}$](https://latex.codecogs.com/gif.latex?R_{curve}). Here ![$A$](https://latex.codecogs.com/gif.latex?A) and ![$B$](https://latex.codecogs.com/gif.latex?B) are first and second order coefficients of the fitted lane line respectively, and ![$y$](https://latex.codecogs.com/gif.latex?y) is y coordinate of the vehicle in the image. Since the image is camera view, ![$y$](https://latex.codecogs.com/gif.latex?y) is set to the height of the image. 

![alt text](https://latex.codecogs.com/gif.latex?R_{curve}&space;=&space;\frac{(1&space;&plus;&space;(2Ay&space;&plus;&space;B)^2)^{\frac{3}{2}}}{|2A|})


`measure_position()` uses the following equation to calculate the the position of the vehicle with respect to center ![$D_{center}$](https://latex.codecogs.com/gif.latex?D_{center}). Here ![$x_{left}$](https://latex.codecogs.com/gif.latex?x_{left}) and ![$x_{right}$](https://latex.codecogs.com/gif.latex?x_{right}) is x coordinates of the left and right lane line respectively, ![$(x_{left} + x_{right})/2$](https://latex.codecogs.com/gif.latex?(x_{left}+x_{right})/2) means x coordiate of the lane center. ![$x_{vehicle}$](https://latex.codecogs.com/gif.latex?x_{vehicle}) is x coordinate of the position of the vehicle, which is the same as x coordinate of the center of the image. 

![alt text](https://latex.codecogs.com/gif.latex?D_{center}&space;=&space;\frac{x_{left}&space;&plus;&space;x_{right}}{2}&space;-&space;x_{vehicle})

In default, the unit of output from `measure_curvature_and_position()` is pixel, but you can change it by `x_per_pix` and `y_per_pix`. In `measure_curvature_and_position_in_meters()`, I set meters per pixel to `x_per_pix` and `y_per_pix`, and get the radius of curvature and the position of the vehicle in meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This step is implemented in the sixth sub-section of "Pipeline" section of the IPython notebook in "./CarND_Advanced_Lane_Lines.ipynb". In this step, `overlay_detected_lane()` first creates a polygon with points of left and right lanes, and projects the polygon to the undistorted road image by `cv2.perspectiveTransform()`. Then it draws the polygon with `cv2.fillPoly()` and overlays it to the undistorted road image by `cv2.addWeighted()`. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
