---
title: "Results"
permalink: /results/
layout: single
---

Our system successfully integrates perception and model predictive control to perform pick-and-place sorting in a cluttered environment. Using the RGB-D perception pipeline, the robot reliably detected and classified multiple cubes of two colors (red and black) and localized a large obstacle positioned between the pickup region and the placement locations. Based on this perception output, the MPC planner generated feasible, collision-free joint-space trajectories that transported each cube from its detected location to the correct destination according to its color classification.

Across multiple trials, the robot was able to sort several objects sequentially without colliding with the obstacle, demonstrating reliable end-to-end performance of the perception, planning, and control pipeline.

Below is a video of our Robot Arm in action! Showcasing the fully working MPC and Vision Model.

<p>Sorting demo (on hardware):</p>
<video width="800" controls preload="metadata">
  <source src="/assets/media/final_robot_mpc.mp4 type="video/mp4">
  Your browser does not support the video tag.
</video>
