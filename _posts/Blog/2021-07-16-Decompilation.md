---
title: "Getting Started In Decompilation"
date: 2021-06-16T15:34:30-04:00

header:
  teaser: /assets/Blog/Decompilation/Teaser.png
  og_image: /assets/Blog/Decompilation/Thumb.png
  image: /assets/Blog/Decompilation/Thumb.png

categories:
  - Blog
tags:
  - Alternative Hardware
  - C
  - Homebrew
  - Low Level
  - Machine Code
  - Assembly
  - Compiler
  - Decompilation
  - GameBoy
---

<img style="float: left; padding-right: 10px;" src="/assets/Blog/Decompilation/Thumb.png">Kirby's dream land for the Gameboy, do you know when this game came out? 1992! That makes this game multiple years my senior, but lately I've found myself drawn to retro tech. I mean look at Kirby, he wasn't even pink yet. I believe this recent interest in retro tech comes from a reasonable amount of nostalgia, plus a desire to work as close to bare metal as possible. 

While working on Scuffedcraft, I have gained a massive amount of respect for those who reverse engineer locked down hardware, How do you even begin to figure out a GPU without any documentation? So I wanted to dip my toes into reverse engineering. A Gameboy game would be perfect, as the instructions directly interface with the hardware including memory mapped IO, the Rom sizes are also very small which gives me fewer instructions to slog through.

But Why Kirby's dream land adventure? Well my brother gifted me a Gameboy Colour, Pokemon and Kirby's dreamland. So despite never actually beating the game before I lost the cartridge, I have a reasonble amount of nostalgia for the game. Plus decompiling Pokemon red or blue has already been tackled. Introduction over

## What Is Decompiling?
