---
title: "Design"
permalink: /design/
layout: single
use_math: true
---

## System Design and Task Goals

Our goal for this project is to design a system that can successfully parse a scene with multiple objects and an obstacle and move all objects to their respective destination without colliding with the obstacle. For simplicity, we used small red and black cubes as our objects, and we use a blue rectangular box as the obstacle. Colored box outlines on the table to mark destinations. Since our perception stack is focused on object and obstacle classification and localization, we opt to hard-code these destinations.

{% include figure image_path="/assets/media/task_design.JPEG" width="70%" caption="Our setup involves multiple red and black objects, a blue obstacle, and two place destinations." %}

To accomplish this task, we design our system in two modules: perception and planning/control. We also separated the task into two phases: reaching and moving. In the reaching phase, the robot must extract object locations and classes from the RGB-D image, select one object to pick, and reach that object using MPC or plain IK. In the moving phase, the held object is already classified, so the destination is already known. The robot only needs to localize the obstacle and plan and execute a trajectory around it.

{% include figure image_path="/assets/media/system_diagram.png" width="70%" caption="Our system can be understood in terms of reaching and moving phases, with distinct perception and planning/control blocks within both phases." %}

## Design: Perception

Our perception stack takes live RGB-D data, filters out irrelevant points, organizes the remaining points into clusters, and then uses color and size thresholding to classify individual objects and the obstacle.

To filter the raw depth image points $P$, we apply a series of two filters. The first filter removes points further than a certain distance away from the camera. This ensures that the remaining points are within the within the relevant workspace, which includes the robot, objects, obstacle, and table.

$$ S = \{(x_{cam},y_{cam},z_{cam}) \in P | z_{cam} < z_{thresh}\} $$

Next, we use [RANSAC](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.RANSACRegressor.html) (RANdom SAmple Consensus) to find the plane of the table surface. We select RANSAC due to its high robustness to outliers. Our point cloud contains primarily points from the table surface, but also points from the robot, objects, and obstacle. Alternative options, such as least-squares fitting, would still give some weight to these "outlier" points, even though they are not a part of the plane. With RANSAC, the consensus plane can appropriately treat these points as outliers and not take them into account. We then apply a filter to $S$ to remove all points outside of a region above the table:

$$ S' = \{(x_{cam},y_{cam},z_{cam}) \in P | y_{cam} - y'(x_{cam},z_{cam}) \in [-0.5,-0.02] cm\} $$

Where $y'(x_{cam},z_{cam})$ is the RANSAC-predicted $y$ given $(x_{cam},z_{cam})$.

## Design: Planning and Control