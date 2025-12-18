---
title: "Implementation"
permalink: /implementation/
layout: single
---

## Perception

To implement our perception stack design, we use a [Realsense D435i](https://www.realsenseai.com/products/depth-camera-d435i/) mounted next to the table, facing across it toward the robot. An ArUco marker is fixed to the base of the robot to enable transformations between the camera frame and robot frame. A `static_tf_transform` node publishes the transformation between the ArUco tag and base link.

{% include figure image_path="/assets/media/setup.png" width="70%" caption="In our setup, the depth camera sits next to the table and looks across it toward the robot." %}

A `cube_detector` node subscribes to the RGB-D point cloud. The point cloud is filtered with depth thresholds and RANSAC-based plane fitting, and then clustered with DBSCAN (see [Design](/design/)). For RANSAC, we use a residual threshold of 0.02 m, meaning any points closer than 0.02 m from the plane are considered inliers. For DBSCAN, we use $\epsilon = 0.015$ m and `minPts` $= 40$. All of these parameters were determined experimentally. We classify the obstacle as the point cluster with $\geq 10000$ points. The rest of the clusters are treated as objects and classified based on their average RGB value.

We publish the obstacle as a custom `BoxBounds` message type, which contains six floats: `x_min`, `x_max`, `y_min`, `y_max`, `z_min`, and `z_max`. These values define the smallest axis-aligned bounding box that envelops the obstacle.

The objects are published in a custom `CubeArray` message type, which contains a list of custom `LabeledCube` messages. Each `LabeledCube` message contains a `geometry_msgs/PointStamped` message defining the object centroid, as well as a string named `color_label` which defines the object classification from ('black', 'red', 'blue', 'green').

The `BoxBounds` and `CubeArray` messages are published in the local (camera) frame. We use another node, `transform_perception`, to transform the position information in these messages into the robot base frame. To prevent our planner from finding trajectories around the obstacle (as opposed to above it), we extend the transformed obstacle `BoxBounds` message by 0.5 m in the +/- y directions with respect to the robot base frame.

## Planning and Control


...

A launch file `warehouse_sorting_bringup.launch.py` deals with launching the helper nodes, and a `main.py` script implements the main loop logic.