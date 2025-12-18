---
title: "Team"
permalink: /team/
layout: single
author_profile: false
classes: wide
---

Using pure HTML gives you the most control over the layout. To get a clean 2x2 grid that doesn't feel cramped, we’ll use CSS Grid. This will automatically handle the spacing and ensure that on smaller screens (like phones), it gracefully snaps back to 1 column.

Here is the complete block you can paste directly into your .md file.

1. Set the Page Width
First, make sure your page allows for a wider layout by updating the top of your file:

YAML

---
title: "Meet the Team"
layout: single
classes: wide  # This expands the container width
---
2. The HTML & CSS Code
Copy and paste this below your front matter. I’ve included the "Mobile First" logic so it looks good on all devices.

HTML

<style>
  .team-grid {
    display: grid;
    grid-template-columns: 1fr; /* 1 column on mobile */
    gap: 40px;
    margin-top: 2em;
  }

  /* 2 columns on tablets and desktops */
  @media (min-width: 600px) {
    .team-grid {
      grid-template-columns: 1fr 1fr;
    }
  }

  .team-member {
    text-align: center;
    padding: 10px;
  }

  .team-member img {
    width: 200px;
    height: 200px;
    object-fit: cover;
    border-radius: 50%; /* Makes headshots circular */
    margin-bottom: 15px;
    border: 3px solid #f2f3f3; /* Optional: light border */
  }

  .team-member h3 {
    margin-bottom: 5px;
    font-size: 1.4em;
  }

  .team-member p {
    font-size: 0.9em;
    line-height: 1.5;
    color: #666;
  }
</style>

<div class="team-grid">
  <div class="team-member">
    <img src="/assets/images/kathy.jpg" alt="Kathy Min">
    <h3>Kathy Min</h3>
    <p>Kathy is a Mechanical Engineering PhD student in the Berkeley Robotics and Human Engineering Lab. She is interested in mechatronics and controls, particularly pertaining to wearable exoskeleton devices.</p>
  </div>

  <div class="team-member">
    <img src="/assets/images/ben.JPEG" alt="Ben Davis">
    <h3>Ben Davis</h3>
    <p>Ben is a Mechanical Engineering PhD student in the Embodied Dexterity Group under Prof. Hannah Stuart. His research interests are in mechanical design, sensing, and control for robotic manipulation.</p>
  </div>

  <div class="team-member">
    <img src="/assets/images/sharaf.jpg" alt="Sharaf Hossain">
    <h3>Sharaf Hossain</h3>
    <p>Sharaf is a MEng Mechanical Engineering student, specializing in Robotics and Control. He is currently working with Dr. Allen Y. Yang for his Capstone Project on Autonomous Race Cars, where he is working on real-time MPC. Sharaf's research interests are in robotics control systems.</p>
  </div>

  <div class="team-member">
    <img src="/assets/images/parham.jpg" alt="Parham Sharafoleslami">
    <h3>Parham Sharafoleslami</h3>
    <p>Parham is an MEng Mechanical Engineering student specializing in Robotics and Autonomous Systems. He is currently working with Dr. Allen Y. Yang on the Autonomous Race Car project. His research interests includes offline MPC and control systems, with applications in robotics and automation.</p>
  </div>
</div>
