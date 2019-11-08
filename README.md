# **Advanced Lane Finding Project**
---
**Path Planning Project**

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

[image1]: ./writeup_img/before_cali.jpg "before_cali"
[image2]: ./writeup_img/undist_perspective.jpg "Undistorted"
[image3]: ./writeup_img/roi.jpg "Regions of interest"
[image4]: ./writeup_img/edge_detection_originaltest*.jpg "Edge_detection_original"
[image5]: ./writeup_img/edge_detection_edgestest*.jpg "Edge_detection_edge"
[image6]: ./writeup_img/edge_detection_stacktest*.jpg "Edge_detection_stack"
[image7]: ./writeup_img/undist_perspective.jpg "Perspective"
[image8]: ./writeup_img/undist_perspective_output.jpg "Perspective_output"
[image9]: ./output_images/test2.jpg "Output"
[video1]: ./output_videos/project_video.mp4 "Video1"
[video2]: ./output_videos/challenge_video.mp4 "Video2"
[video3]: ./output_videos/harder_challenge_video.mp4 "Video3"
---

## Project Description

Step 1: Compute the camera calibration matrix and distortion coefficients using chessboard images.

First step is to calibrate camera for image distortion using set of chessboard images. Using [cv2.findChessboardCorners](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html) function, we find location of each chessboard corners. Then, using [cv2.calibrateCamera](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_calib3d/py_calibration/py_calibration.html) function, we obtain calibration matrix and distortion coefficients for future use.

After we obtain calibration matrix and distortion coefficients, we are able to get undistorted image using [cv2.undistort](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_calib3d/py_calibration/py_calibration.html). Following is the result of undistorting chessboard images:

![alt text][image1]


Step 2: Color Transform/ Gradient Thresholding

Second step is to apply color transform and gradient thresholding to maximize lane detection. For color transform, we will be using red channel from BGR color space and saturation channel from HLS color space. Red channel is chosen as it detects white lane the best, and saturation channel is chosen as it does great job on detecting both yellow and white lane. On the other hand, for thresholding we will be using 4 different thresholding as follow:
1. absolute sobel thresholding: abs_sobel_thresh()
2. Gradient magnitude thresholding: mag_thresh()
3. Gradient direction thresholding: dir_thresh()
4. Color thresholding: apply_non_scaled_mask()

Afterwards, we will be using color_binary() to check how gradient thresholding and color thresholding is working for each red and saturation channel. Green dots represent result of gradient thresholding and blue dots represent result of color thresholding. Finally, using combined_binary(), we are able to see result of combined thresholded image.

Here are the image of output:

![alt text][image4]
![alt text][image5]
![alt text][image6]


Step 3: Apply perspective transform

Next step is to transform our sample image to birds-eye view.
First, using straight lane photo from front facing camera, we select coordinates of trapezoid, where the lane is located. Then we define destination coordinates so that lane would look straight (birds-eye view) using [cv2.getPerspectiveTransform](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_geometric_transformations/py_geometric_transformations.html). From the function, we obtain warped image, tranformation matrix M and inverse transformation matrix M_inv.

Below is the perspective transformed image:

![alt text][image7]
![alt text][image8]

Step 4: Lane detection

Using image that is color transform/ gradient thresholded and perspective transformed, we detect lane pixels by a histogram. Histogram of lower half image is first created then two peaks located at below mid point and above mid point are chosen as left and right lane. Afterwards, Sliding_Window() is used to identify most likely coordinate of each lanes in the window. In our project, 9 sets of windows slides through the image, for each left and right lane. Finally, using coordinates found from Sliding_Window(), we fit the coordinates by using [np.polyfit](https://docs.scipy.org/doc/numpy/reference/generated/numpy.polyfit.html).

Step 5: Curvature and offset from the center of the lane

Using Lane_Curvature(), we obtain curvature of the lane and offset from the center of lane. Curvature and offset are calculated based on U.S. regulation that minimum lane width is 3.7meters, and dashed lane line are 3meters long. We define that in y direction, 30/720 meters per pixel and in x direction, 3.7/700 meter per pixel. Curvature is averaged between left and right lane. Offset is by average location of lane from 600 to 720 pixel in y direction.

These values are stated on final output

Step 6: Final processing for the Output

Here, we unwarp the processed image for the output using M_inv obtained from Step 3. Also, we add warped image and lane detection image on the side. Final output is as follow:

![alt text][image9]

Step 7: Use pipeline on videos

We use pipeline on the videos and output is as follow:
![alt text][video1]

Here is the challenge video
![alt text][video2]

Harder challenge video
![alt text][video3]



---
## Discussion

Following project was really challenging for me and it had educated me in various way.

During the video processing, we are able to see some fluctuation and errors.
Here are the possible improvements that could be made:
* Perform sanity checks to confirm detected lane line is correct:
  - Check whether left and right lanes have similar curvature and if two lanes have different curvature, take the one with higher pixel value
  - Check whether distance between left and right lanes is approximately 3.7 meters.

* Further smoothening image/denoise the image for lane detection

* Perform lane detection only when lane goes out of the margin, instead of calculating lane every frame.
