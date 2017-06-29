# **Finding Lane Lines on the Road**

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[test1]: ./test_images/solidWhiteCurve.jpg "Solid White Curve Test Image"
[test2]: ./test_images/solidWhiteRight.jpg "Solid White Right Test Image"
[test3]: ./test_images/solidYellowCurve.jpg "Solid Yellow Curve Test Image"
[test4]: ./test_images/solidYellowCurve2.jpg "Solid Yellow Curve 2 Test Image"
[test5]: ./test_images/solidYellowLeft.jpg "Solid Yellow Left Test Image"
[test6]: ./test_images/whiteCarLaneSwitch.jpg "White Car Lane Switch Test Image"

[output1]: ./test_images_output/solidWhiteCurve.jpg "Solid White Curve Output Image"
[output2]: ./test_images_output/solidWhiteRight.jpg "Solid White Right Output Image"
[output3]: ./test_images_output/solidYellowCurve.jpg "Solid Yellow Curve Output Image"
[output4]: ./test_images_output/solidYellowCurve2.jpg "Solid Yellow Curve 2 Output Image"
[output5]: ./test_images_output/solidYellowLeft.jpg "Solid Yellow Left Output Image"
[output6]: ./test_images_output/whiteCarLaneSwitch.jpg "White Car Lane Switch Output Image"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 7 steps, with optional debugging output after each image modification.  These steps are:

1. First, I converted the images to grayscale, fed the grayscale image into a gaussian blur function, and lastly performed a Canny Edge Detection on the image.  All of this functionality was provided by the helper functions grayscale(), gaussian_blur(), and canny().

2. Next I set a numpy array to the coordinates of the Region Of Interest I want to use for the images.  The ROI shape I use is a tall, steep trapezoid containing the most likely area where the lane lines will be seen. With this array set I omit all color from the image outside the Region Of Interest via another helper function provided, region_of_interest().  

3. All that is left in the image now is a few lines that are the most likely candidates for lane lines.  I run the image through the Hough Lines helper function, hough_lines(), which I modified to also return the lines found from the Hough Transform.

4. After these modifications to the image have been made, I loop through the hough lines returned in step 3, breaking them into two categories based on their calculated slope: positive and negative.  This separates lines from the left and right of the lane since their slopes are generally mirrored across the y-axis.  With this slope information, I also segregate the X and Y value pairs from their respective lines.

5. With the X and Y values for each side of the lane, I perform a Linear Regression from the SciPy Stats library.  This gives me the slope and intercept of the line of best fit between the points.  I do this for both sides and calculate X and Y values based on this information as well as the width of the image.

6. The X and Y values of the lines of best fit can now be drawn onto the image.  To do this I use another helper function, draw_lines().  After drawing the lines, I clean up the lines using the region_of_interest() function again with the set ROI coordinates to clean up the drawn lines.

7. Finally I have this cleaned line image. I decided to convert it to an image mask. This is made to contain only the red lines.  I then use a bitwise_xor() from the CV2 library to invert any colors under the red overlay line mask and the original image.  This is to fix a problem I kept having where the line would wash out when bright colors were underneath.

This pipeline worked excellent on the test images, and the first video worked just as well.  Video number 2 was a little more difficult, with a few frames having no line detected and a few with erratic lane lines drawn.  The challenge video however, did not work at all, the curvature of the road was too extreme for the lane lines to be drawn or detected correctly.

Below is a sample of the input and output images:

Original
![Original Image][test1]

Output
![Output Image][output1]


#### Other images:

2. Solid White Right
  * [Test][test2]
  * [Output][output2]
3. Solid Yellow Curve
  * [Test][test3]
  * [Output][output3]
4. Solid Yellow Curve 2
  * [Test][test4]
  * [Output][output4]
5. Solid Yellow Left
  * [Test][test5]
  * [Output][output5]
6. White Car Lane Switch
  * [Test][test6]
  * [Output][output6]

#### Videos:

1. Solid White Right
  * [Test](./test_videos/solidWhiteRight.mp4 "Solid White Right Test")
  * [Output](./test_videos_output/solidWhiteRight.mp4 "Solid White Right Output")
2. Solid Yellow Left
  * [Test](./test_videos/solidYellowLeft.mp4 "Solid Yellow Left Test")
  * [Output](./test_videos_output/solidYellowLeft.mp4 "Solid Yellow Left Output")
3. Challenge
  * [Test](./test_videos/challenge.mp4 "Challenge Test Video")
  * [Output](./test_videos_output/challenge.mp4 "Challenge Output")


### 2. Identify potential shortcomings and improvements


I detected a few shortcoming with the current pipeline I have implemented:  

* First I had many BGR/RGB issues and other color glitches.  I'm thinking maybe using HSV color space while working on the image might lead to less headaches.

* I noticed that a few frames would drop the lane lines, or they would jump around erratically for a frame or two (with the exception of the challenge video, which that is the norm for the entire video).  From one frame to the next, the lane lines would not be able to move very far from the previous, so storing the lane line from the last frame and moving it slightly would be a good way to normalize this behavior.

* Detection of lane lines is completely broken for the challenge video.  The sample images got me too comfortable with linear detection, so curves and turns are not detectable with my pipeline.  Using nonlinear detection will need to be used to fix this.

* Another shortcoming effecting the challenge video and curved lanes is that my Region Of Interest is too narrow and cuts the lane off and out of view.  The region will need to be widened or, better yet, curve with the lane.

* Small adjustments to the lane lines accuracy can also be obtained by my implementation of a filter for lines that are outliers in the data.  Extra lines detected by the hough_lines function are added into the lane line detected, moving it slightly.

* Many optimizations can be made to speed up the pipeline for real-time video feed use.  Making fewer image copies and translations, with the removal of debugging code and improved lane line drawing, would make execution between 15 and 40 percent faster.
