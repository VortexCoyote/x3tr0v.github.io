---
layout: post
img: thumbnail_encore.png
title: Encore
description: An editor for osu!mania
onhome: true
---

# An editor for osu!mania

## Screenshot
![](../assets/img/encore_1.png)

## Details:
- 5 Weeks Half-Time
- Written in C++ within openframeworks, with the ImGui and BASS library
- Had prepared a small engine for parsing beatmaps and rendering them before the start of the specialization course

## Goal and Purpose
The goal of my specialization course at The Game Assembly write an editor to a game called [osu!](https://osu.ppy.sh/), and for a specific gamemode within that game called [osu!mania](https://osu.ppy.sh/help/wiki/Game_Modes/osu!mania). The purpose of this project was to get more experience writing a tool for a specific game, while encountering new and unique problems along the way.

## What is osu!mania?
osu!mania is a so called "Vertically Scrolling Rhythm Game" (VSRG), which is loosely based on games such as Dance Dance Revolution, IIDX and Guitar Hero. The point of the game, is to press buttons to their respective columns, where notes fly downwards (or upwards depending on the game) to the rhythm of the song. Score is given based on how well you time your button presses with the notes positioning in relation to the "hit receptor", more info [here](https://osu.ppy.sh/help/wiki/Game_Modes/osu!mania). Those files that describes the levels layout to the song are called "beatmaps", or in many other cases "charts".

## Some interesting problems 
So right away, there are a few interesting problems to solve:
- Rendering notes that are on screen from a big data set
- Handling "beatsnap" lines
- Rendering the waveform efficiently 
- The Editing and placement of "BPM Points"


## Resources
