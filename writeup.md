#**Finding Lane Lines on the Road** 

---
**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps:

1. Converted the image into grayscale.

![GrayScale](./test_images/solidWhiteCurve_grayscale.jpg)

2. Applied gaussian filter to pre-smooth the converted image.

![Smoothed](./test_images/solidWhiteCurve_smoothed.jpg)

3. Aplied Canny edge detector to figure out edge features in the image.

![AfterCanny](./test_images/solidWhiteCurve_after_canny_detector.png)

4. Limited the output image with a region of interest.

5. Applied Hough transform to the output of step 4. (the first figure: without lines connected; the second figure: lines connected)

![Hough](./test_images/solidWhiteCurve_hough.png)
![HoughConnected](./test_images/solidWhiteCurve_hough_connected.png)

6. Limited the output of step 5 with the same region of interest again to crop the drawn lines.

7. Blended with the original image to get the final output.

![Final](./test_images/solidWhiteCurve_final.jpg)

In the beginning, I swapped step 3 and step 4, which introduced artifact lines in the boundaries of region
 of interest when Hough transform was applied. That's because applying region of interest make region outside 
 black, and this make Canny detector think there are significant changing across the boundaries. So it is 
 important to apply region of interest after edge detectors.

#### About modifying the draw_lines() function
The challenging part of writing my pipeline was how to connect the lines Hough transform produced -- modifying
 the draw_lines() function. In order to extrapolating a single line for the left lane and and the right, 
 I categorized the lines into two group by the slope of the lines. 

I tried to just averaging the lines in each group to get the final single line. The output was terrible, 
 since Hough transform detected all the lines meeting the parameters I give to it, not only just the lines 
 for the lane but also the outlier.
 
 ![Hough Outlier](./test_images/outlier.png)
 
 Indiscriminately averaging the lines gave me an very very poor output. 

I then tried throwing away lines that seemed to be the outlier. Intuitively, I can expected the 
 slope for the "good" lines would roughly the same. I calculated the standard deviation for 
 the slope of the detected lines for each group and checked if it was small enough. If no,
 the pipeline thrown away line that differed with the average slope most. The process
 repeated until the standard deviation was small.
 
After filtering out the "bad" line, I fit a line for each group. I did it by treating each lines as
 two dots: (x1, y1) and (x2, y2), then applied the Least Squares method to all the dots to get the fit
 line.
 
Lastly, I calculated the slope and intercept for the fit lines, made draw_lines() draw it across the 
whole image. The rest of the pipeline would cropped the drawn lines in order to limit them inside the boundaries
of the region of interest I specified to the pipeline.

TL;DR for the works of draw_lines after I modified it:

 1. Split the detected lines into the left part and right part according to the slope of line.
 2. Removed outliers of the left lane/right lane by looking at the standard deviation of the slopes.
 3. Fit a line for the remained lines for each side by treating each line as two dots via the
  Least Squares method.
 4. By calculating the slope and intercept of the fit lines, extent it to the boundaries of the image
  and drawn.


###2. Identify potential shortcomings with your current pipeline

Since my pipeline will try to throw away lines with abnormal slopes, the throwaway-strategy it used
 is critical and may become the shortcoming of my pipeline.

First, when the input line set is too small, standard deviation and average deviation cannot telling much 
 meaningful information statistically. In fact, it will be hard to tell which one is the outlier 
 by only looking at the detected lines in the scene.

Second, when tie situation occurs -- when there are too many bad lines, there's no way my pipeline can make
 a correct decision because in such case, it is possible that the average deviations for the good lines
 are bigger than those for the bad lines, then the pipeline will happily picking the bad ones as the majority, 
 discarding the good ones.
 
For lines with similar slopes, their intercept may differ a lot. There was not abnormal intercept detection
 in my pipeline, so it might not handle well when intercept differences matter.
 
The criteria my pipeline used for determine whether it shall continue discarding lines is also tricky and added
 another parameter to be tuned. Parameters good for some cases, but it can also be not good for others.

###3. Suggest possible improvements to your pipeline

By the fact that the direction of lane lines seldom change dramatically, I think there're some improvements
 I can do with my pipeline:
 
 1. In the outlier-throwaway step, I could use the extrapolated lines from previous scenes and compare lines 
 detected in the current scene with them, and throw away those with significant bias to the lines from previous
 scenes.
 
 2. In the line fitting step, I could use the the extrapolated lines from previous scenes too, by averaging them with
 different weights. This can prevent the final output from being change drastically, even if in the bad-line-throwaway
 step the pipeline has improperly thrown away good lines.
