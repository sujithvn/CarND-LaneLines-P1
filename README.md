# Finding Lane Lines on the Road

__Note__: Please refer this [link](Project-1-Finding-Lane-Lines.ipynb) for detailed write-up with images. Same available on my [site](https://sujithvn.me) as well.

## Reflection

In this project, we are trying to identify and mark the lane lines on the road. We first identify it on a image and later employ the same functions to use it in a video stream. Primarily we will be using Python and OpenCV for this project. The steps involved can be broadly classified into few major steps/techniques. 

The following steps/techniques are used:

1. Identifying the best color space & Filtering out the required Color
2. Canny Edge Detection on Gray scaled & smoothed images
3. Region of Interest Selection via Masks
4. Using Hough Transform for Line Detection
5. Fine tuning via Averaging and Extrapolating Lines
6. Visual verification
7. Processing Video streams and final output

### 1. Pipeline

Initially we compare all the three (RGB, HSV, HSL) Color Spaces. Once we have decided to use the HSL Color Space images after comparing , we developed a pipeline which covers the steps from 1 to 5 above. These steps are detailed as under.

#### 1.1- Identifying the best color space

We visualized the images in three Color Spaces [RGB, HSV, HSL] and compared the how the lane lines are visible in all the cases. The Yellow and White lane lines were most identifiable in the HSL format when it was filtered out.

#### 1.2- Canny Edge Detection on Gray scaled & smoothed images

We use the techinique of Canny Edge Detection to detect edges in the images (the major shapes in the image). This technique works better when the image is GRAY scaled and SMOOTHED in order remove unwanted noice.
**Gray Scaling**
Before using Canny Edge detection, we shall convert the images to GRAY scale as it helps to identify the shapes/edges better (better pixel intensity difference).
**Smoothing using Gaussian Blur**
Edges are detected by identifying the rapid change of pixel intensity. But there are certain noice factor in the images due to rough edges which impacts the detection of lines. So we smooth out the edges before trying to detect them. The value of the kernel-size determines the size of matrix used to smooth (this should be a positive and odd number). A small number will retain the sharpness and a large number will make it blur. Fine-tune and find a good value.
**Edge Detection**
We would be using two parms - lower-threshold & upper-threshold. The values are selected based on the below criterias.
- Pixel gradient higher than the upper threshold are retained
- Pixel gradient lower than the lower threshold are rejected.
- Pixel gradient is between the two thresholds are accepted only if it is connected to a pixel that is above the upper threshold.
  Use trial and error to identify the values here which will retain the required edges and remove the noice.

#### 1.3- Region of Interest Selection via Masks
We shall focus our attention to a polygon shape where we expect to find the lane lines and ignore the remaining areas in the image (like other lanes, sky, hills). We do this basically by applying a mask with the below steps:
- Define the polygon by identifying the four vertices.
- Define the mask color (black) and shape (based on channels) 
- Except for the region bounded by the vertices, other area is set to black(0).

#### 1.4- Using Hough Transform for Line Detection
We use Hough transform to identify lines in the edge images. There are multiple parameters to be defined here.
- rho and theta are the Hough Space parameters
- threshold is the minimum votes required for the selected lines
- min line length is the minimum length of the line segment to be accepted
- max line gap is the max value for the gap allowed between points on single line

Please refer cv2.HoughLinesP in opencv documentation for more details.

#### 1.5- Fine tuning via Averaging and Extrapolating Lines
In the process of fine-tuning the detected lines, the multiple lines detected for a lane line should be averaged to a single line and where ever the lines are partially recognized, it should extrapolated to make it a full lane line.
Additionally, classify the lane line as left/right based on teh slope of the line. Normally the left lane comes with positive slope, but in the image y-coordinate is from top-to-bottom, hence left lane has a negative slope and right-lane has positive slope.

**Averaging multiple lines to find the lane-line**
The steps performes are:
- Accumulate the left and right lines (in slope/intercept form) and their weights (length)
- Deciding whether this is a left/right lane is based on the slope
- More weight is added to longer lines    
- Output is the average slope and intercept for the left and right lanes of each image.

**Notes:**
- We will ignore the vertical lines with slope of INF
- neg-slope is left-lane since y is reversed in image
- pos-slope is right-lane since y is reversed in image
- Used cv2.addWeighted function to superimpose the lane-lines on the image rather than mutating the image.

#### 1.6- Visual verification
Let us pull all things together and see how our final image would be. Input will be the images and list-of-lines (from hough lines).

#### 1.7- Processing Video streams and final output
The output looks good and we are good to process the video files. We define a class which carries out the entire pipeline and use the same for processing the video file.

The notebook includes the final output of the video files.


### 2. Identify potential shortcomings with the current pipeline

There are few potential shortcoming in the entire solution.
1. We are identifying our region-of-interest as a ploygon where we expect to find lane-lines. Although, this helps for a start, it may face issues, especially like road-junctions.

2. On a finer point, currently we are handling straight lines comfortably. He also, we would be facing issues when the roads curve beyond a limit. It could also impact if the road has a steep slope (up or down).

   ​

### 3. Possible improvements to the pipeline

Possible improvement would be to address the above mentioned shortcomings.

1. The program needs to be robust in handling various road conditions. Challenges would be different lighting conditions, other road conditions (like curves, slopes, tunnels, underpass, bridges, etc).
2. The region-of-interest may also need to adapt to the road-conditions. For example, if we are on a steep slope, the current algorithm where it is hard-coded to half-way of image needs to change. 

### Thank you

