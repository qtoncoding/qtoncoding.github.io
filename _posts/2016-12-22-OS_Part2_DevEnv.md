---
layout: post
title: "Operating System From Scratch. Part 2: Development Environment"
description: "The tools we'll be using in this project"
category: cplusplus
tags: [C++, C, GCC, QEmu, NASM, WSL, Linux, VS]
---
My development environment
==========================

For this project, I’ll be using NASM as the assembler, GCC for compiling higher
level codes (C, C++) and QEMU as the test machine. Alternatively, you can use
your favorite assembler, compiler (provided it can do standalone compiling) and
emulation software (VirtualBox, VMWare, Hyper-V, etc.).

I’ll be doing development on Windows 10, with the help of Bash on Ubuntu on
Windows (or Windows Subsystem for Linux, WSL) and my primary code editor will be
Visual Studio Code. This is largely because I don’t use Vim or any other command
line editor, so if you prefer that route, there isn’t much difference.

Getting the required software
=============================

The first step is to install Bash on Windows. The installation process and be
found [here on MSDN](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide). Alternatively, you can do this on MinGW, or just straight up Linux.

Next up is to get NASM, which, from Bash, can be installed with just
\`\`\`apt-get install NASM\`\`\`. If you want to do this purely on Windows, it
is also available [here](http://www.nasm.us/pub/nasm/releasebuilds/?C=M;O=D). You’ll need to add NASM to the PATH system variable to
use it from the command line, though.

The emulation environment I’ll be using is QEmu, which can be grabbed [here](http://wiki.qemu.org/Download).
Again, if you’re on windows, you’ll need to add it to PATH. If you opt to use
other virtualization software, you’ll have to go through the process of creating
a disk image for these programs to load. No big deal.

For compiling higher level code, which we’ll eventually move into because
writing a kernel in Assembly is going to be terribly tedious, if not extremely
difficult and time consuming. For this we’ll use GCC. Normally you’ll need to
modify GCC to be a cross compiler (which I think we’ll eventually have to do),
but since we’ll be targeting the 32-bit x86 architecture (specifically i386), if
you are on a 64-bit machine, you can just add a parameter to tell GCC to emit
32-bit code. GCC can be installed by \`\`\`apt-get install
build-essential\`\`\`. This will include several other tools we’ll use later on
(Make).

Finally, install your favorite code editor, be it Vim, Emacs or Sublime.
Personally I’ll be using Visual Studio Code, which can be downloaded from [here](http://code.visualstudio.com/download).

Now we have the tools needed for the job, next post will go into how to write a
simplistic boot sector.

Cheers,

Thai
