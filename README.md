# Udacity-CarND-Advanced-Lane-Lines
<img src="img/cover.gif" width = "800"/>  
This project was developed on windows 10 with Anaconda, jupyter notebook installed.  

## Dependencies  
* matplotlib  
* opencv
* numpy
* glob

## 1. Image Process Pipeline
<img src="img/PipeLine_1.JPG" width = "800"/>  

## 2. Image Pre-Process Pipeline
<img src="img/PipeLine_2.JPG" width = "800"/>  

### 2.1 Camera Calibration  
The purpose of camera calibration is to eliminate to image distortion caused by
physical installation error of camera mounting position or by unparallel camera
lens.  

Chessboard is introduced here for performing calibration. By applying
'cv2.findChessboardCorners', precise corner position can be pinpointed.
This function requires input of number of points in each row and column. For
example, image below has 9 points each low and 6 points each column.  
<img src="img/Pic_2_1.JPG" width = "600"/>  
It is also essential to obtain destination point indices before calculating
reverse-distortion matrix. Function *cv2.calibrateCamera* calculates matrices
for undistortion. Last step is cv2.undistort to undistort raw image.  
<img src="img/Pic_2_2.JPG" width = "800"/>

<img src="img/Pic_2_3.JPG" width = "800"/>

### 2.2 Pixel Selection
#### 2.2.1 Sobel
Multiple methods are candidates for keeping image Pixels. Instead of general
Canny edge detection, Sobel methods can provide gray scale derivative in
specific situation.  
<img src="img/Pic_2_4.JPG" width = "800"/>

#### 2.2.2 HLS Color Space
In addition to Sobel, HLS color space is also taking into consideration that it
provide useful information regardless strength of lightness.  

<img src="img/Pic_2_5.JPG" width = "800"/>

#### 2.2.3 HLS_S Channel and Sobel X
After examining results multiple methods, binary images filtered from HLS s
channel and from sobel absolute x can provide distinctive lane pixels. Final
decision is to add up these images together for preserve maximum points data
for polynomial calculation.  

<img src="img/Pic_2_6.png" width = "600"/>

### 2.3 Perspective Transformation
In order to perform transformation, source point and destination point are
essential for function cv2.getPerspectiveTransform(src, dst) to retrieve
transformation matrix. In order to determine these points, HoughLinesP can
offer straight lines and end points position. To provide noise free data for
HoughLinesP, manually select region of interest.  

Input data used here is absolute sobel x, whose capability of providing lane
contour can create decisive straight line.  

#### 2.3.1 Left and Right side ROI
<img src="img/Pic_2_7_Left_Roi.png" width = "400"/>  <img src="img/Pic_2_8_Right_Roi.png" width = "400"/>  
Right and left region of interest are separately picked to keep the function’s line
number as small as possible. Function Get_Source_Points(Binary_image,
mask_points, top_y_coornate) performs HoughLinesP averaging line and
extrapolate line to calculate end points.  

#### 2.3.2 DST Points Selection
<img src="img/Pic_2_9_Dst.JPG" width = "600"/>  
In image above green polygon shows four source points at four corners.
Easiest way is to use red intersections as upper part destination points.
Perspective transformation result is shown in next page.  

#### 2.3.3 Perspective Transformation
Now src and dst data are available, transformation matrix can be access by:
``M = cv2.getPerspectiveTransform(src, dst)`` , switch src and dst, inverse
transformation matrix is also calculated.
``Inv_M = cv2.getPerspectiveTransform(dst, src)``
To warp an image, use following function:
``output = cv2.warpPerspective( img, M, (width , height ) )``

#### 2.3.3.1 Transformation result
<img src="img/Pic_2_10_warp.png" width = "800"/>
Figure above shows transformation result on straight lane, both left and right
lane lines are shown clearly in the picture. However, when testing this on curve,
as shown in picture below, a large portion of right hand side lane is not
included. This will lead to problems while collecting pixels.  
<img src="img/Pic_2_11_warp2.JPG" width = "800"/>
A little tweak on dst points can successfully address this issue. Below are
original and modified code. 

##### Originel:
```
src = np.float32([ left_up, right_up, right_down, left_down, ])
dst = np.float32([ [left_down [0] ,0 ],
                   [right_down[0] ,0],
                   [right_down[0] , right_down[1]],
                   [left_down[0] , left_down[1]] ])

```  
##### Modified
```
offset = 150
src = np.float32([ left_up, right_up, right_down, left_down, ])
dst = np.float32([ [left_down [0] + offset ,0 ],
                   [right_down[0] - offset ,0 ],
                   [right_down[0] – offset , right_down[1]],
                   [left_down [0] + offset , left_down[1]] ])

```

Warp image with offset dst points is show in the next page. Right lane line
which is previous excluded is now inside image.
<img src="img/Pic_2_12_warp3.JPG" width = "700"/>

## 3. Lane Detection
<img src="img/Pic_3_1.JPG" width = "800"/>

### 3.1 Sliding Window
As show in the chart above, if a new image goes through lane detection
algorithm, this image will be convolved and then the peak is where the most
likely position for the lane marker  

First version of convolve algorithm is like introduced in lesson as shown below:  
```
l_center = np.argmax(conv_signal[l_min_index:l_max_index])+l_min_index-offset
```
Visualized output is shown in next page.  
<img src="img/Pic_3_2_window.JPG" width = "600"/>  
While zero points are undergone convolve, next center output subtract offset
nevertheless. It’s more reasonable to presume the next center x location
remains the same since most lane line is continuous.  
Here, a threshold and condition are added to code.  
```
if conv_signal[np.argmax(conv_signal[l_min_index:l_max_index])+l_min_index]
<= thresh:
l_center = l_center
else:
l_center =
np.argmax(conv_signal[l_min_index:l_max_index])+l_min_index-offset
```
Visualized output is shown below.  
<img src="img/Pic_3_3_window.JPG" width = "600"/>

### 3.2 Mark Line Pixels
In this part, pixels in windows as shown in previous picture are select and
colored.  
<img src="img/Pic_3_4_color_lane.png" width = "600"/>

### 3.3 Polynomial coefficient, Radius, Center
When left and right lane line pixels are separately isolated, these data are used
to calculated polynomial coefficient. Once coefficients are available, curvature
rate and road center can be computed.  
<img src="img/Pic_3_5_poly.png" width = "600"/>

### 3.4 Search around Polynomial
Once polynomial coefficients are calculated, search for lane pixels can be
more efficient by offset left lane line and focuses on pixels fall in the region.
This is useful while processing videos.  
<img src="img/Pic_3_6_searchPoly.JPG" width = "600"/>

## 4. Image Post-processing
Last step is to reverse the warped image and add visualized lane data along
with curvature rate and vehicle center position on raw image input.  
<img src="img/Pic_4_1.png" width = "600"/>

