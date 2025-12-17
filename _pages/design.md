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

$$ S = \{(x,y,z)_{cam} \in P \mid z_{cam} \leq z_{thresh}\} $$

Next, we use [RANSAC](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.RANSACRegressor.html) (RANdom SAmple Consensus) to find the plane of the table surface. We select RANSAC due to its high robustness to outliers. Our point cloud contains primarily points from the table surface, but also points from the robot, objects, and obstacle. Alternative options, such as least-squares fitting, would still give some weight to these "outlier" points, even though they are not a part of the plane. With RANSAC, the consensus plane can appropriately treat these points as outliers and not take them into account. We then apply a filter to $S$ to remove all points outside of a region above the table:

$$ S' = \{(x,y,z)_{cam} \in P \mid -0.5\text{ m} \leq d((x,y,z)_{cam}) \leq -0.02\text{ m}\} $$

Where $d$ is the distance from $(x,y,z)_{cam}$ to the RANSAC-fitted plane. In complex environments where many plausible planes may be present in the pointcloud, this approach may be insufficient. However, in our task, the table forms a dominant plane feature in the pointcloud, meaning RANSAC can reliably find it.

We then use [DBSCAN](https://www.open3d.org/docs/latest/tutorial/Basic/pointcloud.html) (Density-Based Spatial Clustering of Applications with Noise) to cluster the filtered pointcloud $S'$ into individual points. Although our objects and obstacles have primitive rectangular shapes, DBSCAN's ability to find clusters of irregular shapes would have allowed us to work with less normal objects (should we have gotten that far). Had we used an approach like K-Means clustering, we would have been limited to roughly spherical shapes. Additionally, since the RGB-D image can be noisy and sometimes generate outlier points, DBSCAN's natural outlier handling (by ignoring points with less than the minimum amount of neighbors) improves our perception stack robustness. But, for DBSCAN to be successful, it is important that we tune the parameters $\epsilon$ and `minPts` to correctly identify clusters while rejecting noise. Once the objects are clustered, we classify by size and color. Since we know that our obstacle will be blue and the biggest cluster, we can extract it accordingly. Similarly, we can distinguish between red and black objects by comparing average color.

One major drawback of this approach is that we cannot handle objects in contact with each other in stacked or adjacent configurations. Some heuristic approaches based on point cloud size and color may be able to fix this, although generalization to non-uniform objects would be tricky. Another approach would be to use some form of instance segmentation to identify individual objects, and then create masks to find each object's associated pointcloud. However, the complexity of these approaches did not fit the limited timeline of this project.

## Design: Planning and Control

here, I think we should talk about MPC formulation: state vector, cost, constraints, etc