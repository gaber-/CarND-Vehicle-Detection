**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./saved/vehicle.png
[image2]: ./saved/non_vehicle.png
[image3]: ./saved/vehicle_hog.png
[image4]: ./saved/non_vehicle_hog.png
[image5]: ./saved/finished.png
[image6]: ./saved/heat1.png
[image7]: ./saved/label1.png
[image8]: ./saved/boxes1.png
[image9]: ./saved/heat2.png
[image10]: ./saved/label2.png
[image11]: ./saved/boxes2.png
[image12]: ./saved/heat3.png
[image13]: ./saved/label3.png
[image14]: ./saved/boxes3.png
[image15]: ./saved/heat4.png
[image16]: ./saved/label4.png
[image17]: ./saved/boxes4.png
[image18]: ./saved/heat5.png
[image19]: ./saved/label5.png
[image20]: ./saved/boxes5.png
[image21]: ./saved/heat6.png
[image22]: ./saved/label6.png
[image23]: ./saved/boxes6.png

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first code cell of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]
![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an ecample

![alt text][image3]
![alt text][image4]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and chose 16 pixels per cell, 9 orientations and 4 cells per block as it gave a good performance with few parameters

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the selected HOG parameters (cells 2-13), I tried various histogram features (on HSV and RGB) and spatial binning on an 8*8 image, but chose to only use the saturation, Cb component of YCrCb and spatial binning on a 8*8 downscaled version of the image.

I noticed that applying a histogram equalization to the image passed to the HOG function seems to improve the result.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I applied different sizes of sliding window at different heights (15-17), to account for perspective:

I chose a "generous" 0.8 overlap for the 124*124 window and 0.6 or 0.5.

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector.  Example:

![alt text][image5]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./out/project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video (cells 18-26). From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

I carry over the old heatmap to te new frame of the video, and divide it to make the reading more stable.

Here's an example result showing the heatmap from a series of images, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image6]
![alt text][image9]
![alt text][image12]
![alt text][image15]
![alt text][image18]
![alt text][image21]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image7]
![alt text][image10]
![alt text][image13]
![alt text][image16]
![alt text][image19]
![alt text][image22]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image8]
![alt text][image11]
![alt text][image14]
![alt text][image17]
![alt text][image20]
![alt text][image23]



---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

The first issue I faced was the huge number of parameters and features that could possibly be used. I tried naively applying HOG, color histogram, spatial binning and HSV together, and then chose the most significative features, and tuned the techniques individually.

Then I noticed the car identification was kind of rigid: if the car didn't fall precisely inside a window it would not get identified. I solved this by increasing the overlapping of windows.

Lastly the heatmap generated had some noise, so I added a high threshold.
