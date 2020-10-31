# Advanced Augmented Reality Exercise {#cra-apriltag-augmented-reality-exercise status=ready}

Excerpt: Apply your competences in software development in a perception pipeline.

The goal of this exercise is to put your skills in computer graphic to the test by projecting a complex 3D model on an AprilTag in a random position.

<div class='requirements' markdown='1'>
  Requires: [Camera calibration](+opmanual_duckiebot#camera-calib)

  Requires: [Docker basics](+duckietown-robotics-development#docker-basics)

  Requires: [ROS basics](+duckietown-robotics-development#sw-advanced)

  Requires: [Knowledge of the software architecture on a Duckiebot](+duckietown-robotics-development#duckietown-code-structure)
  
  Requires: [Basic Augmented Reality Exercise](+duckietown-classical-robotics#cra-basic-augmented-reality-exercise status)

  Results: Skills on how to develop new code as part of the Duckietown framework

  Results: Insights into a computer graphics pipeline.
</div>


<!--
* add dt-apriltags in py3 dependencies
* april tag reference https://github.com/duckietown/lib-dt-apriltags
* We provide you the renderClass.py, this is a class that allows you to render a 3D .obj model (It is a modified version of http://www.pygame.org/wiki/OBJFileLoader) 
  * this class contains the following method:
      * render(img, projection_matrix) where img is the image you want to project the model on. Projection Matrix is a 4x3 matrix that project the the obj on the qr code
  * When you define the class construction you need to pass the .obj model (we provide it to you).
    put this model in a res folder in your src. then do the following:
      ```Python
      rospack = rospkg.RosPack()

      # Render the obj
      self.rend = Renderer(rospack.get_path('YOUR PACKAGE NAME')+'/src/res/duckie.obj')
      ```
* In order to load the calibration matrices of the camera do the follwoing ....s


QUESTIONS 
* try to play with the parameters nthreads and quad_decimate what is it changing, is the rendering improving and how? -->

