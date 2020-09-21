# Udacity-CarND-Advanced-Lane-Lines
<img src="img/cover.gif" width = "800"/>  
This project was developed on windows 10 with Anaconda, jupyter notebook installed.  

## Dependencies  
* matplotlib  
* opencv
* numpy
* glob

## 1. Image Process Pipeline
<img src="img/PipeLine_1.JPG" width = "600"/>  

## 2. Image Pre-Process Pipeline
<img src="img/PipeLine_2.JPG" width = "600"/>  

### 2.1 Camera Calibration  
The purpose of camera calibration is to eliminate to image distortion caused by
physical installation error of camera mounting position or by unparallel camera
lens.  

Chessboard is introduced here for performing calibration. By applying
'cv2.findChessboardCorners', precise corner position can be pinpointed.
This function requires input of number of points in each row and column. For
example, image below has 9 points each low and 6 points each column.  
<img src="img/Pic_2_1.JPG" width = "400"/>
It is also essential to obtain destination point indices before calculating
reverse-distortion matrix. Function *cv2.calibrateCamera* calculates matrices
for undistortion. Last step is cv2.undistort to undistort raw image.  
<img src="img/Pic_2_2.JPG" width = "600"/>
<img src="img/Pic_2_3.JPG" width = "600"/>

### 2.2 Pixel Selection
#### 2.2.1 Sobel
Multiple methods are candidates for keeping image Pixels. Instead of general
Canny edge detection, Sobel methods can provide gray scale derivative in
specific situation.  
<img src="img/Pic_2_4.JPG" width = "600"/>

#### 2.2.2 HLS Color Space
In addition to Sobel, HLS color space is also taking into consideration that it
provide useful information regardless strength of lightness.
<img src="img/Pic_2_5.JPG" width = "600"/>


## Line Extrapolation  
<img src="img/non_e.gif" width = "400"/> <img src="img/extrp.gif" width = "400"/>
<img src="img/Pipi_Extrap.JPG" width = "400"/>

## Shortcoming  
While extrapolating lines, only slope are considered as threshold to
categorized points set into left or right group. Which means the outputs can be
sever influence by road signs, cars ahead and split lane lines or any objects
with edges can pass Hough transformation criteria.
