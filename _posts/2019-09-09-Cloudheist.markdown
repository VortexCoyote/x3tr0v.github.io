---
layout: post
img: thumbnail_cloudheist.png
tite: Cloudheist
description: 3D Space Shooter
onhome: true
project: true
---
## Trailer
<style>.embed-container { position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; } .embed-container iframe, .embed-container object, .embed-container embed { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }</style><div class='embed-container'><iframe src='https://www.youtube.com/embed/O7j-lOPWf1A?list=PLtiFs_CcTAQ6oM7Gz_35faHwmmsGMf1tR' frameborder='0' allowfullscreen></iframe></div>

## Details
- 3D Space Shooter
- 10 Weeks Part-Time
- Made in our own DirectX11 based C++ engine

## Contributions
- **Component System** - The system that's the backbone of every entity in the game. Inspired by Unity's Component System with the exception that a GameObject can only have one instance of each component type. The Component System's interface was designed to be as user-friendly as possible. More info [here](https://www.andreaswinthersorman.com/Component-System/).

- **Spline-Based Player Movement** - I implemented both the spline and the spline-based player movement, which is similar to how Starfox does it. I tried to replicate the movement and feel from Starfox as closely as I could. Values for the player movement were exposed via an [ImGui](https://github.com/ocornut/imgui) window so that our level designers could find the right values for the movement. 

- **GameObject Debugger** - A tool to debug GameObject via clicking on a object in-game. Once a object is clicked, a ImGui window will open with a list of each component. Each component have the ability to display certain data that the creator of the component wish to expose to the debug system. The logic for displaying the component specific debug data was defined through a virtual function. 

- **Particle Editor** - An editor where you can both edit particle and emitter parameters, and save it to a .json file. 
![](../assets/img/particle_editor.png)
The editor was mainly made with [ImGui](https://github.com/ocornut/imgui).

- **Post-Processing** - Mostly shaders such as Godrays (based on [this article](https://developer.nvidia.com/gpugems/gpugems3/part-ii-light-and-shadows/chapter-13-volumetric-light-scattering-post-process)), Radial-Blur (blur amount based on the current player speed) and red vinette that appears based on the players health.


## Screenshots
![](../assets/img/cloudheist_01.png)
![](../assets/img/cloudheist_02.png)
![](../assets/img/cloudheist_03.png)
