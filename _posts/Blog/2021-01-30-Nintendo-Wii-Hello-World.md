---
title: "Nintendo Wii: Hello world!"
date: 2021-01-29T15:34:30-04:00

header:
  teaser: /assets/Blog/HelloWorldWii/Teaser.png

categories:
  - Blog
tags:
  - Alternative Hardware
  - C
  - Homebrew
  - Devkit
  - Nintendo Wii
---
The Nintendo Wii has always been a favourite console of mine, and it has supplied me with countless hours of joy. Because of this, I want to join my passion for programming with this fantastic console. Thankfully there exists a toolchain called devkitPPC for programming this particular device. Today I would like to introduce you to those tools. This blog entry will be a traditional “Hello, world” style introduction to devkitPPC, taking you through the installation process and your first program. DevkitPPC contains the following: 
- A C and C++ compiler.
- libogc a library for interfacing with the hardware.
- A collection of ported libraries.
- Tools, such as a texture converter.

On the Windows installation an MySys environment is also included, this facilitates the use of shell scripts and make files.

## Installation Process
DevkitPro is the name of the organisation that supplies the toolchain. They produce development kits for a range of Nintendo consoles. We are specifically interested in devkitPPC, which is the development kit aimed at the Nintendo Wii platform.

Firstly navigate over to [the devkit wiki](https://devkitpro.org/wiki/Getting_Started). If you’re a Windows user, there will be a graphics installer available. Unix users might have to install via Pacman package manager. For Unix distributions that do not come installed with Pacman, there is an option to install dkp-pacman or the toolchain binaries directly. Detailed information for installing Pacman supplied [here](https://devkitpro.org/wiki/devkitPro_pacman).

![Windows Installer](/assets/Blog/HelloWorldWii/installer.png)

Once in the graphical installer, ensure that the Nintendo Wii option is selected and let the wizard do its thing. For Unix, the different platforms get placed into groups.  
Sync the Pacman database:
```console
sudo (dkp-)pacman -Sy
```
Install any updates:
```console
sudo (dkp-)pacman -Syu
```
Install the Wii devkit 
```console
sudo (dkp-)pacman -S wii-dev
```

## Building Examples
Navigate to the location that you installed devkitPro, and you'll find a folder called /examples/wii/. Open a terminal in this location, and enter the command: 
```console
make
```
Those familiar with Linux development will know this command well, however Windows developers might not be quite so sure. When calling a compiler command there are certain arguments that need to be passed, such as which libraries to link against. Make files automate that process, so that you don't have to specify command line arguments every single time. Usually make would not be a command availble on Windows, but the adition of the MySys environment allows for calling make.  

Calling make in this location activates the makefile, which will loop through all the examples and build all of the Wii examples. Inside each example folder, you'll find a .dol and a .elf file. The .dol file is the raw binary file, the .elf file is the output of the linker.

 .elf files have debugging information and .dol files are best for distribution. Both file formats can be ran on the Wii, or more conviently the Dolphin emulator. 
![example](/assets/Blog/HelloWorldWii/Example.png)

To run these executables on an actual Wii, you will need a Wii with homebrew installed. Navigate to the homebrew menu and find your console's IP adress, then you can set the environment variable ***"WIILOAD"*** to *"tcp:IP_ADDRESS"*. From here we can use the wiiload tool to send and load the executable to the console. The tool is located at /devkitPro/tools/bin/wiiload, on Windows you can simply drag and drop the binary file onto wiiload.exe, Unix users can use the following command:
```console
wiiload EXECUTABLE COMAND_LINE_ARGS
```

## Hello, World!

Return back to writing here
