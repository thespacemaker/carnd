#**Finding Lane Lines on the Road**

##Writeup Template

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

I approached this project under the idea that piggy-backing on another open-source projects would be better than trying to build a pipeline from scratch. After looking through a couple of github repos I settled on the one created by Kidra521

I’ll outline his pipeline below and describe some of the changes I decided to implement in my fork in order to improve on his work.

###Pipeline Steps
####The pipeline consists of 4 steps
1. Initialize variables and define classes used for deciding on candidate line segments. These lines are used to create both the left and right lane. There is also a decisions matrix declared as a means of identifying the stability of the lane. Interesting things to consider about line and lane classes
	2. Lines are interpolated using a linear regression. There is the ability in the code to use polynomial expressions (I imagine this could be useful in the extra hard challenge video in order to better extrapolate the curves.)
	3. The Vanishing point of both lane lines is solved with the function update_vanishing_point. This is later used to consider candidates of lines
	4. Lines are a separate class and will be compiled into a lane line after found to be a good candidate using the Property “Candidate”. This property narrows down lines based on a few assumptions outlined in the comment.
	~A simple domain logic to check whether this hough line can be a candidate for being a segment of a lane line.  
	~	1. The line cannot be horizontal and should have a reasonable slope.
		~2. The difference between lane line's slope and this hough line's cannot be too high.
		~3. The hough line should not be far from the lane line it belongs to.
		~4. The hough line should be below the vanishing point.
	5. After proper candidates are chosen they are assigned to either lane with the assign_to_lane_line_fuction
	6. A good module to also note is the stable/unstable rating assigned via “update_current_line_coeff. _ This function checks the lane each frame against a buffer (set as a variable Buffer Frames) and if the new frame exhibits a drastic change in slope compared to the buffer the lane state is changed to unstable. This has proven a vital function in checking the accuracy of the image pipeline.

2. The next block outlines the image preprocessing functions
	3. GIMP is used to convert to HSV. This was a major part of choosing this pipeline. By using HSV instead of greyscale, the ability to pull out yellow lane lines is greatly amplified. These thresholds are defined as
		 '' WHITE_LINES = { 'low_th': gimp_to_opencv_hsv(0, 0, 80),
		''                 'high_th': gimp_to_opencv_hsv(359, 10, 100) }
		''
		'' YELLOW_LINES = { 'low_th': gimp_to_opencv_hsv(35, 20, 30),
		''                  'high_th': gimp_to_opencv_hsv(65, 100, 100),
		''                  'kernel': np.ones((3,3),np.uint64)}

	4. Hough transform is described in the code as
'' A modified implementation of a suggested `hough_lines` function which allows Line objects initialization and in-place line filtering.Returns a list of Line instances which are considered segments of a lane.
	5. The individual lines from the Hugh Transform are then compiled into a single lane and updated every frame
6. The function imagePipeline() is the three phase function where all of the magic happens.
	7. The first phase in the function calls all the preprocessing functions on the image. HSV, binary, mask, zeroing the pixels and applying canny functions are all applied in this phase
	8. The lane lines are then updated with updateLane()
	9. The final stage is drawing.
		10. The first image Snapshot 1. This is the top left box in the final output and displays a simple masked binary image with the filtered lines drawn on top (filtered lines are the output of hough transform)
		11. The second image is the small window to the right of the first and shows Canny edges with the Lane lines drawn in right and left colors (variables set at the beginning of the notebook). **I have added some text using CV.2puttext() in order to track the amount of times the lane line becomes unstable, This allows me to track how changing variables such as the length of buffer, white and yellow thresholds, and others will effect the final outcome. Before adding this functionality it was impossible to actually quantify performance.**
		12. The final image is the large image. This is where the final lane lines are drawn with all the functions acting in the imagepipeline.
	13. The final set of functions outline the drawing process as well as establish the canny, gaussian, and hough transform variables. This was interesting to me as, for the majority of the course so far, these were called much earlier in the program. But…. this works, so i’m not touching it :P The final function, weighted_img_ is compiles the output of hough_lines_ with the original image.


###2. Identify potential shortcomings with your current pipeline


This pipeline really seems to work well. I’m glad to find the extra challenge video because it really outlines some of the areas where even a clever solution like this is sure to fail. In the curvy mountain roads there are very large shifts in light. This can be seen when the vehicle passes between tree coverage and the sun bleaches the road. This immediately causes the pipeline to fail. This can possibly be remedied by adjusting the white thresholds but honestly I cannot see it being improved too much without jeopardizing the pipeline in other light conditions. This is a tough problem and I do not think that visual imagining used alone is the best way to tackle it.

Other shortcomings, though not as dire, are worth noting
1. By using the vanishing point and calculating slope we end up with a linear line that does not curve with the road. Some may argue that this doesn’t matter as the intended slope is more important than the curve but I would disagree in cases where the radius of the road is not constant. Using a non-linear function would most certainly perform better for predicative modeling and insuring the vehicles position in the lane
2.  The hood lines have not been accounted for in the polygon mask. This is something I could fix but I imagine it will be addressable in further lessons. I will focus on continuing.
3. I imagine this is an entirely different beast at night. Seeing how it handles road lines in the dark is something that needs to be tested
4. If it snows…. good luck




###3. Suggest possible improvements to your pipeline

1. Using a non-linear function would definitely help give a more accurate representation of the lane lines
2. Change the polygon mask to cut out the hood lines.
3. I really like the birds-eye view approach I have found while researching online. Using the moving boxes and birds-eye would significantly help in tracking the curves.
4. I cannot help but feel like some type of filter on the camera could prevent drastic changes in light to affect the pipeline as it does in the extra challenge.
5. Another way to approach the problem in number 4 could be to apply some contrast preprocessing to try and even out some of the precipitous changes in light. How this would affect the white thresholds however would also have to be tested
6. I think the vanishing point solution is really only a temporary solution. It fails to track the curve of the road and certainly does not illustrate where the final vanishing point down the road is. Non-linearity will have to be implemented to solve this
7. Adding test images taken at night time would definitely be a way to test more conditions, same applies to weather
8. I think tracking the horizon is an interesting though. Especially on hilly roads, the horizon would have to dynamically update in order to update the vanishing point appropriately as well as compensate for any distortion in lane lines.
9. implemented some time of metrics for tracking distances would be useful for more elaborate computations. I believe this is something that we will address in later lessons.
10. There are many more but I’ll close with this one. There is a risk, in scenarios with curbs and other delineating structures where the image pipeline can confuse the edge of the road or barrier as the lane line. This is very obvious with the extra hard challenge where the grass edge is repeatedly mistaken for the lane line. I think adding sensors and the incorporation of lira is really important in tackling this challenge. Understanding where the road edges are in a spatial manner will play a key role in narrowing down what is a lane line and what is simply an edge that follows along in a parallel path.

###Thank you so much for taking the time to read this. I have absolutely enjoyed this course so far and cannot thank you enough for helping me through this exciting endeavor! Cheers!

Adnan E. Chahbandar
