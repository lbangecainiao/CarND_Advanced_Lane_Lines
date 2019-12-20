## Advanced Lane Finding Project**


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

[image1]: ./output_images/ChessboardUndistortion.jpg "Undistorted chessboard image"
[image2]: ./output_images/RoadImageUndistortion.jpg "Undistorted road image"
[image3]: ./output_images/BinaryTransformed.jpg "Binary transformed road image"
[image4]: ./output_images/PerspectiveTransformed.jpg "Perspective transformed"
[image5]: ./output_images/PolynomialFitted.jpg "Polynomial fitted"
[image6]: ./output_images/ImageAnnotated.jpg "Image annotated"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

The criterias and corresponding relizations will be presented in this file.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.  

The codes of the camera calibration appears between the line 12 and 42 in the `AdvanceLineFinding.ipynb`. 
I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. The image points are detected using the function `glob.glob()`  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code appears in the Helper functions section between lines 3 and 5 named `undistort()`.
Given the undistortion parameters computed off-line, the`cv2.undistort` function is implemented to obtain the undistortion image, which is shown as follows,
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code appears in the Helper functions section between line 7 and 26 named as `color_gradient_transform()`
I used a combination of color and gradient thresholds to generate a binary image.The image is transformed into HLS color space and gradient in x direction using the function `cv2.cvtColor` and `cv2.Sobel` separately. Then different upper and lower thresholds are applied to the saturation channel and x gradient. The points with corresponding value between the upper and lower limit will be assigned the binary value as 1. Then the results obtain from the s channel and x gradient are combined on the same binary plot.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code appears in the Helper functions section between line 28 and 37 named as `perspective_transform()`
The `perspective_transform()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:
  1.On a picture with straight road line I chose four source points along the left and right road lane.
  2.Then destination points are defined to form the rectangular. The lower two points have the same x coordinate with the lower two source points.

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 600, 445      | 240, 0        | 
| 680, 445      | 1060, 0      |
| 1060, 700     | 1060, 720      |
| 240, 700      | 240, 720       |

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code appears in the Helper functions section between line 42 and 117 named as `find_lane_pixels()`
The following steps are performed to find the lane pixels
1.First the histogram is found by summing the binary points. The approximate base position of the left and right road lanes are obtained.
2.Next, the sliding windows approach is implemented to identify the binary pixels related to the road lane.
  1.The number and width of the windows are set in the parameters `nwindows` and `margin`.Then the windows are plotted starting from the bottom of left and right road lane.
  2.Once a window is defined, the nonzero pixels in the window are also searched and added to the lists recording the left and right road lane `left_lane_inds`,`right_lane_inds`.
  3.If the number of pixels found in the window is larger than the minimum number of pixels in `minpix`, the left and right lane position will be moved to the average value of the left and right lane pixels in the x direction.
  4.Then the position of the left and right pixels in the x and y coordination are extracted and store in array `leftx`,`lefty`,`rightx`,`righty`.
3.Finally, the polynomial functions are fitted for the left and right road lane according to the following steps,the code appears in the Helper functions section between line 119 and 156 as function `fit_polynomial()`
  1.First the parameters of a second order polynomial function are obtained using `np.polyfit()` function for left and right road lane.
  2.Then the x value is computed for the whole range y value from bottom to top using the parametes obtained.
  3.At last the left and right lines are plotted back to the figure.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code appears in the Helper functions section between line 158 and 173 named as `measure_curvature_real()`
First the conversion ratio from pixels space to meters for x and y direction are defined. Then the left and right lane curvature are computed implementing the curvature equation. At last the current curvature is computed by averaging the left and right curvature.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code appears in the Helper functions section between line 175 and 205 named as functions`project_lines()` and `annotate_image()`.
1.First the pixels of left and right lane obtained are exploited to plot a polygon using the function `cv2.fillPoly()`. Then the inverse perspective transform is implemented to transform the birds view plot back to the original plot.
2.Then the curvature and offset are annotated on the figure.


![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video 

Here's a [link to my video result](./output_images/ProjectLines.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

1.The selection of the source points during the perspective transform has a large impact on the results. A proper criteria to select the source points should be established.

2.A better sturcture of the codes can be adopted. Codes of different functions might be stored in different files.

3.The parameters(threshold of s channel etc.) are tuned only for a certain road condition. The pipeline might fall for other road conditions.
