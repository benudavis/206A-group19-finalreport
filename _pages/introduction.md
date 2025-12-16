---
title: "Introduction"
permalink: /introduction/
layout: single
---

# Introduction

Pick and place sorting is a common application of robotics in areas like e-commerce fulfillment, food packaging, and recycling. In such applications, robots must incorporate robust perception and planning/control pipelines, so that cluttered objects can be located, classified, and moved efficiently and with minimal errors. This presents some major engineering challenges. For perception, how can RGB-D data be reliably parsed into accurate object positions and classifications, especially in environments with high clutter and occlusions? For planning and control, how should the robot move to pick the object and move it to its destination without colliding with other objects or obstacles in the environment?

{% include figure image_path="/assets/media/robot_sorting.png" width="50%" caption="Recycling is one promising application for robotic pick and place sorting." %}

In this project, our end goal was to demonstrate a robot that can successfully pick objects out of a clutter, classify them, and guide them to their destinations with collision-free trajectories. To do so, we were required to develop and implement strategies for the two challenges mentioned above--extracting object classes and locations from RGB-D data, and planning collision-free trajectories for the robot during object placement.