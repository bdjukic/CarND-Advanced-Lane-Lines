# Finding road lanes using computer vision

One of the last projects in [Udacity's self-driving car course](http://udacity.com/drive) is detecting road lanes using only the camera mounted in the front of the car. The outcome of the project is a video pipeline which is capable of drawing boundaries of the car's lane and calculate the radius curve of it.

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/1.jpg)

This is somewhat similar to the very first project from the course, but the provided video from the car this time around is much more complicated than the previous one (contains curves, shadows, different lane colors, etc.).

These are the steps for the pipeline which will be applied on the test video:

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/2.jpg)

I will go now into more details for each one of them.

### Camera calibration

This one came as a surprise for me. I was aware that some cameras are distorting the final images (think about GoPro and other action cameras) but definitely was not aware that all cameras do that.

The calibration is necessary due to the transformation camera is doing from the 3D world into 2D world. This transformation is not perfect since some objects in the image might appear to have wrong shape and size. We need to address this since we are relying only on camera input in this case in order to properly locate our car in the surroundings.

In this particular step, the first thing we need is a camera matrix which represents the tranformation matrix from the 3D object (_P(X, Y, Z)_) into 2D object (_p(x, y)_).

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/3.jpg)

Secondly, we need to calculate the distortion coefficient from the camera. This coefficient is affected by the amount of [radial and tangential distortion](https://en.wikipedia.org/wiki/Distortion_%28optics%29)of the image.

We will be using chess board image and OpenCV library for calibrating our camera. This is a standard way of camera calibration and OpenCV is providing [specific APIs](http://docs.opencv.org/2.4/doc/tutorials/calib3d/camera_calibration/camera_calibration.html) for this exercise.

### Create undistorted image

Once we have the camera matrix and distortion coefficient, we can apply them on our test images which are screen grabs from the car's camera.

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/4.jpg)

In the picture below it's hard to notice the difference. Maybe the most obvious one would be the front of the car. These changes are indeed subtle but crucial to get it right.

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/5.jpg)

### Warp the image in the area of interest

After getting our undistorted image we need to mark the area of interest on the undistorted image which is in our case trapezoid covering the lane curvature. The next step would be doing a perspective transform of the marked area into the new coordinates space (bird's eye view) which will make things easier for us in order to calculate the lane curvature and eventually mark them in video frames.

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/6.jpg)

### Apply gradient and color threshold

We need to clearly mark the lanes in our warped image. The initial step we will take is to apply Sobel operator on the image in order to detect steep edge changes. We know upfront that the lanes on the image are vertical, therefore applying Sobel operator on X coordinate emphasizes edges which have vertical direction. After applying Sobel operator we will create binary threshold based on gradient strength. After multiple tries, I've found that threshold between 20 and 150 gives me the best results.

In order to improve the lane detection I've applied also color thresholds based on R channel (210, 255) from RGB spectrum, S (190, 255) and H (40, 100) channels from HLS color spectrum and L (210, 255) from LUV spectrum.

These are the individual results from different thresholds from above:

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/7.jpg)

In the end, by combining these different thresholds we're getting the final output:

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/8.jpg)

### Lane detection and marking

The final step in this pipeline is detecting the vertical lanes and marking them in the thresholded and warped image.

In order to detect the pixel grouping in the lane we will be looking at the histogram of the output we have above. Here's the histogram for the bottom half of the image:

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/9.jpg)

We will be using the concept of sliding windows and moving this histogram from one end of the image to the other and detecting the areas of it where the pixels are concentrated. Once we identify these areas, we will be fitting a polynomial (**f(y) = Ay² + By + C**) which would represent our lanes.

**A**represents curvature of the lane line, **B** is the heading or direction that the line is pointing, and **C** gives you the position of the line based on how far away it is from the very left of an image.

![alt tag](https://raw.githubusercontent.com/bdjukic/CarND-Advanced-Lane-Lines/master/md_images/10.jpg)

Once we identify the polynomial we can draw it on the original frame with applied perspective transformation which we have already calculated in previous steps.

I've also introduced sanity checks for each frame on the widths of top and bottom coordinates from the left and right polynomials. This is fixing an issue with an occasional and unexpected jumps of the curvatures drawn on the video frames.

### Final output

After applying the pipeline with the above steps on all the frames from the test video we will get this as a result:

[![IMAGE ALT TEXT HERE](https://img.youtube.com/vi/GWFhU_jCPvo/0.jpg)](https://www.youtube.com/watch?v=GWFhU_jCPvo)

### Conclusion

You can find the code for this project on my [Github](https://github.com/bdjukic/CarND-Advanced-Lane-Lines) profile. This was quite a challenging project for me, especially getting the right values for the Sobel operator and color thresholds since this step affects the final output quite a bit. The current pipeline would probably have issues with the road which contains uphill and/or downhill. Also, I would assume that direct sun reflection would causes issues with detecting road lanes.
