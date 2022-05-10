---
layout: default
title: Finger Counter
parent: Software Engineering
nav_order: 2
---

# Finger Counter
{: .fs-9 .no_toc}

An application that uses Computer Vision to interpret the number of fingers a person is displaying with their hand.
{: .fs-6 .fw-300 }

[Get started now](#project){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 } [View it on GitHub](https://github.com/venGaza/fingerCounter){: .btn .fs-5 .mb-4 .mb-md-0 }

---

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>
  
## Project

Computer vision is a scientific field that allows the machine to develop an understanding of an image or video. With the use of openCV and Google's MediaPipe library, you can easily integrate computer vision capabilities into various projects. To demonstrate this idea, I built the FingerCounter project which allows users to leverage a neural network to interpret basic hand gestures for the numbers between 0 and 5.

## Demo

<p align="center"><img src="assets/finger-counter.gif?raw=true" width="70%" height="70%"></p>
  
## Project Details

### Language 
- Python

### Primary Modules
- openCV
- MediaPipe

### Design Thoughts
- MediaPipe is blazingly fast even running on just the CPU. 
- Implementing the algorithm for counting fingers seemed easy at first, but I realized how complex it could easily become. This program currently only succeeds at counting the fingers when the user uses their right hand with palm facing the camera. This limitation arises from the algorithm to distinguish if the thumb is extended or not.  One potential way around this is to add the capability to detect if the palm is facing the camera and then determining which hand is being evaluated based on the orientation of the thumb. (Sorry to my left-handed peers but you'll have to use the right hand to test this program for now)
- Overlaying a transparent PNG file over the openCV image took some time to understand because I did not have a true understanding of how the various image formats are shaped differently (in this case a 200x200x4 and 600,480,3).
  
### Potential Use Cases
- Interpreting sign language gestures
- Control system such as ordering at a restaurant or commanding a drone
- AR/VR 