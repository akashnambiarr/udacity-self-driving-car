# **Project1 : Finding Lane Lines on the Road** 

## Introduction

For a self driving vehicle to stay in a lane, the first step is to identify lane lines before issuing commands to the control system. Since the lane lines can be of different colors (white, yellow) or forms (solid, dashed) this seemingly trivial task becomes increasingly difficult. Moreover, the situation is further exacerbated with variations in lighting conditions. Thankfully, there are a number of mathematical tools and approaches available nowadays to effectively extract lane lines from an image or dashcam video. In this project, a program is written in python to identify lane lines on the road, first in an image, and later in a video stream. After a thorough discussion of the methodology, potential shortcomings and future improvements are suggested.

## Methodology

Before attempting to detect lane lines in a video, a software pipeline is developed for lane detection in a series of images. Only after ensuring that it works satisfactorily for test images, the pipeline is employed for lane detection in a video. The pipeline consisted of 5 major steps excluding reading and writing the image.

   *The test image is first converted to grayscale from RGB using the helper function grayscale().
   [image1]: ./examples/grayscale.jpg "Grayscale"

   *The grayscaled image is given a gaussian blur to remove noise or spurious gradients.

   *Canny edge detection is applied on this blurred image and a binary image is produced.

   *This image contains edges that are not relevant for lane finding problem. A region of interest is defined to separate the lanes from sorrounding environment and a masked image containing only the lanes is extracted using cv2.bitwise_and() function from opencv library.

   *This binary image of identified lane lines is finally merged with the original image using cv2.addweighted() function from opencv library. This produces an image given below. Note that, this is without making any modifications to the drawlines() helper function. It can be observed that the lines are not continuous as required.

## Modification of drawlines() function
Since the resulting line segments after the processing the image through the pipeline are not continuous, a modification is made to the drawlines() helper function. Consider the code snippet below:
``` 
def draw_lines_robust(img, lines, color=[200, 0, 0], thickness = 10):
  
    x_left = []
    y_left = []
    x_right = []
    y_right = []
    imshape = image.shape
    ysize = imshape[0]
    ytop = int(0.6*ysize) # need y coordinates of the top and bottom of left and right lane
    ybtm = int(ysize) #  to calculate x values once a line is found
    
    for line in lines:
        for x1,y1,x2,y2 in line:
            slope = float(((y2-y1)/(x2-x1)))
            if (slope > 0.5): # if the line slope is greater than tan(26.52 deg), it is the left line
                    x_left.append(x1)
                    x_left.append(x2)
                    y_left.append(y1)
                    y_left.append(y2)
            if (slope < -0.5): # if the line slope is less than tan(153.48 deg), it is the right line
                    x_right.append(x1)
                    x_right.append(x2)
                    y_right.append(y1)
                    y_right.append(y2)
    # only execute if there are points found that meet criteria, this eliminates borderline cases i.e. rogue frames
    if (x_left!=[]) & (x_right!=[]) & (y_left!=[]) & (y_right!=[]): 
        left_line_coeffs = np.polyfit(x_left, y_left, 1)
        left_xtop = int((ytop - left_line_coeffs[1])/left_line_coeffs[0])
        left_xbtm = int((ybtm - left_line_coeffs[1])/left_line_coeffs[0])
        right_line_coeffs = np.polyfit(x_right, y_right, 1)
        right_xtop = int((ytop - right_line_coeffs[1])/right_line_coeffs[0])
        right_xbtm = int((ybtm - right_line_coeffs[1])/right_line_coeffs[0])
        cv2.line(img, (left_xtop, ytop), (left_xbtm, ybtm), color, thickness)
        cv2.line(img, (right_xtop, ytop), (right_xbtm, ybtm), color, thickness)



```
Observe that a classification of lines identified through houghlines criteria is made based on their slope. Evidently, lines with positive slope are classified as being on the left lane and lines with negative slope are classified as being on the right lane. Flat lines having slope below absolute value of 0.5 are discarded. After storing points for respective left and right lanes, a linear curve fit (degree 1) using polyfit() function from numpy library is done to obtain the slope and intercept of left and right lanes. Following this, x coordinates are found for respective y top and btm coordinates (user defined) using the lane equations for both lanes. This gives us starting and ending coordinates for both left and right lane. Finally, lines are drawn using cv2.line() function to connect these points and the image is merged with the original image as before.


## Implementation of the pipeline in the test video

[![Watch the video](https://classroom.udacity.com/nanodegrees/nd013/parts/168c60f1-cc92-450a-a91b-e427c326e6a7/modules/5d1efbaa-27d0-4ad5-a67a-48729ccebd9c/lessons/7c075239-1f65-4952-bde8-1810354d7988/concepts/ccb577c4-2242-4739-b9c1-eb4c52291391)](https://youtu.be/evr0__y6cpU)




##  Identify potential shortcomings with your current pipeline


Since the first step is converting the image to grayscale from RGB, shadows and light variations in the environment are difficult to capture. This can be gleaned from the fact that the current pipeline while working reasonably well for the first two test videos breaks down for the challenge video.


### 3. Suggest possible improvements to your pipeline

1) The way of dealing with shadows and variations in light is to use a different colorspace, for example, say HSV instead of RGB. A function definition for applying RGB to HSV transform is shown below. The test image before being fed to the pipeline is given an RGB to HSV transform. This approach while producing acceptable results for the first two test videos did not result in any significant improvements in the performance of identifying lane lines for challenge video.

2) Further, we can use machine learning techniques for a better results.