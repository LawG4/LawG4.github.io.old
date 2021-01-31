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


#### Windows

Firstly navigate over to the latest version of the graphical installer [here](https://github.com/devkitPro/installer/releases). Click through the wizard and ensure that Wii development option is selected, you can also choose to install any other devkits. After this, just let the installer run and do its job.  
![Windows Installer](/assets/Blog/HelloWorldWii/installer.png)

#### Unix Like (macOS and Linux)

On Unix systems, all devkitPro packages are distributed via pacman. If you are on an Arch based system this should be fairly easy. For systems without pacman, devkitPro maintains their own version called *dkp-pacman*. Go to their [latest pacman release](https://github.com/devkitPro/pacman/releases/), scroll down to find a .deb file for Debian based systems, and a .pkg file for macOS.  
For systems like Ubuntu, you may need to install *gdebi-core*. Then run the gdebi command on the right .deb file for your architecture. On macOS this process is simplified to simply *right click > open*

Once pacman is installed to your system, we can use the following set of commands to download the wii-dev package group.  
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

On Linux, devkit installs a script to /etc/profile.d/devkit-env.sh, which will automatically set the necessary environment variables when you log in. These should be fine, but if you've installed to a custom location, then you will need to edit this file. After ensuring the environment variables are set correctly, log out and log back in to set your environment variables. On macOS you will need to reboot your device.  
 To ensure that this page always links to up to date information I will also include the links to devkit's [getting started](https://devkitpro.org/wiki/Getting_Started) page and their [pacman explanation](https://devkitpro.org/wiki/devkitPro_pacman).
## Building Examples
Navigate to the location that you installed devkitPro *(on Linux this should be /opt/devkitpro/)* and you'll find a folder called /examples/wii/. Copy the Wii examples folder to wherever you plan to be developing, open a terminal in your folder *(Windows users use ctrl + shift + right click > Open PowerShell here)* and enter the command: 
```console
make
```
Those familiar with Linux development will know this command well, however Windows developers might not be quite so sure. When calling a compiler command there are certain arguments that need to be passed, such as which libraries to link against. Make files automate that process, so that you don't have to specify command line arguments every single time. Usually make would not be a command availble on Windows, but the adition of the MySys environment allows for calling make. Note, on Windows, be careful which terminal emulator you use; I found gitbash had a conflicting version of cygwin.dll, but Powershell and Cmder both worked fine.   

Calling make in this location activates the makefile, which will loop through all the examples and build all of the Wii examples. Inside each example folder, you'll find a .dol and a .elf file. The .dol file is the raw binary file, the .elf file is the output of the linker.

 .elf files have debugging information and .dol files are best for distribution. Both file formats can be ran on the Wii, or more conviently the Dolphin emulator. 
![example](/assets/Blog/HelloWorldWii/Example.png)

To run these executables on an actual Wii, you will need a Wii with homebrew installed. Navigate to the homebrew menu and find your console's IP adress, then you can set the environment variable ***"WIILOAD"*** to *"tcp:IP_ADDRESS"*. From here we can use the wiiload tool to send and load the executable to the console. The tool is located at /devkitPro/tools/bin/wiiload, on Windows you can simply drag and drop the binary file onto wiiload.exe, Unix users can use the following command:
```console
wiiload EXECUTABLE COMAND_LINE_ARGS
```
[![ezgif-7-da98a6e46993.gif](https://s2.gifyu.com/images/ezgif-7-da98a6e46993.gif)](https://gifyu.com/image/UR1d)
## Hello, World!

Return back to writing here
