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

<img style="float: left; padding-right: 10px;" src="/assets/Blog/Decompilation/Thumb.png">This article covers a brief introduction to decompaction and hardware. Advanced readers can skip right to the [Gameboy Cartridge specifics](/blog/Decompilation/#gameboy-cartridge-specifics). Kirby's dream land for the Gameboy, do you know when this game came out? 1992! That makes this game multiple years my senior, but lately I've found myself drawn to retro tech. I mean look at Kirby, he wasn't even pink yet. I believe this recent interest in retro tech comes from a reasonable amount of nostalgia, plus a desire to work as close to bare metal as possible. 

While working on Scuffedcraft, I have gained a massive amount of respect for those who reverse engineer locked down hardware, How do you even begin to figure out a GPU without any documentation? So I wanted to dip my toes into reverse engineering. A Gameboy game would be perfect, as the instructions directly interface with the hardware including memory mapped IO, the Rom sizes are also very small which gives me fewer instructions to slog through.

But Why Kirby's dream land adventure? Well my brother gifted me a Gameboy Colour, Pokemon and Kirby's dreamland. So despite never actually beating the game before I lost the cartridge, I have a reasonble amount of nostalgia for the game. Plus decompiling Pokemon red or blue has already been tackled. Introduction over

## What is machine code and what does decompiling mean?

In order to understand decompiling, you first need to understand compiling. When we write code it is in human readable language, however computers cannot understand human language. Instead computers limit what they can process to just a few simple instructions. The list of instructions a CPU can support changes based on the architecture, and is called the instruction set.

Each instruction a CPU supports is assigned a unique number, this is called the operation code, or OP code for short. Data can also be supplied to the instruction, such as what number to add, this is called an operand. An instruction is made up of its OP code and appropriate operands. These instructions are stored in memory and in order. Meaning that we can refer to an an instruction by its address in memory.  

 This means that the CPU can collect an instruction from memory, decode what its numerical value means, and then run that command; this is commonly referred to as the fetch decode execute cycle. Compiling is the process of turning human readable code into a set of instructions that a CPU can process, otherwise known as machine code. Decompiling is the process of turning that machine code back into some kind of human readable code.

## Structure Of  The ROM 

### Hexadecimal

As mentioned earlier the OP code is represented as a number, the operands are data and as such also represented as numbers, these numbers are stored directly in binary. The binary is aligned into groups of 8 bits, otherwise known as a byte. This means when we ask the ROM chip for the data stored at address 0x01, we will get the second byte of data in the rom check.

The notation I used above is called *hexadecimal,* which is where numbers range from 0-F. Two hex numbers represent a single byte of data. Data on the ROM is 0 indexed. Meaning we start counting at 0, think of it as how many bytes we are offset from the start of the ROM chip.  The ROM (Read Only Memory)

### Rom Chip 

How does a ROM chip actually output data? Well there's a couple things it needs. All chips need power and ground, the rom then takes an address line as input, and a data line as output. There are other pins like the clock signal, and the chip enable pin. The CPU can set the address of the byte it wants to receive using binary, it physically sets the wires to have a high or a low voltage representing a 0 and a 1 respectively. The ROM chip will (after a very small delay) output the byte stored at that address.  

![Cartridge](/assets/Blog/Decompilation/SimpleRomChip.png)*Disc: Very over simplified*

In the above image you can see that the CPU is requesting the byte at 0xA2 which is the 162nd byte into the ROM, which in this example happened to be 0x66. Note that this is an absurdly simplified example, Ben Eater does some excellent videos on this if you want some more information. The important takeaway is that ***the entire game can be found by requesting one byte of the machine code at a time***. One thing to note is that the data in the game (such as sprites) and the machine code instructions have to share the same address space. The CPU cannot determine if the contents of the data lines are data or code; if we're not careful we could end up interpreting sprites as instructions, and this is actually the source of numerous famous gameboy glitches.

### Gameboy Cartridge Specifics

It's at this point in the article I am realising that I have yet to accomplish any decomplication, so unfortunately things will start to get a bit more technical. The Gameboy has a 16bit address space, meaning that it can only address bytes from 0x0000 - 0xFFFF. Considering that the Gameboy has Memory mapped IO (Which means all the hardware has to share the same address space) there is not much left over for our code.    

![Cartridge](/assets/Blog/Decompilation/CartLabled.png)*Source: [https://imgur.com/user/GameboyMicro]( https://imgur.com/user/GameboyMicro)*

To this end most cartridges have a MBC (Memory Bank Controller) Which will let the Gameboy split the ROM into separate sections. This means the CPU could request to Read 0x100 twice, and get a different result depending on which bank is selected. You can actually see that the MBC traces connect to some of the address pins on the ROM. I'm am honestly not sure how this will effect the layout of the ROM contents.

However the whole ROM can be accessed by carefully setting the right flags to the MBC one at a time, we can get the ROM data in the same order as it was stored on the ROM chip, there are tools to do this automatically, and the process is called ROM dumping

### Interrogating  Values Of  The ROM

Now that you have the contents of the ROM file dumped, we can use a hex dumping tool to read the binary data at specific locations; this along with Nintendo's documentation will be our key to reverse engineering. 

The Nintendo documentation specifies that the catridge header is stored at address range [0x100 - 0x14FF], this mainly contains meta data about the game, such as the game's name. However, the byte at 0x147 contains the cartridge type.

```bash
# xxd Hex dumping tool
# -s Offset to start reading from
# -l How many bytes to read
xxd -s 0x147 -l 1 kirby.gb
# Returns 0x01
```

 For Kirby's dream land, we get byte 0x01, which we can look up in a table to find that it corresponds to a cartridge with just an MBC type 1 and a ROM. Keep this in mind, we'll need this information for later. 

The last useful thing in the cartridge header is the initial instruction, These are the three bytes [0x100 - 0x103]. After the Gameboy's internal boot ROM has finished executing, the ***PC*** (Program Counter) is set to 0x100, meaning this is the first bytes of code ran from the cartridge that the developer has control over. It is almost always: *0x00 0xc3 0x50 0x01* The first byte is a NO OP, meaning the CPU does nothing and increases the PC. 0xc3 is the next OP code and it means to jump to the address in the two operands, On this CPU the low byte is ordered first, so these 3 bytes translate into *"Jump to address 0x150"* i.e. start executing the code after the cartridge header.

```bash
xxd -s 0x100 -l 4 kirby.gb
# Returns:
# 00c3 5001
```

Using the hex dump tool, we can see that Kirby's dream land is the same. However, we won't get far interrogating this ROM one byte at a time. Let's start automating this process a little.

