# **Finding Lane Lines on the Road**

## Writeup

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_assets/output_6_2.png "Original"
[image2]: ./writeup_assets/output_18_1.png "HSL"
[image3]: ./writeup_assets/output_20_1.png "yello hsl"
[image4]: ./writeup_assets/output_21_1.png "white hsl"
[image5]: ./writeup_assets/output_23_1.png "hsl masked"
[image6]: ./writeup_assets/output_25_1.png "grey scale"
[image7]: ./writeup_assets/output_26_1.png "gaussian blur"
[image7b]: ./writeup_assets/output_100.png "gaussian blur2"
[image8]: ./writeup_assets/output_28_1.png "canny edge"
[image9]: ./writeup_assets/output_30_1.png "region of interest"
[image10]: ./writeup_assets/output_32_1.png "hough transform"
[image11]: ./writeup_assets/output_33_1.png "weighted"

[image50]: ./writeup_assets/output_39_0.png "p1"
[image51]: ./writeup_assets/output_39_1.png "p1"
[image52]: ./writeup_assets/output_39_2.png "p1"
[image53]: ./writeup_assets/output_39_3.png "p1"
[image54]: ./writeup_assets/output_39_4.png "p1"
[image55]: ./writeup_assets/output_39_5.png "p1"

[image60]: ./writeup_assets/output_41_0.png "p2"
[image61]: ./writeup_assets/output_41_1.png "p2"
[image62]: ./writeup_assets/output_41_2.png "p2"
[image63]: ./writeup_assets/output_41_3.png "p2"
[image64]: ./writeup_assets/output_41_4.png "p2"
[image65]: ./writeup_assets/output_41_5.png "p2"

[image70]: ./writeup_assets/output_43_0.png "p3"
[image71]: ./writeup_assets/output_43_1.png "p3"
[image72]: ./writeup_assets/output_43_2.png "p3"
[image73]: ./writeup_assets/output_43_3.png "p3"
[image74]: ./writeup_assets/output_43_4.png "p3"
[image75]: ./writeup_assets/output_43_5.png "p3"

[image100]: ./writeup_assets/output_56_3.png "left right separated"

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

In summary, my pipeline consisted of the following steps:

1. Change colour space from RGB to HSL
2. Mask image with yellow-filter and white-filter
3. Convert to grey scale
4. Apply Gaussian blur
5. Canny edge detection
6. Define region of interest
7. Perform Hough transform
8. Separate left and right lanes
9. Interpolate lanes with linear regression

I have modified the `draw_lines()` functions to implement step 8 and 9 in the pipeline.

### Pipeline

#### 1. Change colour space from RGB to HSL

This step is necessary when the contrast between the lane and road is not clear enough or the lighting on the road is inconsistent with things like shadows. To pick out the yellow and white lanes against a bright road, first convert the colour space from RGB to HSL (Hue, Saturation, Lighting).

![alt text][image1] ![alt text][image2]


#### 2. Mask image with yellow and white filter

Once the image has been converted to the HSL colour space, I created a filter that only picks up yellow and another filter with white. For creating the filter, I first used the colour range based on the traffic-yellow in RGB:(250, 210, 1) converted to HSL:(25, 126, 253). Then I experimented by tweaking each hue, saturation and lighting  and chose the value that worked best with the test images. The same was done to find the range for white.

After creating the two filter, I masked the original image with both of them to pick out only the colours that fit in yellow range and white range in the image.

![alt text][image3] ![alt text][image4]

#### 3. Convert to grey scale

This step reduces the amount of data that needs to be processed in the future steps. In the current step, the image holds only white and yellow which will contrast very highly on the dark road in gray scale. Therefore, we can flatten the image to single dimension (gray scale) without losing important information. This will speed up the computation in the following steps. The image on the left shows the output of the original image masked with yellow and white HSL filter. The image on the right is the greyscaled version of the left image.

![alt text][image5] ![alt text][image6]

#### 4. Apply Gaussian blur

In this step, I smoothed out the image using the Gaussian blur technique. Just like grey scaling, this step reduces noise and helps prepare for edge detection. The kernel size will determine how much blurring will occur. By experimentation, I have chosen kernel size of 5 to be a good balance that doesn't overly smooth out the image.

![alt text][image7] ![alt text][image7b]

#### 5. Canny edge detection

We can finally use the canny edge detection to find the lines in the image. The lower and higher boundaries are found based on experimenting and according to the OpenCV's documentation (1:2 or 1:3, low:high ratio). You don't want to capture too much detail but also don't want to lose any information of the lanes either.

![alt text][image8]

#### 6. Define region of interest

This step creates mask that will be applied to capture the region of interest. We are only interested in the road ahead of the car's dashboard and therefore, we can apply a trapezoid mask (passed as array of vertices) on top of the image to identify the critical region we are interested in.

![alt text][image9]

#### 7. Perform Hough transform

This is the final step required to identify which edges found from the last step are actually road lanes. The parameters define how long and linearly aligned the points are to be considered as lane. I tuned the parameters by testing against the sample images and chose the ones that yielded the best result.

![alt text][image10]

Results from test images:

Step 1-2
![alt text][image50]
![alt text][image51]
![alt text][image52]
![alt text][image53]
![alt text][image54]
![alt text][image55]

Step 3-5
![alt text][image60]
![alt text][image61]
![alt text][image62]
![alt text][image63]
![alt text][image64]
![alt text][image65]

Step 6-7
![alt text][image70]
![alt text][image71]
![alt text][image72]
![alt text][image73]
![alt text][image74]
![alt text][image75]

### Modify draw_lines()

#### 8. Separate left and right lanes

Before we interpolate the lanes, we must classify them as left lane or right lane. This was simply done by calculating the slope of the line. Positive slope was classified as left lane where as negative slope was classified as right lane. I also neglected slopes that were smaller than certain threshold, as you can't have horizontal lane going across the road.

#### 9. Interpolate lanes with linear regression

Using the scipy library, I simply applied the `linregress()` function to interpolate each of left and right lanes. Linear regression is basically reducing the error region (least mean square) between the estimation of line against the actual points given to us from the Hough transform. The picture below shows the final result of the interpolated lanes.

![alt text][image100]

### 2. Identify potential shortcomings with your current pipeline

This is an overly simplified lane detection method, and it will have many shortcomings if the environments are even slightly modified.

The biggest problem with my current pipeline is that it won't be able to handle any curvatures on the road. For example, a strong road curve to right will result in positive slopes for both left and right lines. My pipeline will not be able to detect the right lane, even if it some how managed to identify the curved line as a lane.

Other problems that my pipeline might struggle with is handling any variations of road. For example, my method will get confused with road markings such as the carpool lane signs (white diamond) or an arrow signs drawn on the road. It could possibly mistake the arrow line as the lane line since the lanes don't have any defined width. There could also be words written on the road like "SLOW", or the lanes could even be doubled to indicate no passing zones. There can also be passenger crossings with big white vertical blocks ahead. Roads can also merge, which can potentially confuse my pipeline when the lanes get really close in slightly diagonal direction. On some big four way crossings, the lanes can be missing for a while until you reach the end of the crossings.

Finally, my pipeline hasn't been tested against variables like weather conditions or driving behaviour. Snow can cover parts of lanes on road or even the entire view of the dashboard. Very bright sunbeam can create blind spots for some period of time. When another car merges ahead, it can cover up parts of the lanes for small amount of time. When my car changes lanes, it needs to figure out which lanes to look for depending on the direction of lane change.

### 3. Suggest possible improvements to your pipeline

A possible improvement to overcome lane detection for curved road would be to implement curve fitting (second degree polynomial) instead of straight lines (linear). Also, identifying left and right lanes by slope will no longer be working. We should instead determine the lanes with respect to center of the image.

Another potential improvement that could be made to my pipeline is updating my region of interest function to handle dynamic situations. Currently, I have hardcoded a list of vertices that creates the same polygon (trapezoid) for every frame in the video. If the video changes in size, or if there is a sharp curve in the road, my pipeline will not function properly.
