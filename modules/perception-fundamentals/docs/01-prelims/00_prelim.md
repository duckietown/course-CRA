# Preliminaries {#cra-per-prelim status=ready}

## Required steps

### Camera calibration

In this exercise the only sensor that will be used is the camera. So, ensure that you have already done intrinsics and extrinsics [camera calibration](https://docs.duckietown.org/daffy/opmanual_duckiebot/out/camera_calib.html) of your Duckiebot.

## Workflow tips

In this exercise you will extensively use images and inevitably make some mistakes that you will have to fix.
Debugging ROS packages that manipulate images can be very difficult only from the terminal output of your node and visualizing the image stream can be useful. 
Within the Duckietown infrastructure we have a very nice way to do that: use duckietown shell.

You can start a container connected to the ROS master on you Duckibot with:

laptop $ dts start_gui_tools ![DUCKIEBOT_NAME] 

Once inside this container you can call the rqt commands to visualize the state of the pipeline on you robot. 

    duckiebot-container $ rqt_image_view
    
    duckiebot-container $ rqt_graph

Or you can also use all the ROS related commands like rostopic, rosparam, rosnode, etc..
  
    duckiebot-container $ rostopic ...
    
    duckiebot-container $ rosparam ...

 