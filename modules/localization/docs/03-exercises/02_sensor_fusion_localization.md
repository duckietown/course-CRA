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

The first package you'll have to create in your Dockerized ROS workspace is one that will contain a node that publishes a `TransformStamped` message at 30Hz with the following properties: `frame_id: map`, `child_frame_id: encoder_baselink`, `stamp`: timestamp of the last wheel encoder tick, `transform`: a 2D pose of the robot baselink (see FIGURE for a definition of this frame).

Deliverable:

* A screen recording similar to this EXAMPLE where you drive the robot in a loop and try to close the loop. Make sure the images are recorded as well as the TF tree in Rviz. Use a visible landmark as the origin of the map frame.

* A link to a github repository containing a package called `encoder_localization`

Hints:

* For the estimates to be in a global frame, you will have to provide an initial state.

* Use a `tf.TransformBroadcaster()` to broadcast a TF which can be visualized in RViz. Note that you can pass the exact same message as the one that your Publisher uses.

* For the kinematic model of the Duckiebot, you'll have to load the following parameters from the calibration file: `baseline`, `radius`.

* To open RViz, simply run `dts start_gui_tools hostname.local` and run `rviz` from inside the container. Note that if you keep the container running you can save your RViz configurtion file so that when you reopen it, it automatically displays your topics of interest.




