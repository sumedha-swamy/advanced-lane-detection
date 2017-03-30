Sumedha Swamy (<sumedhaswamy@gmail.com)>

The goals / steps of this project are the following:

-   Compute the camera calibration matrix and distortion coefficients
    given a set of chessboard images.
-   Apply a distortion correction to raw images.
-   Use color transforms, gradients, etc., to create a thresholded
    binary image.
-   Apply a perspective transform to rectify binary image ("birds-eye
    view").
-   Detect lane pixels and fit to find the lane boundary.
-   Determine the curvature of the lane and vehicle position with
    respect to center.
-   Warp the detected lane boundaries back onto the original image.
-   Output visual display of the lane boundaries and numerical
    estimation of lane curvature and vehicle position.

[//]: # (Image References)
[DistortionCorrected]: ./Report/DistortionCorrected.png
[ProcessedImage]: ./Report/ProcessedImage.png
[GradientAndColorThresholds]: ./Report/GradientAndColorThresholds.png
[calibration]: ./Report/calibration.png
[PerspectiveTransform]: ./Report/PerspectiveTransform.png
[laneLines]: ./Report/laneLines.png
[equation]: ./Report/equation.png


Camera Calibration
==================

### Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the
IPython notebook located in "AdvancedLaneDetection.ipynb" in the
function `calibrate_camera()`.

I started by preparing "object points", which will be the (x, y, z)
coordinates of the chessboard corners in the world. Since the chessboard
is fixed on the (x, y) plane I set z=0. The object points are the same
for all calibration images. I therefore set objp to coordinates such as
0,0,0), (1,0,0), (2,0,0) ....,(8,5,0).

Next, I read the calibration chessboard images and converted them to
grayscale. I used OpenCV `cv2.findChessboardCorners()` to detect chessboard
corners. I saved the (x,y) coordinates of the detected chessboard
corners in imgpoints. I then used OpenCV `cv2.calibrateCamera()` function
to compute the distortion coefficients. I used the these coefficients in
OpenCV `cv2.undistort()` function to undistort images as seen in the
picture below ![calibration]:

Pipeline (test images)
======================

### Provide an example of a distortion-corrected image.

The below picture shows an example of the two images and the effect of
applying distortion correction to them:
![DistortionCorrected]


### Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.

I used a combination of Gradient Thresholds (X gradient, Y gradient,
Magnitude gradient and direction gradient thresholds) and color
thresholds (HLS thresholds) to transform the image to a binary result.
The function 'gradient_color_threshold()' in the
'AdvancedLaneDetection.ipynb' file contains the code where I combine the
various thresholds. The AdvancedLaneDetection.ipynb also contains
various test images to show the result of the various thresholds. The
image here is a sample image with thresholds applied along with a mask
to focus on the area of interest. ![GradientAndColorThresholds]

### Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

In the AdvancedLaneDetection.ipynb notebook, I’ve defined the Perspective
Transform in the function 'warp_image()``. I used the following source and
destination points that I chose through visual inspection to generate
the transformation and inverse transformation matrices:

\# Source and Destination points for Perspective transform

src = np.float32(\[\[437,590\],\[945,590\],\[523,525\],\[833,525\]\])

dst = np.float32(\[\[450,590\],\[900,590\],\[450,505\],\[900,505\]\])

I used the OpenCV function getPerspectiveTransform() to calculate the
transformation matrix. I used warpPerspective() to transform the image
perspective.
![PerspectiveTransform]

### Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I identified the lane lines and fit a polynomial to approximate the
lanes in the function `find_lanes()` in the AdvancedLaneDetection.ipynb
notebook. I followed the following steps to identify the lane lines:

1.  Given that lane lines are somewhat
    vertical after applying a perspective transform, I took a histogram
    of the pixels distribution in the binary image and identified the
    peaks in the two halves of the image – The peaks in the left and
    right halves corresponded to the left and right lanes.

2.  I defined a sliding window around the detected peaks. The sliding
    window has a margin of 70 and 9 sliding windows will cover the
    height of the image.

3.  I looked for pixels in a sliding window and if there were more than
    50 pixels in a window, I took the mean of those pixels and set the
    position of the next window to the mean of those pixels. I continued
    this process for all 9 sliding windows on left and right lane lines.

4.  Next, I fit those points to a 2^nd^ order polynomial using
    `np.polyfit()`for each of the left and right lane lines.

The process is illustrated in the image. ![laneLines]

### Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

**Radius of Curvature:**

I calculated the radius of curvature of the road in the function
radius\_of\_curvature(). After calculating the polynomial fit for the
left and right lane lines, I fit those points to a new polynomial – this
time converting those pixels to meters. Next, I used the formula

Radius of curvature = ![equation]

Reference:
<http://www.intmath.com/applications-differentiation/8-radius-curvature.php>

**Position of vehicle:**

In `process_image()` function, I have calculated the position of the
vehicle with respect to the center. In this case, I’ve assumed that the
car camera is mounted in the center. Therefore, I have calculated the
deviation of the mid point of the lane from the center of the image. The
below three lines of code summarize my calculations:
````
lane_middle = (right_lane + left_lane) / 2.0

car_position = image.shape[1]/2

deviation_from_middle = (car_position - lane_middle)*xm_per_pix
````
### Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

![ProcessedImage]


Discussion
===========
### Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

Some of the ways I could improve this model are:

- Identifying source and destination points for perspective transform can be automated. This is a critical step in the pipeline and manually determining the points can be prone to error.
- I’ve currently only used the “S” channel from the HLS image transform to help identify lanes. I can use more color information from other channels and transforms because as humans we use more information than just “saturation” to identify lanes as we drive.
- My pipeline performs poorly when there are a lot of trees at the side of the road. That is because the thick set of trees will compete with the lane lines in the pixel histogram of a binary image. My pipeline must more accurately identify straight lines and reject foliage.
- My pipeline doesn’t use information about the size of the road (eg. width) more aggressively.
