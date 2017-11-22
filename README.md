## Advanced Lane Finding Project 4
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)



The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### Camera Calibration

Camera calibration code can be found in the first cell of the jupyter notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function to obtain the undistorted images.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Please check the second cell of the notebook. All images are distortion corrected and printed with the left side being original images, and the right side have the distortion corrected images. Pay attention to the hood of the car, where it is the most visible.


#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Perspective transform is done by a function called `birdseyeView()` in  the notebook `carndp4.ipynb` , which is the third cell of the notebook.  The `birdseyeView()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src=np.float32([(580,460),(205,720),(1110,720),(703,460)])

dst=np.float32([(320,0),(320,720),(960,720),(960,0)])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. The result can be seen in the end of the cell.


#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Color and gradient filtering is by far the hardest part of this project. There are many options to choose from like using different colorspaces, using sobel operators and using directional filtering. I'd like to thank forum moderator Subodh for his recommendations.

In summary, I used RGB and HSL color spaces with Sobel X direction derivative filtering and directional filtering using Sobel X and Y derivatives. In the notebook, an example is provided in the end of cell 4. "test5.jpg" is a very hard input image to filter since it has lots of dark shadows, as well as asphalt color changes.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Cells 5 and 6 include the lane line identifying algorithms. First algorithm is called `newlinepolyfit()`  and it finds the lane lines by using histogram method on the x axis with sliding windows.

Then there is `contlinepolyfit()` function, which is used in case there was a successful detection from the `newlinepolyfit()` function and in the next frame of the video, in order to save computing power, `contlinepolyfit()` takes the previous line fit information `left_fit` and `right_fit`  and computes the new fit information.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code can be found in the 7th cell of the notebook. `findradius()` function takes in the line fits as well as left and right non-zero x and y points matrices.  This function also outputs the distance of the car from the center of the lane.

Radius of the curvature is calculated by the formula
 `f(y)=Ay​​+By+C`
 
 And the assumptions in order the convert pixels into meters are:
 
`ym_per_pix = 30/720 # meters per pixel in y dimension
 xm_per_pix = 3.7/700 # meters per pixel in x dimension`
Which means 720 pixels equals to 30 meters in y axis and 700 pixels equals 3.7 meters in x axis.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Cell 8 of the notebook has an example of the projected lane line detection. It is constructed using `cv2.fillPoly()` function.

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Hardest part of the project was the image filtering function `colormagThresh()` . My initial approach was to use HSL colorspace by extracting the S channel and using Sobel X derivative to filter out horizontal lines.

Although this approached worked for most test images, when there is a color change occurs on the road with lots of shadows the pipeline had hard time finding the lines. 

Then I started reading the udacity forums, and had a chance to experiment with Subodh's algorithm, where gray,R,G,S,L color channels and Sobel x operator. This way, the function had no problem detecting the lines in shadowy or darker colored roads.

For the challenge video, I need an additional sanity check for the lane line detection algorithms. Without sanity checks, when `np.polyfit()` function returns no coefficients for the line fit, the video pipeline freezes. This would be a problem when the roads are windy and line of sight is not far. For this problem, I also need to implement a function to find the warp src and dst image points. This can probably done by using OpenCV houghlines function to find out the lane lines roughly and using the X,Y coordinates of the lines.