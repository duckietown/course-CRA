# Exercises - state estimation and sensor fusion {#exercise-sensor_fusion status=ready}

Excerpt: Build your own localization system through sensor fusion and the Apriltag library

In this exercise you will learn how to create estimators with both proprioceptive and exteroceptive sensors and how to manipulate the TF tree so that you can fuse these estimates.

<div class='requirements' markdown='1'>
  Requires: [Odometry with Wheel Encoders](+duckietown-robotics-development#odometry-modeling)

  Requires: [Modeling the Duckiebot and Representations](+duckietown-robotics-development#representations-modeling)

  Requires: [Advanced Augmented Reality Exercise](#cra-apriltag-augmented-reality-exercise)

  Results: Understand how to build a TF tree in ROS and visualize it in RViz.

  Results: Be able to create a multi-package Dockerized ROS workspace, and deploy it on the Duckiebot.

  Results: Experience with working with ROS Timers and Services

  Results: Create a wheel-encoder based estimator. Understand the benefits and drawbacks of such an estimator.

  Results: Create an Apriltag-based estimator. Understand the benefits and drawbacks of this estimator.

  Results: Create a sensor fusion node that fuses estimates from the above individual estimators.
</div>


The overall goal of this exercise is to create a state estimation system that localizes the Duckiebot in global coordinates (without relying on a lane, so in any flat environment). The code developed for this exercise will be used in the coming exercise for a parallel parking task! Therefore, throughout this exercise, keep in mind that better performance of your localization code will make your control task later much easier. 

To be able to perform such a complex task, we will have to use all sensors onboard the robot, namely the camera and the wheel encoders. We will achieve our localization goal in incremental steps: first, we will develop an estimator using only wheel encoders, then, we will develop an estimator using only camera information and the Apriltag detection library, and finally, we will fuse these estimates so that we hopefully get the best of both worlds.

#### Encoder localization package {#exercise:encoder_localization}

The first package you'll have to create in your Dockerized ROS workspace is one that will contain a node that publishes a `TransformStamped` message at 30Hz with the following properties: `frame_id: map`, `child_frame_id: encoder_baselink`, `stamp`: timestamp of the last wheel encoder tick, `transform`: a 2D pose of the robot baselink (see FIGURE for a definition of this frame). The node will also have to broadcast the TF `map`-`encoder_baselink`.

Deliverable:

* A screen recording similar to this EXAMPLE where you drive the robot in a loop and try to "close the loop" (by driving to the point where you started from). Make sure the images are recorded as well as the TF tree in Rviz. Use a visible landmark as the origin of the map frame.

* A link to a github repository containing a package called `encoder_localization`

Hints:

* For the estimates to be in a global frame, you will have to provide an initial state.

* Use a `tf.TransformBroadcaster()` to broadcast a TF which can be visualized in RViz. Note that you can pass the exact same message as the one that your Publisher uses.

* For the kinematic model of the Duckiebot, you'll have to load the following parameters from the calibration file: `baseline`, `radius`.

* To open RViz, simply run `dts start_gui_tools hostname.local` and run `rviz` from inside the container. Note that if you keep the container running you can save your RViz configurtion file so that when you reopen it, it automatically displays your topics of interest.

* Rviz uses the following color-code convention for frame axes: `red: x-axis`, `green: y-axis`, `blue: z-axis`.

* To publish messages at a fixed frequency, consider using a ROS Timer. It works very similar to a subscriber except for the fact that the callback get called after a fixed duration of time.

* [This link](http://docs.ros.org/en/jade/api/tf/html/python/transformations.html) contains many useful tf library methods which allow you to switch between representations (e.g. expressing an euler yaw angle as a quaternion)

<end/>

<!-- TODO: change video link -->
<figure id="encoder-exercise-example">
    <figcaption>Example video deliverable for [](#exercise:encoder_localization) </figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure>

<!-- TODO: add picture of baselink on Duckiebot (waiting for Aleks) -->
<div figure-id="fig:mod-kin" figure-caption="Position of the baselink and camera frames on the robot">
  <img src=".png" style='width: 30em; height:auto'/>
</div>

As you have probably realized, some of the advantages of this localization system is that as long as the robot is moving, the wheel encoders will provide information about the state of the robot, at a high rate and with little delay. However, the pose of the robot is an integration of the measurements of the robot, meaning that it is subject to drift as any inaccuracies in measurement get propagated through time. When driving aggressively or through a slippery surface these inaccuracies are amplified (try this for yourself!). 

A common way of getting rid of drift in mobile robots is to either use absolute position measurements (with GPS, for instance) or to use fixed landmarks in the world as reference. This is how we humans also navigate the environment. Since there is no GPS on-board the Duckiebot, we will have to use the latter approach. This is what we will explore in the next exercise, where our landmarks will be traffic signs with Apriltags (see FIGURE).

<!-- TODO: add picture of traffic sign with height arrow -->
<div figure-id="fig:mod-kin" figure-caption="Example traffic sign to be used for Apriltag localization. In order to accurately construct your TF tree, please measure the height indicated">
  <img src=".png" style='width: 30em; height:auto'/>
</div>


#### Apriltag localization package {#exercise:apriltag_localization}


In this package you'll have to place a node that subscribes to `/camera_node/image/compressed` and publishes a `TransformStamped` message (if an image with a detected AApriltag has been received) with the following properties: `frame_id: map`, `child_frame_id: at_baselink`, `stamp`: timestamp of the last image received, `transform`: a 2D pose of the robot baselink (see FIGURE for a definition of this frame). This node will also have to broadcast the following TFs: `map`-`apriltag`, `apriltag`-`camera`, `camera`-`at_baselink`. Make sure that when you place the apriltag in front of the robot you get something that looks roughly like FIGURE.. This will make it easier to fuse this pose with the pose from the encoders.

Deliverable:

* A screen recording similar to this EXAMPLE where you move the Apriltag in front of the robot from one side of the Field-of-View (FOV) to the other. Make sure the images are recorded as well as the TF tree in Rviz. 

* Instead of directly using compressed images, rectify them before passing them to the Apriltag detector. You will see that this significantly improves the accuracy of the detector but at the cost of significant delay. Provide a screen recording similar to this EXAMPLE where you move the Apriltag in front of the robot from one side of the Field-of-View (FOV) to the other. Make sure the images are recorded as well as the TF tree in Rviz. 

* (Bonus) Offload the computation of the rectified image to the GPU of the Jetson Nano so that the improved accuracy can be obtained without significant delay.

* A link to a github repository containing a package called `at_localization`

Hints:

* A frame cannot have two parents. If you try to broadcast the following TFs: `map`-`baselink` and `camera`-`baselink`, you will see that you'll only be able to see the individual frames in RViz.

* To rectify images, use the following `cv2` methods: `cv2.getOptimalNewCameraMatrix(cameraMatrix, distCoeffs, (640, 480), 1.0)`, `cv2.initUndistortRectifyMap(cameraMatrix, distCoeffs, np.eye(3), newCameraMatrix, (640, 480), cv2.CV_32FC1)` and `cv2.remap(compressed_image, map1, map2, cv2.INTER_LINEAR)`.

* To use the Apriltag detector, consult [this link](https://github.com/duckietown/lib-dt-apriltags). You can extract the pose of the Apriltag in the camera frame with `tag.pose_R` (rotation matrix) and `tag.pose_t` (translation vector). Keep in mind that the coordinate frame convention (see FIGURE) is different than the one you are supposed to use in the deliverable!

* To convert between frames, we recommend that you use 4x4 transformation matrices. An example of such a matrix is $$ T_{AB} = \begin{bmatrix}
R_{AB} & {}_{A}r_{AB}\\
\vec{0} & 1
\end{bmatrix} $$ which transforms a vector in frame B to frame A. During your manipulations, keep in mind that $$ T_{BA} = (T_{AB})^{-1} $$ and $$ T_{AB} = T_{AC} T_{CB} $$

</end>

<!-- TODO: add picture of traffic sign with height arrow -->
<div figure-id="fig:mod-kin" figure-caption="Example traffic sign to be used for Apriltag localization. In order to accurately construct your TF tree, please measure the height indicated">
  <img src=".png" style='width: 30em; height:auto'/>
</div>

<!-- TODO: change video link -->
<figure id="at-exercise-example-compressed">
    <figcaption>Example video deliverable for [](#exercise:at_localization) with compressed images </figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure>

<!-- TODO: change video link -->
<figure id="at-exercise-example-rectified">
    <figcaption>Example video deliverable for [](#exercise:at_localization) with rectified images </figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure>


The Apriltag detected provides accurate pose estimates with respect to landmarks but at the cost of delay and low frequency. Moreover, one cannot continuously rely on such estimates since the Apriltag could go out of sight. To combat this, we can fuse the estimates of the Apriltag and the wheel encoders for continuous, accurate and robust localization. This is the goal of the next exercise.

#### Fused localization package {#exercise:fused_localization}

In this package you'll have to place a node that publishes a `TransformStamped` message with the following properties: `frame_id: map`, `child_frame_id: fused_baselink`, `stamp`: current time, `transform`: a 2D pose of the robot baselink (see FIGURE for a definition of this frame).  The node will also have to broadcast the TF `map`-`fused_baselink`. As a minimum, your fusion node should have the following behaviour:

* The first time the Apriltag becomes visible, you have to calibrate the `encoder_baselink` frame/estimate to match exactly the Apriltag pose. This should be done with a ROS Service between  `fused_localization_node` and `encoder_localization_node`.

* Publish the robot pose using the Apriltag estimate (when available).

* If the Apriltag is not visible, use the encoder estimates, starting from the last Apriltag pose received (there should be no pose jump if the Apriltag goes out of sight).

* If the Apriltag becomes visible again, switch back to using Apriltag estimates (a jump in fused pose is allowed)

* You don't have to handle the delay or the variance of the individual estimates to complete the exercise, but you are more than welcome to! Remember that the more accurate you make this node, the better your parking controller will perform in the coming exercise!

Deliverable:

* A screen recording similar to this EXAMPLE,  where you drive the robot aggressively in a loop around the Apriltag with a joystick, and one similar to EXAMPLE2, where you slowly drag the robot manually around the Apriltag making sure that the wheels are always turning forward and that you have good grip on the ground. In both cases you should end the trajectory where you started it (feel free to use a marker on the ground). Make sure the images are recorded as well as the TF tree in Rviz. 

* (Bonus) A screen recording where you use rectified images and still drive fast. This will require you to handle the delay of the Apriltag estimates in some way.

* A link to a github repository containing a package called `fused_localization`

</end>

<!-- TODO: change video link -->
<figure id="fused-exercise-example-compressed-agressive">
    <figcaption>Example video deliverable for [](#exercise:fused_localization) with compressed images and aggressive driving</figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure>

<!-- TODO: change video link -->
<figure id="fused-exercise-example-rectified-slow">
    <figcaption>Example video deliverable for [](#exercise:fused_localization) with rectified images and slow driving</figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure>