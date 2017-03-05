#**Finding Lane Lines on the Road** 
---

**Finding Lane Lines on the Road**

The goal of this project is to detect lanes with an image processing pipeline. The input to the pipeline is a video of a road from a dashboard camera and the output is the lanes detected by the pipeline overlaid on top of the input video.

[//]: # (Image References)
[image1]: ./test_images/solidYellowCurve2.jpg "input"
[image2]: ./test_images/solidYellowCurve20ut.jpg "yw"
[image3]: ./test_images/solidYellowCurve20ut0.jpg "smooth"
[image4]: ./test_images/solidYellowCurve20ut1.jpg "edge"
[image5]: ./test_images/solidYellowCurve20ut2.jpg "mask"
[image6]: ./test_images/solidYellowCurve20ut3.jpg "lines"
[image7]: ./test_images/solidYellowCurve20ut4.jpg "output"

![alt text][image1]
![alt text][image7]


---
###Pipeline
The pipeline is made of several stages, the output from each stage becoming the output to the next. 
####Stage 1 Color selection
The input video is fed to the pipeline one frame at a time. The first step of processing an image is to select colors that correspond to the lane markings on the road. Selecting only a few relevant colors in an image makes the subsequent stages of image processing operate on data that is less noisy. The image is converted HSV color space is extract yellow and white colors in the image.
![alt text][image2]

####Stage 2 Noise reduction
A Gaussian filter is employed to smoothen the image. This step pre-processes the image by removing some noise for the following stage.
![alt text][image3]

####Stage 3 Edge detection
A Canny edge detector operates on the monochrome image that has everything except the yellow and white colors masked out. The Canny detector is tuned so that the sufficient contour details are captured as edges while ensuring we don't pass through noise for the next stage. The only detail that we care about is the lane edges. So, the goal is to eliminate any details that is not part of the lane.
![alt text][image4]

####Stage 4 ROI selection
It is likely that the lane markings are restricted to certain section of the image, this depends on the position of the camera that captured input video. Polygon shaped masks are used to select the region of interest (ROI) that contains the lanes. Selecting the ROI vertices needs some attention to the image resolution.
![alt text][image5]
####Stage 5 Line detection
Now, the edge image is ready for line detection. Hough Line transform is used to extract straight line segments in the image of edges. As there can be several lines that do not correspond to lanes, this stages requires careful tuning.
![alt text][image6]
####Stage 6 Statistical post-processing
There can and will be several lines for each lane. This pipeline employs several statistical measures to identify the lane lines and convert them into a single line per lane.
 - the slope and y-intercepts of the detected lines is used to distinguish lines that correspond to left and right lanes
 - the slope and y-intercepts of the lines is averaged while providing more weight to longer segments
 - stray line segments that do not constitute the lanes are removed by comparing their slope and y-intercepts with the mean and standard deviation of detected lane segments. This assumes most of the segments that were identified are actually part of the lane. So this depends on the previous stages doing a good job of detecting sufficient number of lane lines.
 - the co-ordinates of the final line segment that extends between the edges of the ROI is computed from the slope and y-intercepts
 - a moving average of the co-ordinates is maintained across frames to reduce jitters and jumps in the computed co-ordinates
![alt text][image7]

###2. Shortcomings
 1. The moving average filter provides a smooth output when the lane markings are closer, but when they are farther away from each other, it results in jumps. This happens especially during steep curves, when the slope and intercept averages of the last n frames tends to be very different from the lane marking of the current frame.
 2. The ROI selection has to be hand-tuned based on the input video. The same pipeline will have trouble if the ROI vertices are not adjusted for a video that is shot from a different camera position and angle.
 3. As the lane colors are hard-coded, the pipeline will have trouble if the lane marking are not clear, worn out or completely different color (green for bike lanes).
 4. Handling other road markings like railway crossing, intersection markings, bicycle lane markings is an issue for the current pipeline.
 5. Robustness against changes in ambient light and road color.
 6. Large vehicles with same color as lane markings.
 7.  The pipeline makes use of several hard-coded values which may not work well for other input videos,

###3. Possible improvements
 1. The issue with the moving average filter can be overcome by providing more weight to the current frame.
 2. Use curve-fitting to ignore stray lines and make lane detection more robust.
 3. Machine learning to possible reduce or eliminate hard coded parameters.