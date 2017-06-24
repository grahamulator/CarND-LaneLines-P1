# **Finding Lane Lines on the Road**

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image0]: ./examples/pipeline_steps/00_Original.jpg "Original"
[image1]: ./examples/pipeline_steps/01_Grayscale.jpg "Grayscale"
[image2]: ./examples/pipeline_steps/02_Gaussian.jpg "Gaussian"
[image3]: ./examples/pipeline_steps/03_Canny.jpg "Canny"
[image4]: ./examples/pipeline_steps/04_Interest.jpg "Interest"
[image5]: ./examples/pipeline_steps/05_Hough.jpg "Hough"
[image6]: ./examples/pipeline_steps/06_Final.jpg "Final"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.
As an example, the original image that runs through the pipeline looks something like this:
![alt text][image0]

I run this image through my pipeline which consists of the following 6 steps:
##### 1. Convert original image to grayscale
This reduces the dimensionality of the image.
![alt text][image1]
##### 2. Apply a Gaussian blur
This smoothes the image for the next step.
![alt text][image2]
##### 3. Run Canny edge detection
As the name indicates, Canny edge detection attempts to find edge features in the image, drawing white pixels on edges and black pixels on non-edges.
![alt text][image3]
##### 4. Mask off an area of interest
Here I manually carve out an area of the image where I expect to find lanes. This will ensure that I'm not looking in places like the sky or far periphery for road lines.
![alt text][image4]
##### 5. Apply a Hough transformation and call `draw_lines()`
First, the Hough transformation generates a set of line segments from the previous image. Then, `draw_lines()` is called to draw the left and right lane markers. I modified draw lines to do the following:
- Calculate the slope of all line segments from the Hough transform and discard outliers which may have been generated from noisy pixels or horizontal markings.
- Separate the remaining line segments based on positive/negative slope which represent the right/left lane markings respectively.
- For each set of segments, find the point where a given segment, if extended, would intersect the bottom of the image and the top of the area of interest, then weight these points based on the length of the line segment. This will give more confidence to longer line segments and less confidence to shorter line segments.
- Take the weighted average of these points to find one line (for each side) which is the weighted average of all the line segments.
- Draw the line for each side.

![alt text][image5]
##### 6. Overlay lines onto original image
Finally, overlay the drawn lines onto the original image.
![alt text][image6]

### 2. Identify potential shortcomings with your current pipeline

Although my pipeline proved to be robust with the example videos, it is not designed to generalize well on an actual vehicle in the field. A few shortcomings come to mind.

1. The algorithm would certainly fail if the vehicle shifted lanes. Using the example image above, imagine if the vehicle shifted left. The left lane marking would slowly become more vertical in the field of vision, and my method discards overtly vertical line segments. Although it's uncertain what would happen, my guess is that the left drawn line would become erratic as the vehicle switches lanes, and then disappear entirely.

2. Similarly, if a vehicle shifts into the field of view too close to the camera, this could obscure the lane markings and make detection unreliable.

### 3. Suggest possible improvements to your pipeline

1. One possible improvement would be a more stable line drawn on the dashed lane line. In comparison to the solid lane line, it is slightly jittery in the video analysis. Perhaps adjusting how detected line segments are weighted and aggregated could smooth this.

2. Although outside the scope of this project, another improvement would be incorporating a sense of memory, meaning that the system would have a record of the previously detected lines and use that in calculating the lines for the current image. This would likely help with the issues of line jitter and vehicle obscuration mentioned above.
