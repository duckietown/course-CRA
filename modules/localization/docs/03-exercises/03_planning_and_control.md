# Exercises - Sensor Fusion for Control {#exercise-sensor_fusion_control status=draft}

Excerpt: Teach your Duckiebot to parallel park using sensor fusion

In this exercise you will combine your knowledge of sensor fusion developed in the earlier sections to build a simple controller that can follow a pre-determined parallel parking maneuver.

<div class='requirements' markdown='1'>

  Requires: [Modeling the Duckiebot and Representations](+duckietown-robotics-development#representations-modeling)

  Requires: [Advanced Augmented Reality Exercise](#cra-apriltag-augmented-reality-exercise)

  Requires: [Sensor Fusion Localization Exercise](#exercise-sensor_fusion)

  Results: Teaching your Duckiebot how to parallel park!

  Results: Develop a controller for following a pre-defined path.

  Results: Create a multi-package Dockerized ROS workspace, and deploy it on the Duckiebot.

</div>

The goal of this exercise is to combine your knowledge from ROS, Docker, sensor fusion and control to develop a novel parking maneuver for your Duckiebot. The exercise has  several incremental steps which will help you develop the new functionality. You will create a new ROS package that will combine your knowledge from the previous exercise in sensor fusion, as well as all your previous experience working with Wheel Encoders, cameras, and ROS. The main deliverables are listed below:

1. Planning in ROS

2. Control development

An example of what the solution should look like is shown in [](#fig:parking-maneuver)

<div figure-id="fig:parking-maneuver" figure-caption="A Duckiebot performing parallel parking">
  <img src="parking-maneuver.gif" style='width: 30em; height:auto'/>
</div>

#### Planning in ROS {#exercise:planning-in-ros}

The first step in this complex task will be to create a new ROS package for your controller. This will be very similar to your previous implementation, and you can use the same template as with the Apriltag or Wheel Encoder solution.

The idea is to make the robot move, therefore, we want the robot to do two things: subscribe to the appropriate sensor stream, and also publish the commanded velocity (linear and angular) for the Duckiebot. Your solution has to be able to subscribe to the three packages you developed in [sensor fusion exercise](#exercise-sensor_fusion) on demand.

  TIP: Use environmental variables for switching between sensor streams once the solution has been built. Reminder, environmental variables can be passed as command line information using the syntax `--export MY_VAR` or `-e MY_VAR`, and are retrieved in Python using `my_var = os.environ['MY_VAR']`.

For making the Duckiebot move, you can do two things, either publishing directly to the wheels using the `wheels_driver_node/wheel_cmd` topic with a message of type `WheelsCmdStamped` which includes the angular velocity of each wheel. Alternatively, you can publish the car's linear and angular velocity of its center of rotation (Point A in [](#fig:kinematics-db)) by publishing a message of type `Twist2DStamped`. The kinematics node handles the conversion to individual wheel velocities.

  TIP: You can publish directly to the kinematics node by remapping your topic to the subscriber in the kinematics node. This is done using the `<remap from="/topic_example1" to="/topic_example2" />` tag in your launchfile.

<div figure-id="fig:kinematics-db" figure-caption="Kinematics model of a differential drive robot">
  <img src="mod-kin.jpg" style='width: 30em; height:auto'/>
</div>

##### Planning in ROS

To make all this happen, you will first have to plan a route to follow in the global reference frame. This can be done using the [ROS navigation messages package](http://wiki.ros.org/nav_msgs). You will create a plan, and publish it as a `nav_msgs/Path` message which is just an array containing `geometry_msgs/Pose` type messages.

Your task is to measure your desired parking maneuver in the physical world (e.g. 30cm along the lane, and 22cm parallel to the lane), and create a trajectory in 2D that the Duckiebot will follow. Convert the trajectory to a message of type `nav_msgs/Path()` and publish it. Example of what the measuring might look like is shown in [](#fig:x_10), [](#fig:x_20) and [](#fig:x_30):

<table>
<tr>
<td><div figure-id="fig:x10" figure-caption="Duckiebot 10 cm away from Apriltag">
  <img src="x_10.png" style='width: 20em; height:auto'/>
</div></td>
<td><div figure-id="fig:x10_map" figure-caption="Camera view 10cm away from Apriltag">
  <img src="x_10_cam.png" style='width: 20em; height:auto'/>
</div></td>
</tr>
</table>

<table>
<tr>
<td><div figure-id="fig:x20" figure-caption="Duckiebot 20 cm away from Apriltag">
  <img src="x_20.png" style='width: 20em; height:auto'/>
</div></td>
<td><div figure-id="fig:x20_map" figure-caption="Camera view 20cm away from Apriltag">
  <img src="x_20_cam.png" style='width: 20em; height:auto'/>
</div></td>
</tr>
</table>

<table>
<tr>
<td><div figure-id="fig:x30" figure-caption="Duckiebot 30 cm away from Apriltag">
  <img src="x_30.png" style='width: 20em; height:auto'/>
</div></td>
<td><div figure-id="fig:x30_map" figure-caption="Camera view 30cm away from Apriltag">
  <img src="x_30_cam.png" style='width: 20em; height:auto'/>
</div></td>
</tr>
</table>

Deliverable 1: a screen recording of your Duckiebot moving in rviz where the `baselink` reference frame is shown moving with respect to a fixed path in the global reference frame. see [](#path-spline-video) for a sample submission.

 A screen recording similar to this [](#encoder-exercise-example)
 <figure id="path-spline-video">
     <figcaption>Example video deliverable for the ROS navigation </figcaption>
     <dtvideo src="path-spline-sample.mp4"/>
 </figure>

TIP: For moving your Duckiebot at this stage, you can use the `dts duckiebot keyboard_control HOSTNAME` command to manually control your robot. This is useful for debugging.

#### Control Development - Parking Maneuver {#exercise:parking-maneuver}

Now that you have a robot that is able to localize itself in the global reference frame, in addition to having a predetermined path to follow, your task is to develop a controller to make the Duckiebot parallel park, repeatedly. The goal of this section is to qualitatively compare the performance of your controller as it uses the data from different sensors. You are free to use any type of controller you want (e.g. Pure Pursuit, Go To Angle, etc.).

Deliverable 2: Three separate recordings of your Duckiebot performing the parallel parking maneuver a minimum of three full cycles each. Once using only the encoder signal, once using only the Apriltag, and once using the fused signal.

  * A full cycle is defined as starting in front of the Apriltag, moving backwards to park, and then going back to the starting position.

  * Provide a link to your github repository containing the package called `parking_control_node`.



<end/>
