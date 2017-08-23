# **Vehicle Detection and Tracking** 

## Setup

### Installation

Runs Jupyter Notebook in a Docker container with `udacity/carnd-term1-starter-kit` image from [Udacity][docker installation].

```
cd ~/src/CarND-Trafic-Sign-Classifier-Project
docker run -it --rm -p 8888:8888 -v `pwd`:/src udacity/carnd-term1-starter-kit
```
Go to `localhost:8888`


## Reflection

### A. Histogram of Oriented Gradients (HOG)
##### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

I started by reading in all the 'vehicle' and 'non-vehicle' images. Here is an example of one of each of the 'vehicle' and 'non-vehicle' classes:

![alt text][image1]

I then defined functions for getting the features from images via `skimage.hog()` and tested various HOG parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`). Here's an example of HOG (with `orientations=9`, `pixels_per_cell=8`, and `cells_per_block=2`) applied to a 'vehicle' and 'non-vehicle' image:

![alt text][image2]


##### 2. Explain how you settled on your final choice of HOG parameters.

I tested various parameters:
- color_space (from RGB, HSV, LUV, HLS, YUV, YCrCb)
- orientations
- pixels_per_ell
- cells_per_block 
- hog_channel (from 0, 1, 2, 3, ALL)

I experimented with different `color_space` and `orientations` as well as `pixels_per_cell` and `cells_per_block`.

My final configuration was
```
color_space = 'YCrCb'
orientations = 9
pixels_per_cell = 8
cells_per_block = 2
hog_channel = 'ALL'
```

##### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using `sklearn.svm.LinearSVC()`. I concatenated the extracted features from the 'vehicle' and 'non-vehicle' datasets. Then I normalized the combined dataset because of the extracted features from spatial binning of color and histogram of color with `sklearn.StandardScaler()`. Finally, I shuffled the dataset and split it with a 80/20-split into training and testing datasets.

With the above configuration and additional parameters for the other two feature extractions (code block 8) I achieved a 99.27% accuracy rate on the test dataset.
```
color_space = 'YCrCb'
spatial_size = (32, 32)
hist_bins = 32
hist_range = (0,256)
orient = 9
pix_per_cell = 8
cell_per_block = 2
hog_channel = 'ALL'
spatial_feat = True 
hist_feat = True
hog_feat = True
```

### B. Sliding Window Search

##### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I decided to that only the lower half of the image is relevant for searching for cars as it is unlikely to find cars in the sky. So from `y=400px` to `y=720px` was the range of interest for my sliding window search function.

Cars can appear bigger or smaller on an image depending on its distance to the camera. So I applied a larger scaling factor to the pixel range closer to the end of the image (near `y=720px`) and a smaller scaling factor closer to the horizon (from `y=400px`) as can be seen in `code blocks 13`, `14`, and `15`. Here's an example of how to scaled boxes look like on the image:

![alt text][image3]

##### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately, I searched on 4 scales (`1.0, 1.5, 2.0, 3.5`) using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]

### C. Video Implementation

##### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result][videolink]


##### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I created a `class Vehicle()` with `previous_rectangles` and `add_rectangles()`. 
For each frame of the video, I recorded the positions of positive detections. From the positive detections I created a heatmap and added previously recorded rectangles. Whenever I found new rectangles in a frame, I added them to the `Vehicle()` instance with `add_rectangles()` and removed any old boxes (beyond the 15th rectangle in my case). I used `scipy.ndimage.measurments.label()` to identify blobs in the heatmap. I then assumed each blob corresponded to a vehicle. I constructed bounding boxes to cover the area of each blob detected.  

Here's a link to the video with a smoother vehicle detection using the above mentioned "remembering of vehicle positions": [Updated video][videolink2]

### Here are six frames and their corresponding heatmaps:

![alt text][image51]
![alt text][image52]
![alt text][image53]
![alt text][image54]
![alt text][image55]
![alt text][image56]



### D. Discussion

##### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 


[docker installation]: 				https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/doc/configure_via_docker.md

[image1]: 					./output_images/image1.png 
[image2]: 					./output_images/image2.png 
[image3]: 					./output_images/image3.png 
[image4]: 					./output_images/image4.png 
[image51]: 					./output_images/image51.png 
[image52]: 					./output_images/image52.png 
[image53]: 					./output_images/image53.png 
[image54]: 					./output_images/image54.png 
[image55]: 					./output_images/image55.png 
[image56]: 					./output_images/image56.png 
[videolink]: 				./output_images/result.mp4
[videolink2]: 			./output_images/result_2.mp4
