**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[car]: ./examples/car.png
[notcar]: ./examples/notcar.png
[hog]: ./examples/hog.png
[sliding]: ./examples/sliding.png
[scale1]: ./examples/scale1.png
[scale2]: ./examples/scale2.png
[find_car_windows]: ./examples/find_car_windows.png
[step1]: ./examples/step1.png
[step2]: ./examples/step2.png
[step3]: ./examples/step3.png
[video1]: ./project_video_output.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the #1 - #8 cells of the IPython notebook `code.ipynb`.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

Vehicle:

![alt text][car]

Non-vehicle:

![alt_text][notcar]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the first channel of `RGB` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

![alt text][hog]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters, for example, `LUV`/`YUV`/`RGB` for color space, different number of orient, different number of pix_per_cell, cell_per_block. Finally I picked the best one based on the performance of SVM classifier. 

The final paramters chosen were: color space = `LUV`, orient = 8, pix_per_cell = 8, cell_per_block = 8. And also, I chose spatial_size = (16,16) for spatial color features, and number_of_histogram_bins = 32 for color histogram features.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM with default parameters of sklearn library. (Cell #9 in the notebook) I used HOG features mentioned above, and also spatial intensity and channel intensity histogram features. The final accuracy on test data is 0.9865.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I mostly adpated the `find_car()` function in the course material, with a small modification to support xstart/xstop range. See the code in Cell #15 in the notebook.

Specifically, I chose two different scales with 50% overlap:
1) 0.75, which is used for detecting the far-away cars
3) 1.5, which is used for general scanning of the whole range

I chose these scales mostly by trial-and-error. Based on my experient, this combination can provide acceptable accuracy.

Below are the windows for two scales:

![alt text][scale1]
![alt text][scale2]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

The straight-forward sliding window implementation will be slow, because of a lot of duplicated HOG calculation. The speed optimization is to calucate all the HOG features in one batch, as in the function `find_car()`.

Ultimately I searched on two scales using the L-channel of LUV colorspace HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result. Here are some example images:

![alt text][find_car_windows]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected. The corresponding code is in Cell #17/18 of the notebook.

Here's an example result showing the heatmap from a frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here is one frame and its corresponding heatmap:

![alt text][step1]

### Here is the output of `scipy.ndimage.measurements.label()` on the heatmap:
![alt text][step2]

### Here the resulting bounding boxes:
![alt text][step3]

To eliminate the jiggle between frames, I used a "smoothing" apporach - basically accumulate the heatmap of multiple recent frames.

Besides heatmap, I also used two another heuristic apporaches to filter out the false positive:
1) Set xstart and xstop range, to make sure the scanning range only contains the road lanes only.
2) Filter out the boxes with invalid width / height ratio (The bounding boxes should be rectanges not too far away from square).

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The main problem I met was the false positive detection, some non-vehice things are easily considered as cars, like mud on the road, tree along the side of road. I tried a few approaches to eliminate them, but there are still some false-positive existing.

I think when the enviormental conditions become more complicated (like white car on bright road, or rainy weather), my pipeline will more likely fail. And also if we met some types of vehicles which are significant different from that in the training set (like a big truck?), my pipeline will not give good results.

I believe that the better approach, given enough time to pursue it, would be considering the following improvements:
* work together with the lane detection, to accurately set the scan range to be on the road
* determine the direction and speed of the vehicles by analyzing adjacent frames
* use convolutional neural network instead of sliding window search







