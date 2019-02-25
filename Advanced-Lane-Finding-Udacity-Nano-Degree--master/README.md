## Advanced Lane Finding Project
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./output_images/project_vieo_subclip.gif)

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

[image1]: ./output_images/Undistorted.jpg "Undistorted"
[image2]: ./output_images/Undistorted_test2.jpg "Road Transformed"
[image3]: ./output_images/Thresholded.jpg "Binary Example"
[image4]: ./output_images/warped.jpg "Warp Example"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/Processed.jpg "Output"
[video1]: ./output_images/project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. implementation of camera calibration using OpenCV functions.

The code for this step is contained in the third code cell of the IPython notebook located in "main_project.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Example of a distortion-corrected image.

Distortion can be corrected easily by applying `cv2.undistort()` function with calibration matirx `mtx`.  
```python
img = mpimg.imread('camera_cal/calibration1.jpg')
undistorted = cv2.undistort(img, mtx, dist, None, mtx)
```

To demonstrate this step, I apply the distortion correction to one of the test images:
![alt text][image2]

#### 2. Threshold the image using collor and grandiant

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps is contained in the sixth code cell of the IPython notebook located in "main_project.ipynb".)  Here's an example of my output for this step:

![alt text][image3]

#### 3. Perspective transform

The code for my perspective transform includes a function called `warp()`, which appears in the 8th code cell of the IPython notebook located in "main_project.ipynb".  The `warp()` function takes as inputs source (`src`) and destination (`dst`) points. Outpus perspective transfor matrix (`M`) and inverse perspective transfor matrix (`Minv`). I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[(img_size[0] / 2) - 60, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 60), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1127, 720     | 960, 720      |
| 690, 460      | 960, 0        |

The code for this step is contained in the 8th code cell of the IPython notebook located in "main_project.ipynb".  I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Lane-line pixels detection and fit their positions with a second order polynomial

Then I found right and left lines by finding the peaks in histogram of binary images. This is done by sliding window. The defalut value for the number of sliding window is 10, which works well for most of the straight lines. A hight number is required for roads with higher curvature. 
The code for this step is contained in the 9th code cell of the IPython notebook located in "main_project.ipynb".  
![alt text][image5]

#### 5.Radius of curvature of the lane and the position of the vehicle with respect to center.

It is important to know the radius of curvature of the lane because it will be used to determind the steering angle of the vehicle. After finding the polynomial coefficeints of the lane lines, we can use this formula to find the radius os curvature:

\begin{equation}
R_{curve}  = \frac{(1+(2Ay +B)^2)^{3/2}}{|2A|}
\end{equation}
here $A$, $B$, and $C$ are coefficeints of second order polinomial I found in previous step:
\begin{equation}
f(y)  = Ay^2 + By + C
\end{equation}
The code for this step is contained in the 11th code cell of the IPython notebook located in "main_project.ipynb".  

#### 6. Plot lane area and display the radius and offset on frame
I implemented this step in  in the 12th code cell of the IPython notebook located in "main_project.ipynb".   Here is an example of my result:

![alt text][image6]
---

### Pipeline (video)
Processing a video is just sequentianly precessing each frame of video as single image. Since the frames in video has a lot of common futureas, we can take adventage of those futures to speed up the algorithm and compensate for dirty images(E.g. missing lane lines, extra lines due to nonuniform road surface colar etc.)

#### 1. Test on project video.

Here's a [link to my video result for project](./output_images/project_video_output.mp4)

![Lanes Image](./output_images/project_vieo_subclip.gif)

#### 2. Test on challenge video.

Here's a [link to my video result for challenge](./output_images/challenge_video_output.mp4)

![Lanes Image](./output_images/challenge_vieo_subclip.gif)

#### 3. Test on harder challenge video.

Here's a [link to my video result for harder challenge](./output_images/harder_challenge_output.mp4)

![Lanes Image](./output_images/harder_challenge_subclip.gif)

---

### Discussion
Pipeline works pretty good on project video without any parameter tuning. However, it requires some parameter change for challenge and harder challenges. Even with tuning parameters, algorithm doesn't work very well for portion of challenges where there are some non-unirom road surface color or abrupt changes in brightness. It also very problematic when the curvature of road is vey hight in harder challenge. This pipeline very likely to fail at night and under rain.

Possible solution might be using deep learing methods which has less number of parameters to tune more robust. Looking forward explore it in next project. 

