# **Finding Lane Lines on the Road** 

## Writeup 

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./test_images_output/yellow_white_result.png "White And Yellow Mask"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

I had a few pipelines that changed all the time until it fitted all situations.

My first pipline consisted of few steps:
1) Convert to grayscale image to reduce complexity and for later edge detection
2) Blur the image a bit, to remove unnccessary details(noise)
3) Find edges with Canny's edge detection
4) Get out region of intereset(where our lanes are) and mask the edges image with it.
I chose the upper region to be where the "horizon" is, in all images it's in the same position.
5) Run the Hough transformation, I generalized its values until I got a good result.
    + Return image with new lines by hough lines
6) Override the original image with the new processed image.

This pipeline worked good, but it's not what I was asked to do, I was asked to draw fixed lines on the lanes.
How should we modify draw_lines function?
I needed a way to find the best line fit for all the different lines I got
, simple linear regression(least squares, minimize squared error) was the obvious winner.
y = b + mx ==> x = (y-b)/m, from this equation and slope and intercept values, I could find the x's that best fit 
the bottom y which is the image height, and the upper y which is the "horizon".

That did the job, but on the second video(11th second), the line fit was off because of an horizontal line on the road.
I needed a way to filter out this anomalous lines.
I tried to do it with a relative approach of removing everything that is more than 2 standard deviations away from the mean.
This approach didn't work, as we have small amount of samples(lines), so every sample as a huge affect on the mean.
Median and precentile approach didn't work as well, as it filtered out good samples.
My last approach was to define valid boundaries for slopes, which worked pretty well after some tuning.

So I passed the second video, and then came the challange video where my lines were way off.
I modified the video with hough lines and anomaly filter off just to see why it's so off, I saw that shade and non smooth road(differnt colors) are detected as edges and therefore as lines.
The grayscale was not good enough, I must filter out by color and not just by intensity of pixel.
After some reading, I saw that it's easy to do it with hsv, adjusted the parameters of white and yellow(lane colors)
and outputed the image below, it worked.

![alt text][image1]

Added this step before grayscaling, but it wasn't enough.
The anomalous slope removal has left us with no valid lines for left lane at some frames, the best idea is to continue
with the same slope as the last frame, so I turned all the points lists to global variables, and overrided it only if 
I had valid lines.

After that the lane detection is much better, still not perfect though.


### 2. Identify potential shortcomings with your current pipeline


Few potential shortcomings:
1) Winding roads
2) Different position of camera
3) Different image resolution,angle,quality
4) Different road conditions
5) Relative/absolute parameters not working for other situations


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to do a polynomial curve fitting.

Another improvment would be to automatically identify the horizon and the region of interest 
that will fit different conditions.
