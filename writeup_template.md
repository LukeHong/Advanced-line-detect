## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

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

[image1]: ./examples/undistort.png "Undistorted"
[image2]: ./examples/thresh.png "Binary Example"
[image3]: ./examples/warp.png "Warp Example"
[image4]: ./examples/line_histogram.png "Line Histogram"
[image5]: ./examples/window_fitting.png "Fit Example"
[image6]: ./examples/output.png "Output"
[image7]: ./examples/undistorted.png "Undistorted"
[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This is my writeup document.

`Line Detect.ipynb` is the notebook in research, and `process.ipynb` is the notebook to process the result.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in `Line Detect.ipynb` (#2 and #3 `camera_calibration()`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![Undistorted][image7]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![Origin vs Undistort][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.(`combined_thresh()`)

The first binary contained values in both of x axis gradient and y axis gradient.
The second binary contained values in both of magnitude of gradient and direction of gradient.
The third binary contained values in the S channel of HLS color space which is in the range of theshold.

I combined these three binary images to get the final combined binary image.

![Combined binary][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I define a reuseable class `Perspective` that we will no longer need to calculate the perspective M every frame. I try the value of `src` from two straight line images to fit a acceptable value. And I used `Perspective.warp()` to warp the source image.

```python
img_size = (img_shape[1], img_shape[0])
offset = 300
src = np.float32([[215, 700], [590, 450], [690, 450], [1100, 700]])
dst = np.float32([[offset, img_size[1]], [offset, 0],
                  [img_size[0]-offset, 0], [img_size[0]-offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 215, 700      | 300, 720      | 
| 590, 450      | 300, 0        |
| 690, 450      | 980, 0        |
| 1100, 700     | 980, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![Warp][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First I visualize the value of warped binary, and I found that there were many noise in the left and right side of warped binary. So I ignore the both side of `300` px. 

![Line Histogram][image4]

After that I used the sliding windows to fit the lane lines.

![Lane Lines][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I calculated the radius of curvature in `calculate_curvature()` in `process.ipynb`.

And calulated the vehicle offset from center of line in `calculate_offset()`.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `process.ipynb` in the function `plot_lane_area()`. Here is an example of my result on a test image:

![Output][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I try to optimize my result in changing the threshold of binary, the position of `src` and `dst` in `warp()`, and the porcess in fitting the lines. 
I got a good enough result in most part in video, but in some place like concrete gound, shadow of tree, or a car too close the line will fail the pipeline.