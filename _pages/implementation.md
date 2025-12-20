---
title: "Implementation"
permalink: /implementation/
layout: single
use_math: true
---

## Perception

To implement our perception stack design, we use a [Realsense D435i](https://www.realsenseai.com/products/depth-camera-d435i/) mounted next to the table, facing across it toward the robot. An ArUco marker is fixed to the base of the robot to enable transformations between the camera frame and robot frame. A `static_tf_transform` node publishes the transformation between the ArUco tag and base link.

{% include figure image_path="/assets/media/setup.png" width="70%" caption="In our setup, the depth camera sits next to the table and looks across it toward the robot." %}

A `cube_detector` node subscribes to the RGB-D point cloud. The point cloud is filtered with depth thresholds and RANSAC-based plane fitting, and then clustered with DBSCAN (see [Design](/206A-group19-finalreport/design/)). For RANSAC, we use a residual threshold of $0.02$ m, meaning any points closer than $0.02$ m from the plane are considered inliers. For DBSCAN, we use $\epsilon = 0.015$ m and `minPts` $= 40$. All of these parameters were determined experimentally. We classify the obstacle as the point cluster with an average blue color with $\geq 10000$ points. This number was decided based on the observation that the sizes of the cube pointclouds were roughly 1000-2000 points and the size of our sample obstacle was roughly 20,000 points. The rest of the clusters are treated as objects and classified based on their average RGB value.

We publish the obstacle as a custom `BoxBounds` message type, which contains six floats: `x_min`, `x_max`, `y_min`, `y_max`, `z_min`, and `z_max`. These values define the smallest axis-aligned bounding box that envelops the obstacle.

The objects are published in a custom `CubeArray` message type, which contains a list of custom `LabeledCube` messages. Each `LabeledCube` message contains a `geometry_msgs/PointStamped` message defining the object centroid, as well as a string named `color_label` which defines the object classification from (`black`, `red`, `blue`, `green`).

The `BoxBounds` and `CubeArray` messages are published in the local (camera) frame. We use another node, `transform_perception`, to transform the position information in these messages into the robot base frame. To prevent our planner from finding trajectories around the obstacle (as opposed to above it), we extend the transformed obstacle `BoxBounds` message by $0.5$ m in the +/- y directions with respect to the robot `base_link` frame. This also allows us to resolve problems arising from missing depth information (since we are only working with one camera).

{% include figure image_path="/assets/media/dbscan.gif" width="70%" caption="Published cube positions and obstacle bounding box (before y-axis extension) in RViz." %}

## Planning and Control

Our controls stack lives in the ROS 2 planning package. The main node, `main_mpc_new.py`, subscribes to `JointState`, `LabeledCubeArray` (cube centroids + color), and `BoxBounds` (obstacle AABB) in `base_link`, with TF as a fallback if any message arrives in another frame. It assigns fixed drop locations based on color (e.g red vs. black), queues cubes, and toggles the gripper via a `Trigger` service.

The NMPC controller uses a 30-step horizon at $\Delta t = 0.1$ s with per-step joint limits of $\pm 0.15$ rad. The end effector with grasped cube is approximated by multiple inflated proxy spheres (radii ≈ 0.03–0.05 m) that must clear the perceived obstacle AABB at every step (hard constraint). Table clearance is enforced with slack, and a facing-down condition is enforced only at the terminal waypoint (soft slack). Costs emphasize end-effector position error and terminal accuracy (terminal weight multiplier = 10) with a lighter penalty on joint step magnitude; orientation tracking along the path is disabled.

Execution follows a short-horizon closed loop: solve NMPC with the latest joint state and obstacle AABB, execute the first three 0.15 s joint steps from that solution via `FollowJointTrajectory`, then replan with the updated state. The loop stops when the goal is within 3 cm or a maximum iteration cap (40 solves) is hit. Early failures were traced to oversized proxy spheres (false collisions) and an imbalanced Q/R that favored smoothness over goal seeking; shrinking the spheres and retuning the weights restored feasibility and consistent convergence around the hard obstacle constraints.

`warehouse_sorting_bringup.launch.py` starts the perception transformers and the planning node with these parameters.

We validated the closed-loop NMPC in MuJoCo before hardware. The controller cleared the obstacle and delivered cubes to their drop zones while respecting the proxy-sphere AABB constraints.


{% include figure image_path="/assets/media/mujoco_pt2.png" width="70%" caption="Start of the MPC solve (cube grasped, obstacle AABB visible)." %}
{% include figure image_path="/assets/media/mujoco_pt3.png" width="70%" caption="Approaching the drop zone; terminal facing-down enforced only at the goal." %}




