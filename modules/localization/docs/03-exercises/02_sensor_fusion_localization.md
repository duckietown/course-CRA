# Exercises - state estimation and sensor fusion {#exercise-sensor_fusion status=ready}

Excerpt: Build your own localization system through sensor fusion and the Apriltag library

In this exercise you will learn how to create estimators with both proprioceptive and exteroceptive sensors and how to manipulate the frame transformations tree to easily fuse these estimates.

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


The overall goal of this exercise is to create a state estimation system that localizes the Duckiebot in global coordinates. So far we have only seen how this can be done locally, relative to the lane. This has two major limitations. First, we don't know how far we have travelled in the lane, as we only can get estimates on our lateral offset from the center of the lane. Second, we don't know if we are in a straight or turning lane segment. In this exercise, however, we will work towards localizing the robot in a global coordinate frame. 

<!--The code developed for this exercise can be used in the coming exercise for a parallel parking task! Therefore, throughout this exercise, keep in mind that better performance of your localization code will make your control task later much easier. -->

To be able to perform such a complex task, we will have to use all sensors onboard the robot, namely the camera and the wheel encoders. We will achieve our localization goal in several incremental steps. First, we will develop an estimator using only wheel encoders. Then, we will develop an estimator using only camera information via the Apriltag detection library.  Finally, we will fuse these estimates so that we hopefully get the best of both worlds.

#### Encoder localization package {#exercise:encoder_localization}

The first package you'll create in your Dockerized ROS workspace is one that will contain a node that publishes a `TransformStamped` message at 30Hz with the following fields: 

* `frame_id: map`;
* `child_frame_id: encoder_baselink`;
* `stamp`: timestamp of the last wheel encoder tick;
* `transform`: a 2D pose of the robot baselink (see [](#fig:baselink-frame-def) for a definition of this frame).

The node will also have to broadcast the TF `map`-`encoder_baselink`.

**Deliverables:**

* A screen recording similar to this [](#encoder-exercise-example) where you drive the robot in a loop and try to "close the loop" (by driving to the point where you started from). Make sure the images are recorded as well as the TF tree in Rviz. Use a visible landmark as the origin of the map frame.

* A link to a github repository containing a package called `encoder_localization`

**Hints:**

* For the estimates to be in a global frame, you will have to provide an initial pose estimate.

* Use a `tf.TransformBroadcaster()` to broadcast a TF which can be visualized in RViz. Note that you can pass the exact same message as the one that your Publisher uses.

* For the kinematic model of the Duckiebot, you'll have to load the following parameters from the kinematics calibration file: `baseline`, `radius`.

* To open RViz, simply run `dts start_gui_tools hostname.local` and run `rviz` from inside the container. Note that if you keep the container running you can save your RViz configuration file so that when you reopen it, it automatically displays your topics of interest.

* Rviz uses the following color-code convention for frame axes: red for the x-axis, green for the y-axis, and blue for the z-axis.

* To publish messages at a fixed frequency, consider using a [ROS Timer](http://wiki.ros.org/roscpp/Overview/Timers). It works very similar to a subscriber except for the fact that the callback get called after a fixed duration of time.

* [This link](http://docs.ros.org/en/jade/api/tf/html/python/transformations.html) contains many useful tf library methods which allow you to switch between representations (e.g. expressing an euler yaw angle as a quaternion)

<end/>

<figure id="encoder-exercise-example">
    <figcaption>Example video deliverable for the encoder localization package </figcaption>
    <dtvideo src="vimeo:477772450"/>
</figure>

<div figure-id="fig:baselink-frame-def" figure-caption="Position of the baselink and camera frames on the robot">
  <img src="baselink_camera_frame.jpg" style='width: 30em; height:auto'/>
</div>

As you have probably realized, some of the advantages of this localization system is that as long as the robot is moving, the wheel encoders will provide information about the state of the robot, at a high rate and with little delay. However, the pose of the robot is an integration of the measurements of the robot, meaning that it is subject to drift as any inaccuracies in measurement get propagated through time. When driving aggressively or through a slippery surface these inaccuracies are amplified (try this for yourself!). 

A common way of getting rid of drift in mobile robots is to either use absolute position measurements (with GPS, for instance) or to use fixed landmarks in the world as reference. This is how we humans also navigate the environment. Since there is no GPS on-board the Duckiebot, we will have to use the latter approach. This is what we will explore in the next exercise, where our landmarks will be traffic signs with Apriltags (see [](#fig:traffic-sign-at)).

<div figure-id="fig:traffic-sign-at" figure-caption="Example traffic sign to be used for Apriltag localization. In order to accurately construct your TF tree, please measure the height indicated">
  <img src="traffic_sign_at.jpg" style='width: 12em; height:auto'/>
</div>


#### Apriltag localization package {#exercise:apriltag_localization}


In this package you'll have to place a node that subscribes to `/camera_node/image/compressed` and publishes a `TransformStamped` message (if an image with an Apriltag has been received) with the following fields: 

* `frame_id: map`;
* `child_frame_id: at_baselink`;
* `stamp`: timestamp of the last image received;
* `transform`: a 3D pose of the robot baselink (see [](#fig:baselink-frame-def) for a definition of this frame).

This node will also have to broadcast the following TFs: `map`-`apriltag`, `apriltag`-`camera`, `camera`-`at_baselink`. Make sure that when you place the Apriltag in front of the robot you get something that looks roughly like [](#rviz-final-tf-tree). This will make it easier to fuse this pose with the pose from the encoders.

**Deliverables:**

* A screen recording similar to the one in [](#at-exercise-example-compressed) where you move the Apriltag in front of the robot from one side of the Field-of-View (FOV) to the other. Make sure the images are recorded as well as the TF tree in Rviz. 

* Instead of directly using compressed images, rectify them before passing them to the Apriltag detector. You will see that this significantly improves the accuracy of the detector but at the cost of significant delay. Provide a screen recording similar to this [](#at-exercise-example-rectified) where you move the Apriltag in front of the robot from one side of the FOV to the other. Make sure the images are recorded as well as the TF tree in Rviz. You should observe that the `map` and `baselink` are now in the same plane (or at least much closer to it than with compressed images).

* (Bonus) Offload the computation of the rectified image to the GPU of the Jetson Nano so that the improved accuracy can be obtained without significant delay.

* A link to your Github repository containing a package called `at_localization`

**Hints:**

* A frame cannot have two parents. If you try to broadcast the following TFs: `map`-`baselink` and `camera`-`baselink`, you will see that you'll only be able to see the individual frames in RViz.

* To rectify images, use the following `cv2` methods: 

```python
newCameraMatrix = cv2.getOptimalNewCameraMatrix(cameraMatrix, distCoeffs, (640, 480), 1.0)
cv2.initUndistortRectifyMap(cameraMatrix, distCoeffs, np.eye(3), newCameraMatrix, (640, 480), cv2.CV_32FC1)
cv2.remap(compressed_image, map1, map2, cv2.INTER_LINEAR)
```

* To use the Apriltag detector, consult [this link](https://github.com/duckietown/lib-dt-apriltags). You can extract the pose of the Apriltag in the camera frame with `tag.pose_R` (rotation matrix) and `tag.pose_t` (translation vector). Keep in mind that the coordinate frame convention (see [](#fig:at-lib-frame-convention)) is different than the one you are supposed to use in the deliverable!

* To broadcast static transforms, use `self.static_tf_br = tf2_ros.StaticTransformBroadcaster()`. To calculate the static transform between the `map` and `apriltag` frames ($$ T_{MA} $$) use [](#fig:traffic-sign-at) as reference. To calculate the static transform between the `baselink` and `camera` frames, you can use 
[](#fig:baselink-frame-def) as reference, or you can try to use the homography matrix obtained during extrinsic calibration.

* To convert between frames, we recommend that you use 4x4 transformation matrices. An example of such a matrix is $$ T_{AB} = \begin{bmatrix}
R_{AB} & {}_{A}\vec{r}_{AB}\\
\vec{0} & 1
\end{bmatrix} $$ which transforms a vector in frame B to frame A. During your manipulations, keep in mind that $$ T_{BA} = (T_{AB})^{-1} $$ and $$ T_{AB} = T_{AC} T_{CB} $$ for any frames $$A, B $$ and $$C$$.

<end/>

<div figure-id="fig:at-lib-frame-convention" figure-caption="Frame convention used by Apriltag library when returning pose.">
  <img src="at_camera_frames.jpg" style='width: 30em; height:auto'/>
</div>


<figure id="rviz-final-tf-tree">
    <figcaption>Goal TF tree for the AT localization package with rectified images and the robot facing the Apriltag</figcaption>
    <dtvideo src="vimeo:477649817"/>
</figure>

<figure id="at-exercise-example-compressed">
    <figcaption>Example video deliverable for the AT localization package with compressed images </figcaption>
    <dtvideo src="vimeo:477771555"/>
</figure>

<figure id="at-exercise-example-rectified">
    <figcaption>Example video deliverable for the AT localization package with rectified images </figcaption>
    <dtvideo src="vimeo:477771932"/>
</figure>


The Apriltag detected provides accurate pose estimates with respect to landmarks but at the cost of delay and low frequency. Moreover, one cannot continuously rely on such estimates since the Apriltag could go out of sight. To combat this, we can fuse the estimates of the Apriltag and the wheel encoders for continuous, accurate and robust localization. This is the goal of the next exercise.

#### Fused localization package {#exercise:fused_localization}

In this package you'll have to place a node that publishes a `TransformStamped` message with the following fields: 

* `frame_id: map`;
* `child_frame_id: fused_baselink`;
* `stamp`: current time, 
* `transform`: a 2D pose of the robot baselink. 

The node will also have to broadcast the TF `map`-`fused_baselink`. As a minimum, your fusion node should have the following behaviour:

* The first time the Apriltag becomes visible, you have to calibrate the `encoder_baselink` frame/estimate to match exactly the Apriltag pose. This should be done with a [ROS Service](http://wiki.ros.org/ROS/Tutorials/WritingServiceClient%28python%29) provided by  and `encoder_localization_node` (server) which `fused_localization_node` (client) can call. If you need an example of a Duckietown package that defines a similar service, check [this](https://github.com/duckietown/dt-core/tree/daffy/packages/navigation) out (pay a special attention to the `CMakeLists.txt` and `package.xml` files).

* Publish the robot pose using the Apriltag estimate (when available) projected on the ground plane (recall that this node publishes 2D poses).

* If the Apriltag is not visible, use the encoder estimates, starting from the last Apriltag pose received (there should be no pose jump if the Apriltag goes out of sight).

* If the Apriltag becomes visible again, switch back to using Apriltag estimates (a jump in fused pose is allowed)

* You don't have to handle the delay or the variance of the individual estimates to complete the exercise, but you are more than welcome to! <!--Remember that the more accurate you make this node, the better your parking controller will perform in the coming exercise!-->

**Deliverables:**

* A screen recording similar to this [](#fused-exercise-example-rectified-slow),  where you drive the robot in a loop around the Apriltag with the virtual joystick. Start the recording with the Apriltag not visible, so that you validate that your calibration service is working and when it does bacome visible all the frames align. You should end the trajectory where you started it (feel free to use a marker on the ground). Make sure the images are recorded as well as the TF tree in Rviz. 
<!-- 
* (Bonus) A screen recording where you use rectified images and still drive fast. This will require you to handle the delay of the Apriltag estimates in some way. -->

* A link to a github repository containing a package called `fused_localization`

<end/>

<!-- TODO: change video link -->
<!-- <figure id="fused-exercise-example-compressed-agressive">
    <figcaption>Example video deliverable for the fused localization package with compressed images and aggressive driving</figcaption>
    <dtvideo src="vimeo:477202732"/>
</figure> -->

<figure id="fused-exercise-example-rectified-slow">
    <figcaption>Example video deliverable for the fused localization package with rectified images and slow driving</figcaption>
    <dtvideo src="vimeo:477772244"/>
</figure>