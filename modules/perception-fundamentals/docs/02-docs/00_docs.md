# Learning materials {#cra-per-lm status=ready}

## Introduction

During lectures, we explained the perception direction of the image pipeline:  

<figure>
  <img style="width:30em" src="images/image_pipeline.png"/>
</figure>

In this exercise, we are going to look at the pipeline in the opposite direction.

It is often said that:

> "The inverse of computer vision is computer graphics."  

The inverse pipeline looks like this:  

<figure>
  <img style="width:30em" src="images/graphics.png"/>
</figure>  

In simple words, instead of extracting information from our camera, we want to introduce some data in the imagery.

For this exercise concept like camera calibration, homography and projection matrices, image plane and world coordinates are essential. So be sure to have those in mind while you work your way through the exercises in the next sections.  

Here is a quick reminder on what the homography matrix is and how it is obtained:

<figure>
  <img style="width:30em" src="images/homography_matrix.png"/>
</figure>

<figure>
  <img style="width:30em" src="images/homography_calculations.png"/>
</figure>