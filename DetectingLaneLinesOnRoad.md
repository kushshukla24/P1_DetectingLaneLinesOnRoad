# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road**

Self-Driving Cars have a camera mounted over the roof-top, which provides vision to the car.
The goal of this project is to create a pipeline to detect lane lines on road in a frame captured from a roof mounted camera.

[image1]: ./test_images/solidYellowCurve.jpg?raw=true "Sample Image from Roof-Mounted Camera"
[image2]: ./test_images_output/solidYellowCurve/solidYellowCurve_2.jpg "Grayscale"
[image3]: ./test_images_output/solidYellowCurve/solidYellowCurve_3.jpg "Gaussian Smoothening"
[image4]: ./test_images_output/solidYellowCurve/solidYellowCurve_4.jpg "Masked"
[image5]: ./test_images_output/solidYellowCurve/solidYellowCurve_5.jpg "Hough"
[image6]: ./test_images_output/solidYellowCurve/solidYellowCurve_6.jpg "Final"
[image7]: ./examples/3lanelines.jpg "3 Lane Line"
[image8]: ./examples/zebra_crossing.jpg "Zebra Crossing"
[image9]: ./examples/challenge.jpg "Blue Lane Line"
[image10]: ./examples/sharp_turn.jpg "Sharp Turn"

![alt text][image1]

---

### Concepts Used
* Converting Image to Grayscale
* Color Selection 
* Extracting Region of Interest 
* Gaussian Blur for pixel smoothing 
* Canny Edge Detection
* Hough Tranform line detection
* Linear Regression Line Fitting

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

First, I converted the images to grayscale. This changed my input from 3 channel(RGB) to 1 channel(gray) image. It will help me to make quick computations in later steps of the pipeline. As for detecting lane markings on the road, mostly white and yellow, can be efficiently computed on gray scale while RGB channels will not add much value.

![alt text][image2]
    

Second, I applied gaussian smoothening as pre-work (to gray image) and then applied canny algorithm to detect edges on the image. Gaussian smoothening will introduce slight blur to image as it averages nearby pixel values (based on kernel size). This will help canny algorithm to neglect edges with low gradient as they will smoothened by gaussian blur pre-operation. 

![alt text][image3]
    

Third, I deliberately masked the edge image with quadrilateral to extract region of interest showing the road and lane lines.

![alt text][image4]
    

Fourth, I then applied Hough Transform on the masked image to draw lines based the pixel points detected by canny algorithm. This resulted in multiple lines with approximately similar slopes around the lane lines. Technically, the outut is correct but does not makes much sense for self-driving car. A single line on seperate lanes will be much clear and less-ambiguous for a car to make decision. 
So, in order to draw a single line on the left and right lanes, I modified the draw_lines() function :
1. Bifurcated the lines to either left lane or right lane based on the slope of the line.
2. Fitted a regression line for each of left lane and right lane lines.
3. Extrapolated the regression line to cover the spectrum of road under purview.

![alt text][image5]
    

Fifth, finally I overlayed the line image from the previous step over the actual image. This validates the correct lane line detection on road within car field of view.
    
![alt text][image6]


### 2. Identify potential shortcomings with your current pipeline

I can see shortcomings with current implementation under following scenarios:

* If the number of lanes lines on the road under purview of the frame is more than 2. Consider the image below:

![alt text][image7]


In this case the current implementation will detect just extrapolated left and right lane lines. which ideally should detect three lane lines.


* If the frame contains zebra crossing

![alt text][image8]

* If the frame contains zig-zag lanes. (zig-zag lanes can be seen in the image above as well)

* At sharp turns, it may not be able to detect any lane line.

![alt text][image10]

* If the lane line is of blue color. This is one color I detected on which applying the gray scale the blue line almost disapperas. There can be other many such colors.

![alt text][image9]

### 3. Suggest possible improvements to your pipeline

Possible improvement to the pipeline which may solve problems listed above would be:
* Some intelligent runtime classification of number of lane lines and then assigning a extrapolated line for each lane would be benefical.

* Diffrentiation between lane lines and other road markings eg. zebra crossing, parking line.

* Detecting lane line and applying, say white color to all lane liines, to avoid lane lines being dropped during the grayscaling phase of pipeline.

* Region of Interest may vary with the image. So automated detection of region of interest would help to make the pipeline robust.

* Automated Parameter tuning based on the image quality and edges.

* Using non-linear line approximation (may be, parabolic) for better lane line recognition on sharp turns. 
