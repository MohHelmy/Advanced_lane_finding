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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb" in Section 5th code cell of the IPython notebook.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one also images are embedded in the HTML of the project:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps "./P2.ipynb" (in Section 5 in the HTML).  Here's an example of my output for this step. 

The images are in this path .output_images\gradient\


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `GetBirdEyeView()`, which appears in Section 8th code cell of the IPython notebook `p2.py` (output_images/examples/example.py).  The `GetBirdEyeView()` function takes as inputs an image (`gradient_binary_image`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
    # This should be chosen to present the result at the proper aspect ratio
    # My choice of 100 pixels is not exact, but close enough for our purpose here
    # offset for dst points
    offset = 100
    
    src = np.float32([[w, h-10],    # br
                      [0, h-10],    # bl
                      [(w/2) - offset, (h/2) + offset],   # tl
                      [(w/2) + offset, (h/2) + offset] ])  # tr
    
    dst = np.float32([[w, h],       # br
                      [0, h],       # bl
                      [0, 0],       # tl
                      [w, 0] ])      # tr

    M = cv2.getPerspectiveTransform(src, dst)
    Minv = cv2.getPerspectiveTransform(dst,src)
    
```


I verified that my bird view was working as expected .

"./output_images/bird_eye_output"

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

def fitCurvatureUsingSlidingWindow(binary_warped_image,nwindows,margin,minpix):
    
    # Find our lane pixels first
    leftx, lefty, rightx, righty, out_img = findLanesPixels(binary_warped_image,nwindows,margin,minpix)

    # Fit a second order polynomial to each using `np.polyfit`
    left_fit,right_fit,left_fitx, right_fitx, ploty = fitPolynomial(binary_warped_image.shape, leftx, lefty, rightx, righty)

    ## Visualization ##
    # Colors in the left and right lane regions
    out_img[lefty, leftx] = [255, 0, 0]
    out_img[righty, rightx] = [0, 0, 255]
        
        
    return left_fit,right_fit,ploty,left_fitx,right_fitx,out_img
    



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in Section 11th code cell of the IPython notebook `p2.py`
Function name measureCurvatureReal(ploty, left_fit_cr, right_fit_cr):

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in 10th code cell of the IPython notebook `p2.py`in the function `insertLanesAreaonImage()`.  Here is an example of my result on a test image:

"./output_images/final_image"

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline relies on a restricted region of interest based on several iteration.

I belive the algorithm won't work as expected 
 1-if the road was going sharp up , sharp down Sharp left , or sharp right Like the hard challenge video.
 2-Too many changes in light in very short time , Algorithm can't detect the lanes correctly espiceially the lanes are not drawn in some frame images.
 3-The road has many colors in the same lane (Dark gray and light gray) as it give fake lane lines and the algorithm can't differntiate between the fake lanes and the real lanes
 4-I think the pipeline will perfom badly on darker , foggy road , sharp turns , tunnels or under bridges
