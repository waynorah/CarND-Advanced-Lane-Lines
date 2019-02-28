## Writeup


About Advanced Lane Finding
---

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.


1 Camera Calibration
---

1.1 Briefly state how you computed the camera matrix and distortion coefficients.
Here are the steps I used when calculating the matrix and distortion coefficients for camera calibration.

1）Prepare a replicated array of coordinates (objp) with specific size (nx = 9, ny = 6) to handle object points

2）Prepare the arrays to save image points (imgpoints) & object points (objpoints)

3）Apply following processes to each calibration image (calibration*.jpg)
   * Change image to grayscale
   * Call cv2.findChessboardCorners to find the chessboard corners
   * Add (append) the detected corners as image points together with a copy of "objp" as object points
   * Call cv2.drawChessboardCorners to draw the lines connnecting the detected chessboard corners
   * Save the image with detected corners (in /output_images/ folder)

4) Apply camera calibration (by calling cv2.calibrateCamera) with the imgpoints & objpoints taken above to get the matrix (mtx) and distortion coefficients (dist) of the camera

5) Save the matrix (mtx) and distortion coefficients (dist) to local file (using pickle) for later use

6) Load the saved data (mtx, dist) to avoid running the calibration from the beginning when restarting kernel, etc

7) Call cv2.undistort to undistort each image with calibrated data (mtx & dist)

8) Save the undistorted images (in /output_images/ folder)


1.2 Provide an example of a distortion corrected calibration image.

The output images with detected corners and undistorted images are saved in following path:

/output_images/corners_calibration*.jpg

/output_images/undistorted_calibration*.jpg 

Here are some examples of the undistorted image, showing together with its original image & image with detected corners for easy comparison:

Orginal:

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/camera_cal/calibration2.jpg

Dected Corners:

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_images/corners_calibration2.jpg 

Undistorted image:

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_images/undistorted_calibration2.jpg



Here are the example results after applying the distortion-corrected process mentioned above to the raw images in "/test_images/" folder

Orginal:

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/test_images/test1.jpg

Undistorted image:

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_images/undistorted_test1.jpg


2 Use color transforms, gradients or other methods to create a thresholded binary image
---

I implemented below helper functions to generate a binary image with various thresholds of color & gradient:

* abs_sobel_thresh(): create a binary image with the given sobel kernel size and threshold values of gradient (on x & y orientation)

* mag_thresh(): create a binary image with the given sobel kernel size and threshold values of gradient magnitude

* dir_threshold(): create a binary image with the given sobel kernel size and threshold values of gradient direction

* color_thresh(): create a binary image with a given range of color

* hls_select(): create a binary image with the threshold of S-channel on HLS

* combined_thresh(): create a binary image by combining several thresholds



3 Performed a perspective transform and provide an example of a transformed image
---


1) Choose the hard-coded source (src) and destination (dst) points as follows (similar with the value in template file):


src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
    
    
2) Pass src & dst together with targeted binary image (img) to function warped()

3) Inside warped():

   * Call cv2.getPerspectiveTransform(src, dst) to calculate the transform matrix (M)
   
   * Call cv2.warpPerspective() to execute perpective transform with the calculated matrix (M)
   
Here are some examples of transformed image:

 Original image undistyorted:
 
 https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_images/undistorted_test2.jpg
 
 Wraped:
 
 https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_images/warped_test2.jpg
 
 
4 Identified lane-line pixels and fit their positions with a polynomial
--- 

I implemented below search methods to identify lane-line:

Search method 1: process each frame separately to find lane pixels & fit a polynomial
Search method 2: use the parameters from last frame to search lane pixels & fit a polynomial

Here are the main steps I used in my implementation:


(1) Find lane pixels

    * Take a histogram of the bottom half of the image
    
    * Find the peak of the left and right halves of the histogram as the starting point for the left and right lines
    
    * Define the sliding windows (with optimized size) & step through each window as follows:
    
        Search for all nonzero pixels within the window
        
        Add/Append the indices of the detected pixels to the lists
        
        If the number of detected pixels exceed the threshold (minpix), recenter next window on the mean position
        
    * Concatenate the arrays of indices which previously was a list of lists of pixels
    
    * Extract left and right line pixel positions

(2) Search around to detect lane pixels & fit the detected lane with color lines

    * With search method 1:
    
        Find lane pixels as described in (1) for every frame
        
        Fit a polynomial from detected pixels
        
        
    * With search method 2:
    
        Load the saved parameters of last frame. If no data is saved (first frame), create it as follows:
        
            a. Find lane pixels as described in (1)
            
            b. Calculate & save polynomial fit parameters for left & right lane
            
            c. Load the saved parameters
            
        Set the area of search based on activated x-values within the +/- margin of polynomial function
        
        Search for all nonzero pixels within the area
        
        Fit a polynomial from detected pixels
        
        Check whether the detected lines is suitable for further processing or not by following conditions:
            
            -Average x-value of both lines are within the image (smaller than 1280)
            
            -Average distance between right & left lane is within suitable range (bigger than 680 and smaller than 820)
       
       If not suitable (sometimes happen), apply search method 1 for current frame


(3) Highlight the detected lane lines & moving area with colors

    * Define the coordinates for left lane (left_line_pts), right lane (right_line_pts) & middle area between the 2 lanes (middle_area)
    
    * Call cv2.fillPoly() to highlight specified areas with specific colors (Red & Green)


5 Calculated the radius of curvature of the lane and the position of the vehicle with respect to center
--- 

1）Define conversion rate in x and y from pixels space to meters
2）Change the given lane coordinates (ploty, left_fitx, right_fitx - simulated in previous step) in pixel space to meter space
3）Pass the new coordinates (in meter space) to np.polyfit() to fit second order polynomials for left & right line
4）Use the mathematic equation defined in link to calculate the left & right curvature radius (left_curverad, right_curverad) of the lane points whose y-value are correspondent to the bottom of the image.
5) Take the average of left_curverad & right_curverad as final radius of curvature
6) Calculate the position of left line & right lane nearest to the camera then take the average as the center of the 2 lanes (position)
7) Calculate the distance from center of the lane (position) to center of the image.

measure_curvature_radius_position(): calculate radius of curvature & the relative position (distance from center) from simulated lane coordinates

add_radius_distance(): combine the calculated information to the image being processed



Pipeline - videod
--- 

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_videos/project_video_output.mp4

https://view5639f7e7.udacity-student-workspaces.com/view/CarND-Advanced-Lane-Lines/output_videos/project_video_output_method1.mp4



Discussion
--- 
1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?

* As shown in output of project file, 3 test images (calibration1.jpg, calibration4.jpg, calibration5.jpg) can't be processed by current pipeline with specified size (nx = 9, ny = 6). The size needs to be adjusted accordingly. The calibration process may be more precise with additional imgpoints & objpoints taken from these images.


In comparison with search method 1, search method 2 can be processed more quickly with less computation. However, it may face accuracy problem when processing the frames which are not similar. I faced this issue during the implementation, in which the lane lines disappeared from the middle of project video & can't be detected again. 










