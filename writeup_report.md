
---

**Vehicle Detection Project**

[//]: # (Image References)
[image1]: ./writeup_images/car_not_car.png
[image2]: ./writeup_images/HOG_example.jpg
[image3]: ./examples/sliding_windows.jpg
[image4]: ./examples/sliding_window.jpg
[image5]: ./examples/bboxes_and_heat.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

###Contents
* Histogram of Oriented Gradients (HOG)
  * Extracting HOG features from training images
  * Selecting HOG parameters
  * Training the SVM Classifier 
* Sliding Window Search
* Video Implementation
* Discussion

###Histogram of Oriented Gradients (HOG)

####1. Extracting HOG features from training images

To extract the histogram of oriented gradients, I used the `hog()` function from the `scikit-image` module and two helper functions- `get_hog_features()` and `extract_features()` to handle the input arguments and extract additional features. The source code can be found in the second cell of the submitted `vehicle_detection.ipynb` Jupyter Notebook. 

![alt text][image1]

####2. Selecting feature extraction parameters (Explain how you settled on your final choice of HOG parameters.)

I explored different colorspaces for feature extraction and tested each by comparing the classification accuracies. I found that the YCrCb colorspace yielded the highest classification accuracy on the test set and that altering parameters such as `orientations`, `pixels_per_cell`, and `cells_per_block` did not give such a large effect. This is the reason these parameters were kept at the suggested values.

Below is an example image with HOG parameters extracted in the `YCrCb` color space and  `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`.

![alt text][image2]

In the end, I settled on the following features and parameters: 
* HOG
  * All YCrCb channels
  * 9 orientations
  * 8 pixels per cell pix_per_cell = 8
  * 2 cells per block
* Binned Color features
  * 32x32 bin size
* Color histogram features
  * 32 histogram bins

####3. Training the SVM Classifier (Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).)

An SVM classifier was trained on all of the provided training data after all features were extracted using the method above. All features were scaled prior to training. 

|         | Train | Test |
|--------:|------:|-----:|
|     Car |  7033 | 1758 |
| Non-Car |  7169 | 1792 |

###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to search random window positions at random scales all over the image and came up with this (ok just kidding I didn't actually ;):

![alt text][image3]

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video.mp4)


####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps:

![alt text][image5]

### Here is the output of `scipy.ndimage.measurements.label()` on the integrated heatmap from all six frames:
![alt text][image6]

### Here the resulting bounding boxes are drawn onto the last frame in the series:
![alt text][image7]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

