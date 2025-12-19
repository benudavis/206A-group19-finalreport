---
title: "Conclusion"
permalink: /conclusion/
layout: single
---
\textbf{Discussion of Results}
Overall, the finished system met the primary design objectives of parsing a cluttered scene, classifying objects, and executing collision-free trajectories to place them at predefined destinations. The modular separation between perception and planning/control enabled independent development and debugging of each component, while still allowing them to function cohesively in the final system. The successful sorting of multiple objects in the presence of a large obstacle demonstrates the effectiveness of the overall approach for structured pick-and-place tasks.

\textbf{Challenges Encountered}
Several challenges were encountered during development. From a perception standpoint, isolating the robot arm from the point cloud proved difficult, as it was initially detected as an obstacle. This issue was addressed by incorporating color-based classification to distinguish the blue obstacle from the robot. From a planning perspective, tuning the MPC parameters was particularly challenging; achieving stable, smooth trajectories while maintaining feasibility and obstacle avoidance required extensive manual tuning.

\textbf{Limitations and Future Improvements}
The current system relies on several heuristics that limit robustness. The obstacle must be a specific color (blue) to be reliably identified, which restricts generalization. With additional time, the robot could be removed from the point cloud more systematically using its known kinematic state or end-effector pose rather than color cues.
Grasping performance is also sensitive to calibration and perception accuracy. Small localization errors can cause the gripper to miss or collide with objects. Improvements such as better camera calibration, visual servoing, or compliant grasping strategies could significantly increase robustness.

Finally, because the system uses a single RGB-D camera, depth information is incomplete. To compensate, the obstacle bounding box is artificially extended in depth, which is a deliberate hack. A multi-camera setup or improved depth estimation would eliminate this workaround and enable more accurate obstacle modeling.