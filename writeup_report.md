**Advanced Lane Finding Project**

The goals/steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to the center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/undistort_output.png "Undistorted Chessboard"
[image2]: ./output_images/test5.jpg "Road Original"
[image2b]: ./output_images/undistorted.jpg "Road Undistorted"
[image3]: ./output_images/thresholded.jpg "Binary Thresholded"
[image4]: ./output_images/warped_straight_lines.jpg "Warp Example"
[image5a]: ./output_images/warpedbinary.jpg "Warped Binary"
[image5]: ./output_images/color_fit_lines.jpg "Fit Visual"
[image6]: ./output_images/example_output.jpg "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.

###Camera Calibration

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.
The calibrate_save function in the 4th code cell of the IPython notebook, calibrates the camera and saves the mtx and dist matrices with pickle. The calibrate_load function in the 5th code cell of the IPython notebook, loads the saved mtx and dist matrices.

 I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result(7th code cell):

![alt text][image1]

###Pipeline (single images)

####1. For the fifth test image (test5.jpg)
![alt text][image2]

First the image is undistorted using the cv2.undistort() function and the mtx and dist parameters obtained from cv2.calibrateCamera() function.

![alt text][image2b]

####2. Gradients and color thresholds are used to obtain a binary image.

The beginning transformations performed with X and Y gradient and S threshold of HLS color space, and V threshold of HSV space. The thresholds are as follows:

| Channel/Grad  | Space  | Thresholds    |
|:-------------:|:------:|:-------------:|
| X             | RGB    | 12, 255       |
| Y             | RGB    | 25, 255       |
| S             | HLS    | 100, 255      |
| V             | HSV    | 50, 255       |

X and Y gradients are combined with and, S and V combinations combined with and. These sub combinations are combined with or.

These transformations can only work well on the project video. The color changes on the road are mixed with the lane lines when the images of challenge video transformed with this transformation.

Therefore, Y gradient transformation is removed, and the other threshold values are changed to the following:

| Channel/Grad  | Space  | Thresholds    |
|:-------------:|:------:|:-------------:|
| X             | RGB    | 12, 255       |
| S             | HLS    | 100, 255      |
| V             | HSV    | 170, 255      |

In order to filter the yellow and white colors, 2 different threshold transformations are added. To select yellow pixels, the following thresholds combined with an and.

| Channel/Grad  | Space  | Thresholds    |
|:-------------:|:------:|:-------------:|
| R             | RGB    | 140, 255      |
| G             | RGB    |  50, 150      |
| B             | RGB    |   0, 135      |

In order to select white pixels, the following thresholds combined with an and.

| Channel/Grad  | Space  | Thresholds    |
|:-------------:|:------:|:-------------:|
| R             | RGB    | 170, 255      |
| G             | RGB    | 170, 255      |
| B             | RGB    | 170, 255      |

The code for performing yellow and white selection is the yellow_white function in the 9th code cell of the IPython notebook.
Then, these two filters combined with an or. And this combination combined with the beginning transformation combination, except the Y gradient, with an and. The combination is performed with the color_transform_bxsv function in the 14th code cell of the IPython notebook.

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective()`, which appears in the 16th code cell of the IPython notebook. The `perspective()` function takes as inputs an image (`img`). The source (`src`) and destination (`dst`) points are defined in the function as the percentage of the image height and width. These parameters are given in the following table. The first values are the X values and they are the percentage value of the image width. The second values are the Y values and they are the percentage value of the image height.

| Point         | Source        | Destination   |
|:-------------:|:-------------:|:-------------:|
| Top Left      | 0.46, 0.65    | 0.25, 0.0     |
| Top Right     | 0.54, 0.65    | 0.75, 0.0     |
| Bottom Right  | 0.725, 0.95   | 0.75, 1.0     |
| Bottom Left   | 0.275, 0.95   | 0.25, 1.0     |

The corresponding values for a 1280 x 720 image are given in the following table.

| Point         | Source        | Destination   |
|:-------------:|:-------------:|:-------------:|
| Top Left      | 588.8, 468    | 320,   0      |
| Top Right     | 691.2, 468    | 960,   0      |
| Bottom Right  |   928, 684    | 960, 720      |
| Bottom Left   |   352, 684    | 320, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

These values are selected in order to find lane lines in the challenge video. The previous trapezoid size was bigger. The previous percentage values are given in the following table.

| Point         | Source        | Destination   |
|:-------------:|:-------------:|:-------------:|
| Top Left      | 0.424, 0.68   | 0.25, 0.0     |
| Top Right     | 0.576, 0.68   | 0.75, 0.0     |
| Bottom Right  | 0.805, 0.935  | 0.75, 1.0     |
| Bottom Left   | 0.576, 0.935  | 0.25, 1.0     |

![alt text][image4]

####4. The warped binary image of the test5 image and the identified lane-line pixels are given in the following images.

![alt text][image5a]
![alt text][image5]

The tracker class defined in the code cell 19. It is used to track the information about the previous images. This class has a method find_window_centroids which takes a warped binary image and returns the center points for left and right lane lines. This function starts with looking at the half bottom of the image, and finds the most frequent x value for both the left half of the image and right half of the image. These x values are the starting points from the bottom for the left and the right lanes. Then, the image will be split into windows, and for each window the new most frequent x values will be searched. If an x value is found for the window, then the x value and corresponding y value are added to the list of the lane. The function will return these lists that are called left_centroids and right centroids.

The process_image function in the 21st code cell, performs undistortion, color transformation, and perspective transformation by calling the related functions. After building the warped binary image, if there was no fit in the previous image, this function calls the find_window_centroid function of the tracker instance. Using these centroids polynomials are constructed. The indices of the points that are near to these polynomials are selected, left_lane_inds and right_lane_inds. These points shall be near to the left_fit polynomial or right_fit polynomial by a length of margin. If there was a fit in the previous image, left_fit and right_fit polynomials are obtained from the last image. Therefore the find_window_centroid function is not called.

After new polynomials are fit to the lane lines, sanity check performed. The distance between the left and the right lanes at the bottom (y=720 in the warped image) and at the top (y=0 in the warped image) are saved in a list. When a new polynomial is calculated, the distance values will be compared to the mean of the previous values. Up to 15 previous values are used in the comparison. If the error for the bottom distance is greater than or equal to 50 or the error for the top distance is greater than or equal to 70, the calculated curves will not be used. The previously saved curves will be used instead. If there are more than 10 frames that the calculated curves don't pass the sanity check, the new curves used instead of the previously saved curves.

After the curves selected, lane lines are plotted in an empty image, inverse perspective transform applied and the result is combined with the original image.

####5. Calculating the radius of curvatures

Lane line curves that are in pixel units are converted to the lane line curves that are in meter units. Using the real world space curves and the radius formula given below, the radius of curvatures are calculated. curverad_l value is the radius of the left lane curvature and curverad_r value is the radius of the right lane curvature.

R = ((1+(2Ay + B) ^ 2) ^ (3/2)) / |2A|

The position of the vehicle with respect to the center is calculated by first calculating the camera center. The camera_center value is given by summing the B values of the left lane polynomial and the right lane polynomial and dividing this summation by 2. The difference can be calculated by subtracting the half of the image width from the camera center value. To convert this value to meter units, multiply the result by the x meters per pixel (xm_per_pix) value.

I did this in lines # through # in my code in `my_other_file.py`

####6. Example of result test5.jpg.

![alt text][image6]

###Pipeline (video)

####1. Project Video

[![First Track Clockwise](http://img.youtube.com/vi/z-dF_K5amP0/0.jpg)](http://www.youtube.com/watch?v=z-dF_K5amP0)

####2. Challange Video
[![First Track Counter-Clockwise](http://img.youtube.com/vi/Yivfu9rVxuc/0.jpg)](http://www.youtube.com/watch?v=Yivfu9rVxuc)

###Discussion

I applied filtering, Sobel gradient, color channel thresholding, however with the threshold parameters I applied, the challenge video can not be processed successfully. Especially, in the beginning of the video, there are other lines that are get mixed with the lane lines. There are better techniques with different filters, however, I don't have enough experience with them. As a result, I tried to filter the yellow and white colors by the techniques that were shown in the first project. Then the processing time was considerably increased. In addition, in the challenge video when passing under the overpass, the colors have very low color values. Therefore, they can not be detected as yellow or white with the selected thresholds. Since the pipeline was using the previous curvature when it can not successfully detect a new one, the video could be processed. However, if there was a longer period of low light, the pipeline probably would not be able to process the video.

The choice of the source and destination trapezoids was another issue. I still don't have any idea if it is possible to use the same source and destination values for all of the videos, or it is required to use different source and destination parameters. The best source trapezoid for the project video did not perform well on the challenge video.

This pipeline failed in the harder challenge video. It will likely to fail in low light, and roads other than highways, especially where sharp turns exist. It is required to add more advanced techniques to distinguish lane lines from other lines, and to detect lane lines in low light.
