## Vehicle Detection Project
----

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.


[//]: # (Image References)
[image1]: ./output_images/detected_bounding_boxes1.jpg
[image2]: ./output_images/detected_cars1.jpg
[video1]: ./project.mp4


## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

This is the writeup. 


### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the section named "Step 2" of the IPython notebook located in "./P5.ipynb". 

`skimage.hog()` function is used to extract HOG features. As in the lectures, the parameters used for the HOG feature extraction are `orientations=9`, `pixels_per_cell=(8,8)`, `cells_per_block=(2,2)`. 


#### 2. Explain how you settled on your final choice of HOG parameters.

For limited amount of time, the HOG parameters were left unchanged in the project. Instead, color spaces were explored: `RGB`, `HLS` and`YCrCb`. 

After many trial-and-errors, with a combination of the color features it was found that relatively more features tended to be extracted with `RGB`, relatively fewer features tended to be extracted with `HLS`. `YCrCb` is in the middle of the results of `HLS` and `RGB`. 

From the above, `YCrCb` was chosen because this would have fewer problems of many false positives and few true positives. 


#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The code for this step is contained in the section named "Step 3" and "Step 4" of the IPython notebook located in "./P5.ipynb". 

A linear SVM was used as the classifier. Along with the HOG features with 3 channels, the spatial features and the histogram features with 3 channels were used for the training. The parameters used for the feature extraction are summarized in the section named "Step 4 Test". 

In `trainClassifier()`, `extractFeatures()` extracts each features in the order of the spatial features, histogram features and HOG features of cars and objects that are not cars, and creates a feature vector by concatenating them for each training image sample. Then, after a normalization, the feature vectors were separated into the training part and the test part, and the training part was fed into the classifier. 

The number of samples used for the training was `3000`. This was the limit that my laptop was able to handle. In order to avoid some bias of the sample, the samples were randomly picked up from each data sets of `vehicles` and `non-vehicles` in the GTI vehicle image database and the KITTI vision benchmark suite. 


### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The HOG sub-sampling window search described in the lecture was explored first. However, after many trial-and-errors, the sliding window search described in the lecture was explored. 

The main reason of decision is that HOG sub-sampling window search has relatively more parameters and would be difficult to figure out how it behaves in a limited amount of time. The parameters of the sliding window search are essentially two, the window size and the overlap ratio of the windows at search. 

The value of these two parameters were also decided by trial-and-errors. First of all, the window size `(96, 96)` and the overlap ratio `(0.5, 0.5)` was used. However, cars detected were not fully enclosed in a bounding box. Then, the window size was changed to `(128, 128)`. After some trials with a parameter to control false positives, the overlap ratio was changed to `(0.8, 0.8)`. Although the processing time was increased by this change, cars detected were more likely to be enclosed in a bounding box. Finally, the window size was changed to `(96, 96)` to get a better result. 

Here is an example of output from the sliding window search. 

![alt text][image1] <!-- .element height="450" width="650" -->

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately, 3-channel HOG features plus spatially binned color and histograms of color in the feature vector were used.  Here is an example image:

![alt text][image2] <!-- .element height="450" width="650" -->

---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

This is a link to [the final video result](./project.mp4)

The test video was also used to decided the parameters such as the color space, window size, overlap ratio and the threshold described below. 

This is a link to [a test video result](./test1.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

The code for this step is contained in the section named "Step 6" of the IPython notebook located in "./P5.ipynb". 

The positions of positive detections in each frame of the video was recorded. From the positive detections, a heatmap that indicates likelihood of vehicles detected was created by adding `1` into the corresponding image pixels on the map and then thresholded to identify vehicle positions. `scipy.ndimage.measurements.label()` is used to identify individual blobs in the heatmap. 

The threshold was applied to heatmaps from multiple successive frames. Namely, the sum of the values on the heatmaps over successive frames was calculated, and the value of the corresponding image pixels on the map got `0` if the sum was below the threshold. 


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The first issue I faced was the feature extraction and classifier training. Using a single feature, e.g. HOG feature, would be fine. However, using multiple features required careful implementation to me. 

Because multiple features are used to create a single feature vector for each training image and compared with the one created from an image in the video, the construction of both of the feature vectors has to be the same, e.g. the order of concatenation of those features into a feature vector. 

I noticed this fact after a while. I would design and implement cleaner code to address this issue if I were going to pursue this project further. 

The second issue I faced was relations between the color spaces and features, i.e. how differently features are shown and fit to vehicle detection problem depending on color spaces. I did not have enough time to explore this in detail. If I had understood the relations more, I would have been able to do this project more efficiently. 
