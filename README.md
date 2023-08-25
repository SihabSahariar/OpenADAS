# Open-ADAS
OPEN ADAS Lane detection system v1. Implements OpenCV. non learning platforms. Average accuracy and latency.<br>
# Prerequisites<br>
*Python 3.7 or higher with OpenCV installed<br>
*numpy and matplotlib<br>
# Python Code for Detection of Lane Lines in an Image<br>
Before, we get started, I’ll share with you the full code you need to perform lane detection in an image. The two programs below are all you need to detect lane lines in an image.
You need to make sure that you save both programs below, edge_detection.py and lane.py in the same directory as the image.
edge_detection.py will be a collection of methods that helps isolate lane line edges and lane lines. 
lane.py is where we will implement a Lane class that represents a lane on a road or highway.<br>
Now that you have all the code to detect lane lines in an image, let’s explain what each piece of the code does<br>
# Isolate Pixels That Could Represent Lane Lines
The first part of the lane detection process is to apply thresholding (I’ll explain what this term means in a second) to each video frame so that we can eliminate things that make it difficult to detect lane lines. By applying thresholding, we can isolate the pixels that represent lane lines.
Glare from the sun, shadows, car headlights, and road surface changes can all make it difficult to find lanes in a video frame or image.
What does thresholding mean? Basic thresholding involves replacing each pixel in a video frame with a black pixel if the intensity of that pixel is less than some constant, or a white pixel if the intensity of that pixel is greater than some constant. The end result is a binary (black and white) image of the road. A binary image is one in which each pixel is either 1 (white) or 0 (black).<br> 
# Thresholding Steps <br>
1. Convert the video frame from BGR (blue, green, red) color space to HLS (hue, saturation, lightness).

There are a lot of ways to represent colors in an image. If you’ve ever used a program like Microsoft Paint or Adobe Photoshop, you know that one way to represent a color is by using the RGB color space (in OpenCV it is BGR instead of RGB), where every color is a mixture of three colors, red, green, and blue. You can play around with the RGB color space here at this website.

The HLS color space is better than the BGR color space for detecting image issues due to lighting, such as shadows, glare from the sun, headlights, etc. We want to eliminate all these things to make it easier to detect lane lines. For this reason, we use the HLS color space, which divides all colors into hue, saturation, and lightness values.

If you want to play around with the HLS color space, there are a lot of HLS color picker websites to choose from if you do a Google search.<br>

2. Perform Sobel edge detection on the L (lightness) channel of the image to detect sharp discontinuities in the pixel intensities along the x and y axis of the video frame. 

Sharp changes in intensity from one pixel to a neighboring pixel means that an edge is likely present. We want to detect the strongest edges in the image so that we can isolate potential lane line edges.<br>

3. Perform binary thresholding on the S (saturation) channel of the video frame. Doing this helps to eliminate dull road colors. 

A high saturation value means the hue color is pure. We expect lane lines to be nice, pure colors, such as solid white and solid yellow. Both solid white and solid yellow, have high saturation channel values. 

Binary thresholding generates an image that is full of 0s (black) and 255 (white) intensity values. Pixels with high saturation values (e.g. > 80 on a scale from 0 to 255) will be set to white, while everything else will be set to black.

Feel free to play around with that threshold value. I set it to 80, but you can set it to another number, and see if you get better results.<br>

4. Perform binary thresholding on the R (red) channel of the original BGR video frame. 

This step helps extract the yellow and white color values, which are the typical colors of lane lines. 

Remember, pure white is bgr(255, 255, 255). Pure yellow is bgr(0, 255, 255). Both have high red channel values.

To generate our binary image at this stage, pixels that have rich red channel values (e.g. > 120 on a scale from 0 to 255) will be set to white. All other pixels will be set to black.<br>

5. Perform the bitwise AND operation to reduce noise in the image caused by shadows and variations in the road color.

Lane lines should be pure in color and have high red channel values. The bitwise AND operation reduces noise and blacks-out any pixels that don’t appear to be nice, pure, solid colors (like white or yellow lane lines.) <br>

The get_line_markings(self, frame=None) method in lane.py performs all the steps I have mentioned above.

If you uncomment this line below, you will see the output:<br>
cv2.imshow("Image", lane_line_markings) <br>
To see the output, you run this command from within the directory with your test image and the lane.py and edge_detection.py program.<br>
python lane.py <br>
For best results, play around with this line on the lane.py program. Move the 80 value up or down, and see what results you get.<br>
_, s_binary = edge.threshold(s_channel, (80, 255)) <br>
Now that we know how to isolate lane lines in an image, let’s continue on to the next step of the lane detection process.<br>
# Apply Perspective Transformation to Get a Bird’s Eye View <br>
We now know how to isolate lane lines in an image, but we still have some problems. Remember that one of the goals of this project was to calculate the radius of curvature of the road lane. Calculating the radius of curvature will enable us to know which direction the road is turning. But we can’t do this yet at this stage due to the perspective of the camera. Let me explain.<br>
# Why We Need to Do Perspective Transformation <br>
Imagine you’re a bird. You’re flying high above the road lanes below.  From a birds-eye view, the lines on either side of the lane look like they are parallel.

However, from the perspective of the camera mounted on a car below, the lane lines make a trapezoid-like shape. We can’t properly calculate the radius of curvature of the lane because, from the camera’s perspective, the lane width appears to decrease the farther away you get from the car. 

In fact, way out on the horizon, the lane lines appear to converge to a point (known in computer vision jargon as vanishing point). <br>
The camera’s perspective is therefore not an accurate representation of what is going on in the real world. We need to fix this so that we can calculate the curvature of the land and the road (which will later help us when we want to steer the car appropriately).<br>
# How Perspective Transformation Works <br> 
Fortunately, OpenCV has methods that help us perform perspective transformation (i.e. projective transformation or projective geometry). These methods warp the camera’s perspective into a birds-eye view (i.e. aerial view) perspective.

For the first step of perspective transformation, we need to identify a region of interest (ROI). This step helps remove parts of the image we’re not interested in. We are only interested in the lane segment that is immediately in front of the car.

You can run lane.py from the previous section. With the image displayed, hover your cursor over the image and find the four key corners of the trapezoid. 

Write these corners down. These will be the roi_points (roi = region of interest) for the lane. In the code (which I’ll show below), these points appear in the __init__ constructor of the Lane class. They are stored in the self.roi_points variable.

In the following line of code in lane.py, change the parameter value from False to True so that the region of interest image will appear.<br>
Now that we have the region of interest, we use OpenCV’s getPerspectiveTransform and warpPerspective methods to transform the trapezoid-like perspective into a rectangle-like perspective. 

Change the parameter value in this line of code in lane.py from False to True.<br>
warped_frame = lane_obj.perspective_transform(plot=True)<br>
Here is an example of an image after this process. You can see how the perspective is now from a birds-eye view. The ROI lines are now parallel to the sides of the image, making it easier to calculate the curvature of the road and the lane.<br> 
# Identify Lane Line Pixels <br>
We now need to identify the pixels on the warped image that make up lane lines. Looking at the warped image, we can see that white pixels represent pieces of the lane lines.

We start lane line pixel detection by generating a histogram to locate areas of the image that have high concentrations of white pixels. 

Ideally, when we draw the histogram, we will have two peaks. There will be a left peak and a right peak, corresponding to the left lane line and the right lane line, respectively.

In lane.py, make sure to change the parameter value in this line of code (inside the main() method) from False to True so that the histogram will display.<br>
histogram = lane_obj.calculate_histogram(plot=True) <br>
# Set Sliding Windows for White Pixel Detection <br> 
The next step is to use a sliding window technique where we start at the bottom of the image and scan all the way to the top of the image. Each time we search within a sliding window, we add potential lane line pixels to a list. If we have enough lane line pixels in a window, the mean position of these pixels becomes the center of the next sliding window.

Once we have identified the pixels that correspond to the left and right lane lines, we draw a polynomial best-fit line through the pixels. This line represents our best estimate of the lane lines.
In this line of code, change the value from False to True.<br>
left_fit, right_fit = lane_obj.get_lane_line_indices_sliding_windows(
     plot=True)<br>
# Fill in the Lane Line <br> 
Now let’s fill in the lane line. Change the parameter value on this line from False to True. <br> 
lane_obj.get_lane_line_previous_window(left_fit, right_fit, plot=True) <br>
# Overlay Lane Lines on Original Image <br>
Now that we’ve identified the lane lines, we need to overlay that information on the original image.

Change the parameter on this line form False to True and run lane.py.<br>
frame_with_lane_lines = lane_obj.overlay_lane_lines(plot=True)<br>
# Calculate Lane Line Curvature <br>
Now, we need to calculate the curvature of the lane line. Change the parameter value on this line from False to True.<br>
lane_obj.calculate_curvature(print_to_terminal=True)<br>
# Calculate the Center Offset <br> 
Now we need to calculate how far the center of the car is from the middle of the lane (i.e. the “center offset).On the following line, change the parameter value from False to True.<br>
lane_obj.calculate_car_position(print_to_terminal=True)<br>
# Display Final Image <br>
Now we will display the final image with the curvature and offset annotations as well as the highlighted lane.In lane.py, change this line of code from False to True:<br>
frame_with_lane_lines2 = lane_obj.display_curvature_offset(
     frame=frame_with_lane_lines, plot=True)<br> 
# Detect Lane Lines in a Video <br>
Now that we know how to detect lane lines in an image, let’s see how to detect lane lines in a video stream.

All we need to do is make some minor changes to the main method in lane.py to accommodate video frames as opposed to images.<br>
# Troubleshooting
For best results, play around with this line on the lane.py program. Move the 80 value up or down, and see what results you get.<br>
_, s_binary = edge.threshold(s_channel, (80, 255)) <br>
If you run the code on different videos, you may see a warning that says “RankWarning: Polyfit may be poorly conditioned”. If you see this warning, try playing around with the dimensions of the region of interest as well as the thresholds.

You can also play with the length of the moving averages. I used a 10-frame moving average, but you can try another value like 5 or 25:<br>
if len(prev_left_fit2) > 10: <br>
Using an exponential moving average instead of a simple moving average might yield better results as well.

That’s it for lane line detection. 



















