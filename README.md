# **Finding Lane Lines on the Road** 


**Finding Lane Lines on the Road**

In this project, I used OpenCV, python Matplotlib, numerical python to find lane lines in the given dataset.  

The following techniques are used:

- GrayScale Conversion
- Noise Reduction using Gaussian Blur
- Canny Edge Detection
- Region of Interest Selection
- Hough Transform Line Detection

First i have applied these techniques to my test images and as per the rubric i have gotten the final output
![png](examples/Extrapolated_lines.png)


Then i used it to process video clips to find lane lines in them.


### Gray Scale Conversion

The Input images are three channel RGB images and they have to be converted
into single channel images in order to detect(edges) in them.

It can be done using the formula

Grayscale = 0.3 * RedChannel + 0.59 * GreenChannel + 0.11 * BlueChannel

This operation has to happen pixel wise globally throught the image.

![png](examples/GrayConversion.png)

### Noise Reduction using Gaussian Smoothing

Once after converting the images to grayscale the next step is to reduce
the noise in the image.

The ultimate aim is to detect the edges(rapid change in pixel intensity).

we need to smooth the edges so that the pixel intensity will be distributed
along the neighbouring pixels and the noise in the image will be reduced due
to this operation.

Kernel_size is an important parameter which should be taken into account
while designing the gaussian filter, i have used kernel_size = 7 keeping in 
mind that it suffices the requirement for the given dataset and any further 
increase in the kernel_size would have an impact on the processing time.


![png](examples/GaussianBlur.png)

### Canny Edge Detection

Once after converting applying gaussian filter the next step is to detect the edges in the image.

Canny algorithm has the following steps:

1) Gradient calculation.
   - The gradients can be determined by using a Sobel filter.
   - Its a first order derivative along both x and y axis.
   - The Magnitude and angle of the directional gradients are computed.
2) Non Maximal Suppression
    - Depending upon the direction of the gradient, every pixel will be
    compared against its immediate neighbours on both direction. if the
    current pixel is proven to be local maxima then it will be preserved
    or else it will be discarded.
3)Double Thresholding
    - Each pixel will be compared against two thresholds(lower and upper)
      the pixels whose intensity level is more than upper threshold will be
      considered as strong edges and those who fall in between upper and lower
      thresholds are called weak edges, rest will not be considered as edges.
4)Edge Tracking by Hysteresis
    - Once this the strong edges and weak edges are determined then all the
    weak edges who don't share a connection with strong edges will be removed
    and the weak edges with connection to the strong edges are preserved.


![png](examples/canny.png)


### Region of Interest Selection

Once after we detect the edges with the help of CannyEdgeDetection algorithm

we need to find to establish the working area, the region where we are
interested so as to limit our focus only within those region and eliminate
unwanted features in the image.

In this project i have considered an ROI pyramid which will foucs on the
current lane and it has three co-ordinates
`Apex = [Width/2,(Height/2)+Height*0.05]`
Apex is at the center of the frame(almost) and y-axis is shifted to 5% of its height from the center
`LeftBottom = [0 + Width*0.06,Height]`
Left Bottom is at an 6% offset on the left bottom side of the image
`RightBottom = [Width-Width*0.06,Height]`
Right Bottom is at an 6% offset on the Right bottom side of the image

![png](examples/ROI.png)

### Line Detection using Hough Line Transform

Once after the ROI is determined i have used Hough line transform to detect
the lines representing the left and right lanes

The Hough transform is designed to detect lines, using the parametric representation of a line:

rho = x*cos(theta) + y*sin(theta)

The variable rho is the distance from the origin to the line along a vector perpendicular to the line. theta is the angle between the x-axis and this vector. The hough function generates a parameter space matrix whose rows and columns correspond to these rho and theta values, respectively

- threshold: Accumulator threshold parameter. Only those lines are returned that get enough votes greater than the threshold.
- minLineLength: Minimum line length. Line segments shorter than that will be rejected.
- maxLineGap: Maximum allowed gap between points on the same line to link them.

The parameters for Hough line transform are fixed after series of expeimentation with various combinations of minLineLength and maxLineGap.

![png](examples/HoughLineTransform.png)
Finally i have drawn the detected lines onto the original images.
![png](examples/HoughLineTransform1.png)

### Line Extrapolation

Once the hough transform is applied there are multiple lines detected for a lane line. averaged line for that needs to be calculated.

As some lane lines are only partially recognized, it is necessary to extrapolate the line to cover full lane line length.

Two lanes are required one for the left and the other for the right.  
The left lane should have a negative slope, and the right lane should have a positive slope since the left top is refered as origin of the image

positive slope lines and negative slope lines are collected separately and averages are calcualted.

using the slope and intercept two lines(left,right) need to be derived.

post that there is a need to convert the slope and intercept into pixel points before ploting them.

y-coordinates are hardcoded such as y1 will always be the height of the image and y2 be 60% of the image height
as anything above 60% should lead to intersection of both lane lines.


In terms of video an track history of previous 30 frames are considered so that mean is calculated and
it helps in stabilizing the output for the current frame in case of deviation.
 



![png](examples/Extrapolated_lines.png)


### potential shortcomings with current pipeline


One potential shortcoming would be what would happen when the road is too much curvy as i have very little
time to work i am yet to figure out how to make the output stabilized for the "challenge.mp4"



### possible improvements to current pipeline

A possible improvement would be to consider a trapezoidal approach for ROI rather than a pyramid based approach.

