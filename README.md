##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/Undistorted_output.png "Undistorted"
[image2]: ./output_images/Undistorted_transform.jpg "Road Transformed"
[image3]: ./output_images/binary_image.png "Binary Example"
[image4]: ./output_images/perspective_straight1.png "Warp Example_straight"
[image5]: ./output_images/find_line_noWindow.png "Fit Visual"
[image6]: ./output_images/final_output.png "Output"
[video1]: ./project_video.mp4 "Video"
[image7]: ./output_images/perspective_test1.png "Warp Example"
## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  3 pictures'(1, 4 and 5 calibration pictures) `ret` get false, so I can not get datas about corners. I however can get enough datas from 17 others pictures for camera calibration.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result. This image is calibration1.jpg which can't detect (9,6)corners in `cv2.findChessboardCorners()`function but this image's distortion correction is well because of camera calibration made up of enough datas: 

![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
I use frist block in `Advanced Lane Line.ipynb`. I save pickle file before step, and I bring it back and use some datas which are result of `cv2.calibrateCamera()` function. To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one :
![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (functions about color and gradient thresholds at lines 7 through 49 and pipeline function(`make_binary_image`) about combination about these functions at lines 51 through 63 in second block in  `Advanced Lane Line.ipynb`).  I used 3 `ksize` and (160,255) color threshold in this picture, but in video, I used 13 `ksize` and (120,240) color threshold. Here's an example of my output for this step. 

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 17 through 22 in 3rd block `Advanced Lane Line.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
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
| 700, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image(used 4th code block in `Advanced Lane Line.ipynb`).

![alt text][image4]
![alt text][image7]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First I used histogram, so can detected the location of lane line in binary image. And then made 9 windows each lane lines, got pixel positions. I used `np.polyfit()` function , `f(y)=Ay​^2​​+By+C` and before pixel positions datas, got a second order polynomial to each left and right(in code lines 67 through 68 in the 5th code block and code lines 16 through 17 in the 7th code block `Advanced Lane Line.ipynb`):

![alt text][image5]

above image is not using windows for finding lane line. 5th code block is about `find_line_with_windows` function and 7th code block is about `find_line_without_windows`function. I can use `find_line_without_windows`function when I already find lane line before frame.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
Using `R​_curve​​(y)=​​​(1+(2Ay+B)​2​​)​^(3/2)/​​​​∣2A∣​​` function. Also, I changed pixel space to meters. I can find how car is left form center, I use that camera is center of the car and half road size is 320 pixels. So, `320 - left lane line position`  is the answer of the left of the center. I did this in code lines 2 through 21 in 8th code block in `Advanced Lane Line.ipynb`. 

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 2 through 26 in my code(`draw_info` function) in 9th code block in jupyter notebook.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In this project video, there are the road parts which have shadow. This shadow confused my pipeline so failed detecting lane line. For solveing this problem, I made Line class for save 7 previous frames' datas(in 10th code block). I also changed my thresholds in RGBtoHLS transform function and gradient function. Finally, I solved that problem.

~~~~
