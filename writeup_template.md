# **Finding Lane Lines on the Road** 

---

** Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)
[image1]: ./examples/0-solidYellowLeft.png "Original"
[image2]: ./examples/1-yellow-and-white.png "White and Yellow"
[gray-blur]: ./examples/3-grayscale-blur.png "Grayscale, Blurred"
[canny-edge]: ./examples/4-canny-edge.png "Canny Edge"
[5-hough-lines]: ./examples/5-hough-lines.png "Hough Lines"
[6-mask-relevant-region]: ./examples/6-mask-relevant-region.png "Mask Region"
[6.5-regress]: ./examples/6.5-regress.png "Regress Lines"
[7-regress-and-overlay]: ./examples/7-regress-and-overlay.png "Final Result"
[chal-masked]: ./examples/challenge/masked_image.jpg "Challenge Masked"
[chal-result]: ./examples/challenge/regressed_image.jpg "Challenge Result"
[mask-more]: ./examples/10-mask-more.png " "
[mask-less]: ./examples/11-mask-less.png " "

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline consists of seven steps. To illustrate the effect of each step I will use solidYellowLeft.jpg image as input:

![Source Image][image1]


#### A) Detect Yellow and White objects

First, we select objects colored yellow or white. For white, keep the pixels with the color > (200, 200, 200). For yellow, keep the pixels where (Red > 150, Blue > 150, and Green < 100).

![White and Yellow][image2]

#### B) Convert to Grayscale

Convert to grayscale and apply a blur filter to smooth out the gradient, and eliminate possible noisy gradient jumps that might appear when we do Canny edge detection:

![Grayscale, Blurred][gray-blur]

#### C) Canny Edge Detection 

Apply Canny edge detection algorithms to find edges in the grayscale image:

![Canny Edge][canny-edge]


#### D) Hough Line Detection

Apply Hough like detection algorithm to the image of edges, obtain in the previous step. This will combine distinct dots in the Canny edge image into contigous lines. These lines could be quite short though, combining just a few pixels at a time. We'll deal with this in the subsequent steps.

![Hough Lines][5-hough-lines]

#### E) Mask Relevant Region

Now we will apply the mask to select objects in the middle lower part of the image, and remove everything. We will use a quadrilateral mask:

![Mask Region][6-mask-relevant-region]

#### F) Apply Linear Regression

To produce nice stable lines for each lane we will use a linear regression. 

First, we split the image into the left and right halves at the middle, to separate the two lines. Then, we use `sklearn.linear_model.LinearRegression` to identify a single line that best fits the dots in the left half. 

Repeat the same process with the right half. The next image shows the resulting lines overlaid with the masked image from the previous step: 

![Regress Lines][6.5-regress]

#### G) Final Result

Finally, we can overlay the lines from the previous step with the original image, to show how our lines fit the original lane markings:

![Final Result][7-regress-and-overlay]

#### H) Notes on `draw_lines()`

I *didn't* modify the `draw_lines()` function. Instead, I introduced a separate step, (F) Regression, that generated the lines based on the set of points identified in the previous steps. This way we can change our line detection algorithm later (which we likely will), but still can use `draw_lines()` to do actual drawing.


### 2. Identify potential shortcomings with your current pipeline

A) One of the shortcomings of this pipeline are the arbitrary constants we used to select white and yellow objects. We defined these colors using specific values for red, green, and blue channels, e.g. white meant pixels with color > (200, 200, 200). These values worked reasonably well for our test images. But, there's no guarantee they will continue to work as well when applied to other arbitrary images of various roads under varying light conditions.

B) We made the assumption that our region of iterest is in the middel lower part of the image. This region may not work for other images. It may result in selecting unrelated objects, or excluding the actual lanes. Our detected lines may end up misaligned, misplaces, or missing altogether.

Overall, this algorithm is very fragile to changes in color of objects, lightning conditions, and the layout of the road, cars, and other objects. 


### 3. Suggest possible improvements to your pipeline

#### Do not use color assumption

One of the arbitray assumptions we've made is the color of markings: yellow and white. These are needed to solve the challenge problem. 

Using only only edge detection and region masking introduces fewer assumptions, but fails for the challenge video. We end up with the image like this:

![chal-masked]

The noise at the bottom of the image lead to the wrong lane lines:

![chal-result]

One way to eliminate this noise is to use a more advanced regression and classification technique. We know that we are looking for the sets of points that fit well on a line, and are pointed in the upward direction. We can use clustering algorithm first to collect the points fitting on each line, then eliminate the lines outside our region of interest.

These are more general assumptions and they should be more robust to changes in scenery, lightning, etc. 

#### Relax the region of interest assumption

The same approach can be used to rely less on masking. If we have a more robust algorithm to detect upward looking lines, we can allow more objects into our region of interest. Currenlty we have to aggresively exclude extra objects to elminate noise, and we lose the top part of our lines, e.g. compare the following images:

![mask-more] ![mask-less]

With a more robust line detection algorithm, we can relax the masking region, and have a better fit of our lines to the original lane markings.


#### Use higher degree polynomials to approximate lane markers

The linear approximation may be very imprecise in case of sharp turns, when the lane markings curve. Using a quadratic, or a higher-degree polynomial will help fit curving lines better.
