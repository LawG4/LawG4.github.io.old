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


*(Excuse the terrible screen quality, I bought a cheap HDMI adapter)*
## Hello, World!

In the examples folder, you will find some source in a folder called "Template"; this is your standard *"Hello, world!"* example. This example is quite well documented already by the devkitPro team, but I would like to take some time to walkthough add some additional information.

Firstly *stdio* and *stdlib* are not the original version of the standard C libraries, but instead the ported ones supplied by devkitPPC. *gccore* stands for GameCube Core, this is the library that handles the primary hardware interaction. The Wii is essentially an overclocked Gamecube, so the GC libraries and games are both compatable with the Wii. *xfb* is a void pointer to the chunck of memory that we will allocate for the framebuffer. *rmode* is the video mode of the system, this represents things like the buffer size and the screen refresh rate.

```c
#include <stdio.h>
#include <stdlib.h>
#include <gccore.h>
#include <wiiuse/wpad.h>

static void *xfb = NULL;
static GXRModeObj *rmode = NULL;
```
Enter inside the main function, an interesting thing to note is that we can pass command line arguments. Next we enter into a series of functions supplied by *gccore* and *wpad*, the header files for which can be found inside the *libogc/include* folder. The implementation of these functions can be read on devkitPro's [github](https://github.com/devkitPro/libogc/tree/master/libogc). For example, *VIDEO_Init()* looks to set some video hardware registers using an internal variable called *"_viReg"*.  
The comments here are quite self explanatory, first start the video system and the plugged in controllers, Ask the Wii what video settings it is currently using; and finally, create a framebuffer matching those specifications.
```c
//---------------------------------------------------------------------------------
int main(int argc, char **argv) {
//---------------------------------------------------------------------------------
	// Initialise the video system
	VIDEO_Init();

	// This function initialises the attached controllers
	WPAD_Init();

	// Obtain the preferred video mode from the system
	// This will correspond to the settings in the Wii menu
	rmode = VIDEO_GetPreferredMode(NULL);

	// Allocate memory for the display in the uncached region
	xfb = MEM_K0_TO_K1(SYS_AllocateFramebuffer(rmode));
```
Next we set up the character console, so that we can print to it. Notice, *xfb* is just a pointer to the location of the framebuffer, the details of the framebuffer are stored separately in the video mode variable. After setting the registers required for this framebuffer and video mode, flush the register changes to the hardware. Wait for these updates to finish, this will be announced by enetering into the virtual blanking period.  
```c
	// Initialise the console, required for printf
	console_init(xfb,20,20,rmode->fbWidth,rmode->xfbHeight,
		rmode->fbWidth*VI_DISPLAY_PIX_SZ);

	// Set up the video registers with the chosen mode
	VIDEO_Configure(rmode);

	// Tell the video hardware where our display memory is
	VIDEO_SetNextFramebuffer(xfb);

	// Make the display visible
	VIDEO_SetBlack(FALSE);

	// Flush the video register changes to the hardware
	VIDEO_Flush();

	// Wait for Video setup to complete
	VIDEO_WaitVSync();
	if(rmode->viTVMode&VI_NON_INTERLACE) VIDEO_WaitVSync();
```
Now we can actually print to the console. *\x1b[* is an example of an escape code introducer. For example to use the escape code *2J* (to clear the console), the statement would be *"printf("\x1b[2J");"*.
```c
	// The console understands VT terminal escape codes
	// This positions the cursor on row 2, column 0
	// we can use variables for this with format codes too
	// e.g. printf ("\x1b[%d;%dH", row, column );
	printf("\x1b[2;0H");
	printf("Hello World!");
```
This while loop is similar to a windowing loop, it prevents the program from closing until the home button is pressed on a remote. *VIDEO_WaitVSync* will pause the current thread until the virtual blanking period has been entered; this is essentially waiting for the next frame before scanning the connected controllers.  
```c
	while(1) {
		// Call WPAD_ScanPads each loop, this reads the
		// latest controller states
		WPAD_ScanPads();

		// WPAD_ButtonsDown tells us which buttons were pressed in this 
		// loop this is a "one shot" state which will not fire again 
		// until the button has been released
		u32 pressed = WPAD_ButtonsDown(0);

		// We return to the launcher application via exit
		if ( pressed & WPAD_BUTTON_HOME ) exit(0);

		// Wait for the next frame
		VIDEO_WaitVSync();
	}
	return 0;
```

Finally, let's look at the last important component: The make file, which will be the basis for all your projects. Firstly, the file checks for the environment variable which points to the devkitPPC install. Then, include the instructions of building for the Wii.   
You can look into the wii_rules file, but in summary: wii_rules sets the locations for the default libraries, adds the gekko compiler to the path, sets the details for the Wii Cpu and finally set the output for .dol and .elf files.
```make
ifeq ($(strip $(DEVKITPPC)),)
$(error "Please set DEVKITPPC in your environment. export DEVKITPPC=
		<path to>devkitPPC")
endif
include $(DEVKITPPC)/wii_rules
```
Target automatically sets the name of the outputted files to be the name of the directory the make file is in. Build is the directory that the outputed .a and .o files that are built from the source, before being linked. Sources is the location of the actual source code files, data is the location of any extra assets and finally the Includes location is where any extra header files might be.
```make
TARGET		:=	$(notdir $(CURDIR))
BUILD		:=	build
SOURCES		:=	source
DATA		:=	data
INCLUDES	:=
```
The final section that you might find yourself customising is the library inputs to the linker. You'll notice that that some libraries are already added for us
- libWiiUse : Wii remote library.
- libbte : Bluetooth to connect to the Wiimotes. 
- libogc : Wii hardware interface library.
- libm : maths library.

Extra library directories can also be added, so that the linker knows where to look for external extra libraries. The rest of the make file is dedicated to automating the finding of all the source files included in the project.

```make
#---------------------------------------------------------------------------------
# any extra libraries we wish to link with the project
#---------------------------------------------------------------------------------
LIBS	:=	-lwiiuse -lbte -logc -lm

#---------------------------------------------------------------------------------
# list of directories containing libraries, this must be the top level containing
# include and lib
#---------------------------------------------------------------------------------
LIBDIRS	:=
``` 
So finally we have most of the knowledge to understand what we're doing when we call *"make"* inside of the *"Hello, world!* project.


![makefile](/assets/Blog/HelloWorldWii/make.png)


Finally the result of our labour will run on both Dolphin emulator and actual Nintendo hardware. 
![makefile](/assets/Blog/HelloWorldWii/HelloWorld.png)