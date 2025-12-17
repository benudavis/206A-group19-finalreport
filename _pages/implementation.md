---
title: "Implementation"
permalink: /implementation/
layout: single
---

## Perception

To implement our perception stack design, we use a [Realsense D435i](https://www.realsenseai.com/products/depth-camera-d435i/) mounted next to the table, facing across it toward the robot. An ArUco marker is fixed to the base of the robot to enable transformations between the camera frame and robot frame.

{% include figure image_path="/assets/media/setup.png" width="70%" caption="In our setup, the depth camera sits next to the table and looks across it toward the robot." %}

## Planning and Control