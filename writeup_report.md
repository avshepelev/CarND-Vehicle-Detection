##Vehicle Detection Project

###Aleksey Shepelev

###Self-Driving Car Nanodegree

---

[//]: # (Image References)
[image1]: ./writeup_images/car_not_car.png
[image2]: ./writeup_images/HOG_example.png
[image3]: ./writeup_images/sliding_window_1.png
[image4]: ./writeup_images/pipeline_performance.png
[image5]: ./writeup_images/heatmaps.png
[image6]: ./examples/labels_map.png
[image7]: ./examples/output_bboxes.png
[video1]: ./project_video.mp4

###Contents
* Histogram of Oriented Gradients (HOG)
  * Extracting HOG Features from Training Images
  * Selecting HOG parameters
  * Training the SVM Classifier 
* Sliding Window Search
  * Sliding Window Search Implementation
  * Example of Detection Performance
* Video Implementation
  * Final Video Output
  * Handling of Falso Positive Detections
* Discussion

---

###Histogram of Oriented Gradients (HOG)

####1. Extracting HOG features from training images

To extract the histogram of oriented gradients, I used the `hog()` function from the `scikit-image` module and two helper functions- `get_hog_features()` and `extract_features()` to handle the input arguments and extract any additional features. The source code can be found in the second cell of the submitted `vehicle_detection.ipynb` Jupyter Notebook. 

![alt text][image1]

####2. Selecting feature extraction parameters

I explored different colorspaces for feature extraction and tested each by comparing the classification accuracies. I found that the YCrCb colorspace yielded the highest classification accuracy on the test set and that altering parameters such as `orientations`, `pixels_per_cell`, and `cells_per_block` did not have as large an influence as the colorspace did. This is the reason these parameters were kept at the suggested values.

Below is an example image with HOG parameters extracted in the `YCrCb` color space and  `orientations=8`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`.

![alt text][image2]

In the end, I settled on the following features and parameters: 
* HOG Features
  * All YCrCb channels
  * 9 orientations
  * 8 pixels per cell
  * 2 cells per block
* Binned Color Features
  * 32x32 bin size
* Color Histogram Features
  * 32 histogram bins

####3. Training the SVM Classifier

An SVM classifier was trained on all of the provided training data after the image features were extracted using the method above. Features were scaled prior to training. Training and test set sizes are shown below.

|         | Train | Test |
|--------:|:-----:|:----:|
|     Car |  7033 | 1758 |
| Non-Car |  7169 | 1792 |

The accuracy of the classifier on the test set was above 99%. Source code may be found in cell number 5 of the submitted Jupyter Notebook

###Sliding Window Search

####1. Sliding Window Search Implementation

The sliding Window Search is implemented in the `find_cars()` function, which is located in the second code cell of the submitted Jupyter Notebook. I started with a scale of 1.5 (window size of 96px) and an overlap of 16px. This allowed me to identify deficiencies in performance on the video. I found that vehicles that were further away were not being detected reliably and added another search with a scale of 1 (window size of 64px)

Example detections at the two scales used are shown below.

![alt text][image3]

The search at the lower scale was carried out closer to the horizon as this is where small vehicles are likely to appear. The bounds of the search area are below

|          | Scale | Ystart | Ystop |
|---------:|:-----:|:------:|:-----:|
| Search 1 |   1   |   400  |  550  |
| Search 2 |  1.5  |   400  |  656  |

####2. Example of Pipeline Performance

Searching on multiple scales improved the true-positive preformance of the detector, as redundant detections on the same vehicle in different scales could be used to validate the detection.

Below are some resulting detections (after combining and thresholding multiple detections)

![alt text][image4]
---

### Video Implementation

####1. Final Video Output
The resulting video can be found [here](./project_video_output.mp4)


####2. Handling of Falso Positive Detections Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

To eliminate as many false positives as possible, I kept a detection history for five consecutive frames. From these 'raw' detections, a heat map was built and thresholded to eliminate detections that appeared only in one or two frames. 

The individual bounding boxes were then identified in the heat map with the help of `scipy.ndimage.measurements.label()`. Any detections that did not appear four times in the same location over five frames were 

Below are five frames and their corresponding heatmaps and resulting bounding boxes. Note that the heatmap from the first frame shows a object from one of the previous frames.

![alt text][image5]

---

###Discussion

Overall, the pipeline has reasonably good performance from the standpoint of being able to detect the presence of other vehicles that are moving in the same direction as the video camera. The pipeline is even capable of merging and splitting targets when they get closer and further apart.

However, one of the most important function of vehicle detection by camera is the measurement of the distance to the vehicle of interest and the relative velocity of this vehicle. In order to determine distance, the bounding box around the vehicle must be much more accurate (especially at the interface with the ground), and this is a limitation of the pipeline presented here. 

In addition, one of the limitations of the pipeline is the ability to distinguish between one vehicle and multiple vehicles if the vehicles are close together (in addition to determining where one vehicle ends and another starts). The detection performance is also as good as the training set. As a result, the current pipeline may or may not work on trucks, buses, and motorcycles - all motor vehicles capable of traveling on the highway.