# Real-Time-Vision-Pipeline-for-Autonomous-Navigation-on-Raspberry-Pi-using-Classical-Image-Processing

Designed and implemented a complete vision pipeline on a Raspberry Pi with PiCamera to perceive drivable surfaces, detect obstacles, and generate simple navigation commands. Streaming both raw and annotated frames over HTTP to a remote client. The system runs at about 5 FPS, demonstrating efficient hardware–software co-design for resource-constrained embedded platforms.

- Road Segmentation:
1. Capture each RGB frame and convert it to HSV.
2. Apply hue-range thresholding in two passes: red/pink thresholds to mask the main roadway, and yellow thresholds to highlight lane markings.
3. Clean the red/pink mask via median filtering and erosion with a custom 5×9 trapezoidal structuring element.
4. Use connected-component analysis to isolate the largest contiguous region, producing a binary “road” mask.

- Edge-Based Obstacle Detection:
1. Convert the original frame to grayscale and dilate it using an elliptical kernel; subtract the original to emphasize edges.
2. Threshold the difference image to produce a binary edge map, then apply morphological opening, closing, and dilation to remove noise and fill gaps.
3. Find contours in the cleaned edge map within the road mask, and for each contour compute area, solidity (area/hull_area), and extent (area/bounding_box_area).
4. Reject candidates below minimum area, solidity, or extent thresholds, and draw bounding boxes around confirmed obstacles.

- Navigation Decision Logic:
1. If obstacles are detected, compare the bottom-edge positions of their bounding boxes against “stop” and “slow” thresholds to decide whether to halt, decelerate, or steer.
2. For slow-down scenarios, partition the road mask into left and right avoidance regions based on obstacle extents, compute available free-space areas, and choose the direction with more clearance.
3. If no obstacles are present, compute the centroid of the road mask via image moments and compare left/right area under a tolerance threshold to determine curvature and issue “turn” or “go” commands.
4. Overlay decision text (“Stop,” “Slow Down,” “Turn Left/Right,” or “Go”) on the live frame for visual feedback.

- System Integration & Streaming:
1. Leverage the Picamera2 API for low-latency frame capture at 640×480 resolution.
2. Establish a TCP socket server on the Pi that serializes (pickle) and transmits both the original and road-mask frames to a connected laptop.
3. Maintain a consistent 5 FPS loop by spacing captures and processing steps using a simple timing guard.

This end-to-end pipeline highlights hands-on expertise in OpenCV-based image segmentation, morphology, contour analysis, and embedded network streaming, delivering a transparent, lightweight autonomy solution suitable for robotics, driver-assistance demos, and rapid prototyping in constrained environments.
