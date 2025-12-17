---
title: "Design"
permalink: /design/
layout: single
---

## System Design and Task Goals

Our goal for this project was to design a system that could successfully parse a scene with multiple objects and an obstacle and move all objects to their respective destination without colliding with the obstacle. For simplicity, we used small red and black cubes as our objects, and we use a blue rectangular box as the obstacle. Colored box outlines were placed on the table to mark destinations. Since our perception stack was focused on object and obstacle classification and localization, we opted to hard-code these destinations.

{% include figure image_path="/assets/media/task_design.JPEG" width="50%" caption="Our setup involves multiple red and black objects, a blue obstacle, and two place destinations." %}

To accomplish this task, we designed our system in two modules: perception and planning/control. We also separated the task into two phases: reaching and moving. In the reaching phase, the robot must extract object locations and classes from the RGB-D image, select one object to pick, and reach that object using MPC or plain IK. In the moving phase, the held object is already classified, so the destination is already known. The robot only needs to localize the obstacle and plan and execute a trajectory around it.

{% include figure image_path="/assets/media/system_diagram.png" width="70%" caption="Our system can be understood in terms of reaching and moving phases, with distinct perception and planning/control blocks within both phases." %}

## Design: Perception



## Design: Planning and Control