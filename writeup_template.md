# Advanced Lane Lines Finding

**Predicting the accuracy of lane line is important for self-driving car to drive safely and applicable to the real world. If the self-driving car can not navigate on the assigned lane line consistently and accurately, there would be no way that it could be on the real street.**

**Therefore, it is important that we need to program the self-driving car to understand and recognize the lane lines. And this project is where we will start to do this job.**

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


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

In order to calibrate the camera, I need to find the object points and image points from all the images that were provided in the 'camera_cal' folder.  
The process of finding the object pionts and image points are following:
- Read all images file from 'camera_cal' folder
- Convert image into grayscale
- Find the chessboard corners by using  `cv2.findChessboardCorners()` function
- I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

<img src="https://github.com/loynin/Advanced-Lane-Finding/blob/master/output_images/undistort_output.png" width="800">


### Pipeline (single images)

#### 1. Distortion-corrected and warped image.

Distortion correction and warping image is important in order to correct image to have the real world view. 
Below are the steps of undistortion and warping image:
- After getting `objpoints` and `imgpoints`, I used these data to calibrate the camera.
- Then I use `cv2.undistort()` to correct distortion
- I then set the regional of interest:
``` 
src = np.float32([[490, 482],[810, 482],
                      [1250, 720],[40, 720]])
dst = np.float32([[0, 0], [1280, 0], 
                     [1250, 720],[40, 720]])
```
- After, I use `cv2.getPerspectiveTransform()` to transform image. 

Below is an example of undistorted and warped image:

<img src="https://github.com/loynin/Advanced-Lane-Finding/blob/master/output_images/warped.png" width="800">

#### 2. Combination of color transforms to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color thresholds to generate a binary image.
- First, I apply L channel threshold to the image
- Then, I apply B channel threshold to the image
- Last, combine both of the threshold to produce the combined binary image.
This process is demonstrated in the following code 
```
def to_combine_thresholds(img):
    img = np.copy(img)
    l_binary = l_select(img)
    b_binary = b_select(img)
    combined_binary = np.zeros_like(s_binary)
    combined_binary[(l_binary == 1) | (b_binary == 1)] = 1
    return combined_binary
```
Below is the result of image before and after applying thresholds:
<img src="https://github.com/loynin/Advanced-Lane-Finding/blob/master/output_images/threshold_image.png" width="800">

#### 3. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
There are two processes seperate from each other. One is the process of finding the lane lines and another one is the process of filling the polynomial of the lane.

**a. Finding the lane lines:**
To find the lane lines, I use the sliding window method. The method could be summarize as the following steps:
- Produce the combined threshold binary image
- Create histogram from binary image
- Find peak of histogram for both on the left and the right of image to find the lane lines
- Find the fitting plot for the lane lines
- Create a new image based on the original image
- Draw the lane lines

All these process is the block of code of `to_slide_window` function.

**b. Filling the polynomial:**
Unwarp the image and filling the polynomial is in the code block of the function `to_draw_lane`. The following is the summary step of filling the polynmial:
- Draw the lane onto the warped blank image
- Unwarp the drawed image to original image using inverse prespective matrix
- Combine the original image and the unwarped image.

Below is the result of finding lane line and filling polynomial image:
<img src="https://github.com/loynin/Advanced-Lane-Finding/blob/master/output_images/filling_polynomial.png" width="800">

#### 4. Calculate the radius of curvature of the lane and the position of the vehicle with respect to center.

Below is the code use to calculate radius of curvature and the posistion of the vehicle with respect to the center:

```
#Measure radius of Curvature of each land line
    ym_per_pix = 30./720 # meters per pixel in y dimension
    xm_per_pix = 3.7/700 # meteres per pixel in x dimension
    left_fit_cr = np.polyfit(lefty*ym_per_pix, leftx*xm_per_pix, 2)
    right_fit_cr = np.polyfit(righty*ym_per_pix, rightx*xm_per_pix, 2)
    left_curverad = ((1 + (2*left_fit_cr[0]*np.max(lefty) + left_fit_cr[1])**2)**1.5) \
                                 /np.absolute(2*left_fit_cr[0])
    right_curverad = ((1 + (2*right_fit_cr[0]*np.max(lefty) + right_fit_cr[1])**2)**1.5) \
                                    /np.absolute(2*right_fit_cr[0])
    
    
    # Calculate the position of the vehicle
    center = abs(640 - ((rightx_int+leftx_int)/2))
```

This code block is in the `to_slide_window` function.

---

### Pipeline (video)

That is all about it. Finally, I come to the final step is to build the pipeline for processing the video. This pipeline combined all the previous processing steps in order to create the final images.

The pipeline is run as the following processes:

1. Calibrate camera and undistort image
2. Perform perspective transform
3. Apply thresholds to the image
4. Draw line on the lane line
5. Fill polynomial of the transformed image
6. Perform reverse perspective transform
7. Write curvature info and radius of curvature to the image

Finally, pipeline is used to process the input video and produce the output video with the lane line mark, curvature information, and dadius of curvature information.

Here's a [link to my video result](https://github.com/loynin/Advanced-Lane-Finding/blob/master/result.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

As lightning condition change, there will be some problem for system to identify the lane line. As a result, choosing the right threshold value and threshold method is important for system to run accurately.

There are still room for improvement such as changing the value of thresholds and changing the method of thresholds could improve the system performance and accuracy.
