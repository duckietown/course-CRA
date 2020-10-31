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

## 3D Model Projection Exercise {#cra-apriltag-augmented-reality-exercise-introduction}

In this exercise you are asked to render a complete 3D model of a duckie on an image given a `.obj` file with its specification. Similarly to many augmented reality games the model needs to be projected on an easily recognizable flat pattern and in this exercise you will use an [AprilTag](https://april.eecs.umich.edu/software/apriltag).
You should have received some traffic signs with AprilTags with the initial Duckiebot box, something like:  
<figure>
  <img style="width:30em" src="images/apriltags.png"/>
</figure>

In the Duckietown infrastructure AprilTags are recognized and managed through the [lib-dt-apriltags](https://github.com/duckietown/lib-dt-apriltags). Check it out to see how to use it! 
 
In order to do that you will have to create a package called `augmented_reality_apriltag` with functionalities specified below in [](#cra-basic-augmented-reality-exercise-structure).

## Instructions {#cra-apriltag-augmented-reality-exercise-instructions}

1) This exercise package structure will be based on the one provided by the [AprilTag Template](link to custom repo). Inside there you will find everything you need like the 3D model and the provided files.
TODO(VINCENZO) CREATE AND LINK THE CUSTOM REPO, SPECIFY WHETHER IS A TEMPLATE TO BASE THERI REPO ON OR A REPO TO FORK AND MODIFY.
2) In this exercise you will have to use the AprilTag library so check that it has been added in the `dependencies-py3.txt` as `dt-apriltags`.
3) We provided you a renderClass.py. Inside there is the Renderer class which allows you to render a 3D .obj model drawing it onto the input image. If you are interested the code inside this file is a modified version of [Pygame OBJFileLoader](http://www.pygame.org/wiki/OBJFileLoader).
  The provided Renderer class contains the method `render(img, projection_matrix)` , where `img` is the image you want to project the model on and `projection_matrix` is the 3x4 matrix that transform the 3D model coordinates to the AprilTag reference system allowing you to project it.
  The constructor of an instance of renderClass asks the 3D model as input. Save the 3D model in a `src/models` folder and use the code below to correctly initialize an instance of Renderer class:
    ```Python
    # Import class from file.
    from renderClass import Renderer
    
    rospack = rospkg.RosPack()
    
    # Initialize an instance of Renderer giving the model in input.
    self.renderer = Renderer(rospack.get_path('YOUR PACKAGE NAME')+'/src/models/duckie.obj')
    ```
   Please do not change the file renderClass.py, the code which is inside renderClass has been tested any change might lead to unexpected errors and problems that will not be supported. 
4) You will also get a function to load the calibration parameters of your Duckiebot camera. TODO(VINCENZO) SPECIFY WHICH FUNCTION WE WANT TO GIVE THEM, ORIGINAL DUCKIETOWN OR OURS.


## Exercise Structure {#cra-apriltag-augmented-reality-exercise-structure}

Looking at the project as a whole may make it seem more difficult than it really is. But we will follow the latin way of saying `divide et impera (Divide and rule)`, braking down the problem in more manageable tasks that can be solved one after the other.

First let's focus on what we are trying to achieve: project in an image a 3D model of a figure whose position and orientation matches the position and orientation of some predefined flat surface, an AprilTag.
This can be summarized in a sequence of simpler steps, namely:
1) Recognize the reference flat surface: identify the flat surface of reference(the AprilTag) in an image.
2) Estimate the homography: determine the transformation from the reference surface image (2D) to the target image (2D), this transformation is the homography.
For this first two steps have a look to the AprilTag repository linked in  [](#cra-apriltag-augmented-reality-exercise-introduction), you will find it quite useful.
3) Derive from the homography the transformation from the reference surface coordinate system to the target image coordinate system : if we want to project a 3D model placed on top of the reference surface to the target image, we need to extend the previous transformation to handle cases were the height of the point to project in the reference surface coordinate system is different than zero. This can be achieved with knowledge about transformations a bit of algebra.
4) Project our 3D model in the image (pixel space) and draw it : you can use the provided Renderer class for this.

## Parameter Tweaking {#cra-apriltag-augmented-reality-exercise-parameters}

As you might have seen the AprilTag detection adds little delay to the pipeline. However, you can try to change some parameters for a better accuracy and speed trade-off. Try to play around with the parameters `nthreads` and `quad_decimate`, what does it change? Does the speed improve? What about the stability and accuracy of the detector?
TODO(VINCENZO) SPECIFY WHERE TO FIND THIS PARAMETERS.
