---
title: "Design"
permalink: /design/
layout: single
use_math: true
---

## System Design and Task Goals

Our goal for this project is to design a system that can successfully parse a scene with multiple objects and an obstacle and move all objects to their respective destination without colliding with the obstacle. For simplicity, we used small red and black cubes as our objects, and we use a blue rectangular box as the obstacle. Colored square outlines on the table mark destinations of the colored cubes. Since our perception stack is focused on object and obstacle classification and localization, we opt to hard-code these final destinations.

{% include figure image_path="/assets/media/task_design.JPEG" width="70%" caption="Our setup involves multiple red and black objects, a blue obstacle, and two place destinations." %}

To accomplish this task, we design our system in two modules: perception and planning/control. We also separated the task into two phases: reaching and moving. In the reaching phase, the robot must extract object locations and classes from the RGB-D image, select one object to pick, and reach that object using MPC or plain IK with waypoints. In the moving phase, the grasped object is already classified, so the destination is already known. The robot only needs to localize the obstacle and plan and execute a trajectory around it.

{% include figure image_path="/assets/media/system_diagram.png" width="70%" caption="Our system can be understood in terms of reaching and moving phases, with distinct perception and planning/control blocks within both phases." %}

## Design: Perception

Our perception stack takes live RGB-D data, filters out irrelevant points, organizes the remaining points into clusters, and then uses color and size thresholding to classify individual objects and the obstacle.

To filter the raw depth image points $P$, we apply a series of two filters. The first filter removes points further than a certain distance away from the camera. This ensures that the remaining points are within the within the relevant workspace, which includes the robot, objects, obstacle, and table.

$$ S = \{(x,y,z)_{cam} \in P \mid z_{cam} \leq z_{thresh}\} $$

Next, we use [RANSAC](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.RANSACRegressor.html) (RANdom SAmple Consensus) to find the plane of the table surface. We select RANSAC due to its high robustness to outliers. Our point cloud contains primarily points from the table surface, but also points from the robot, objects, and obstacle. Alternative options, such as least-squares fitting, would still give some weight to these "outlier" points, even though they are not a part of the plane. With RANSAC, the consensus plane can appropriately treat these points as outliers and not take them into account. We then apply a filter to $S$ to remove all points outside of a region above the table:

$$ S' = \{(x,y,z)_{cam} \in S \mid -0.5\text{ m} \leq d((x,y,z)_{cam}) \leq -0.02\text{ m}\} $$

Where $d$ is the distance from $(x,y,z)_{cam}$ to the RANSAC-fitted plane. In complex environments where many plausible planes may be present in the pointcloud, this approach may be insufficient. However, in our task, the table forms a dominant plane feature in the pointcloud, meaning RANSAC can reliably find it.

We then use [DBSCAN](https://www.open3d.org/docs/latest/tutorial/Basic/pointcloud.html) (Density-Based Spatial Clustering of Applications with Noise) to cluster the filtered pointcloud $S'$ into individual points. Although the objects and obstacles in our environment have primitive rectangular shapes, DBSCAN's ability to identify clusters of arbitrary shapes would allow the system to generalize to less structured objects (though we stuck with cubes for this project). In contrast, clustering methods like K-Means clustering assume roughly spherical clusters and require all points to be assigned to a cluster.

RGB-D point clouds are often noisy and contain spurious depth measurements. DBSCAN explicitly labels points that do not satisfy a local density criterion as noise, providng built-in outlier rejection and improving the robustness of the perception pipeline. However, successful clustering depends on appropriate tuning of the neighborhood radius $\epsilon$ and the minimum number of points `minPts`. $\epsilon$ was set slightly larger than the average nearest-neighbor distance on object surfaces, and `minPts` was chosen to exceed the number of points typically produced by isolated depth noise. Once the pointcloud is clustered, the clusters are classified as objects (cubes) or obstacles by size and color. 

Since our obstacles were known to be larger than the cubes, clusters were classified as obstacles if the number of points in a cluster were greater than a set threshold. This implementation allows us to identify multiple obstacles, given they are above a certain size threshold. One challenge we saw, however, was that the robot itself would be identified as an obstacle while executing its trajectory. We remedied this by setting the color of the obstacle to be fixed as blue and classifying large clusters as obstacles if blue and as the robot otherwise. Similarly, red and black objects are distinguished by comparing cluster-level average color values.

One major drawback of this approach is that we cannot handle objects in contact with each other in stacked or adjacent configurations. Some heuristic approaches based on point cloud size and color may be able to fix this, although generalization to non-uniform objects would be tricky. Another approach would be to use some form of instance segmentation to identify individual objects, and then create masks to find each object's associated pointcloud. However, the complexity of these approaches did not fit the limited timeline of this project.

## Design: Planning and Control

The Model Prdictive Control (MPC) formulation for the project was designed to generate feasible joint-space trajectories for the 6-DoF UR7e robot arm with physical and environment constraints. By using a constrained finite-horizon optimaztion problem, it will the robot arm to navigate the objects to the target in a receding-horizon loop.

### Prediction Model

The robot's joint-space motion has been modeled using a discrete-time representation, where the state $x% is defined by joint positions $q \in \mathbb{R}^6$ and the control input $u$ is... TODO: Will edit after Parham finishes up in the lab

$$x_{k+1} = x_k + u_k \Delta t$$

