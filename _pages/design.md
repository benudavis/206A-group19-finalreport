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

To move the grasped cube around the obstacle, we used a joint-space nonlinear MPC (NMPC) that optimizes a short horizon, executes only the first step, and replans. The decision variables are joint positions $q_k$ and bounded joint steps $\Delta q_k$ with simple discrete-time dynamics over a 30-step horizon with $\Delta t = 0.08$ s. Each step is limited to $\pm 0.15$ rad to keep commanded motion smooth and physically feasible, and joint limits are enforced directly.


$$q_{k+1} = q_k + \Delta q_k$$


Collision avoidance is modeled by proxying the end effector plus grasped cube with three spheres placed along the tool ($[0,0,0]$, $[0,0,0.02]$, $[0,0,0.03]$ m) and radii of roughly 0.03–0.05 m after inflation. At every step, each sphere must clear the perceived obstacle AABB by at least its radius (hard constraint). Table clearance is handled with slack to avoid infeasibility when the table height estimate is tight, and the down-facing condition on the tool is applied only at the terminal waypoint (soft slack) so the arm can maneuver freely mid-trajectory but end with the gripper oriented downward at the drop pose.

The cost penalizes end-effector position error, joint step magnitude, and terminal position error (with a terminal weight 10× larger). Orientation tracking along the path is disabled—only the terminal facing-down condition is enforced,so the solver prioritizes obstacle clearance and goal convergence. Obstacle geometry comes straight from perception as an AABB in the robot frame, so any change in the obstacle estimate is reflected immediately in the constraint set.

Execution follows a receding-horizon loop: 

(1) solve NMPC with the latest joint state and obstacle AABB 

(2) execute the first $\sim$0.15 s joint step

(3) replan until the end effector is within a 3 cm goal tolerance or a maximum iteration cap. 

This closed-loop scheme was critical for robustness. Earlier open-loop plans would either violate the inflated obstacle bounds (when proxy radii were too large) or favor smoothness over goal seeking (when Q/R was unbalanced). Smaller proxy spheres and retuned weights, combined with frequent replanning, resolved those failures and kept the arm converging with respect to the hard obstacle constraints.

Key design trade-offs mirror the perception stack: hard AABB clearance for safety, softened table and terminal-facing constraints for feasibility, terminal-only orientation to reduce unnecessary path constraints, and short-horizon replanning to tolerate state drift and updated obstacle estimates.
