
---
layout: post
title: "Operating System From Scratch. Part 1: Introduction"
description: "The journey to learn how to make an OS starts!"
category: operatingsystem
tags: [assembly, C, C++, OS]
---

Why?
====

Operating System has always been the topic that captures my interest most in
Computer Science. I’ve been tinkering with computers for a long time now, and I
have always wanted to know what happens under the hood and how things work
together to provide this rich environment that enables us various activities
that we now take for granted. A little more farfetched reason is I’ll be joining
Microsoft’s Windows and Device Group after finishing school, so I’d like to arm
myself with knowledge of how an Operating System works from the bottom up.

This will be a series of blog posts, documenting my learning process of building
an OS from bottom up. I don’t know where this will end up, but it’s going to be
a learning experience and I want to share the things I find with you guys.

 Crash course on computer boot process
======================================

The first thing we need to concern ourselves with is the overall process that
happens when the computer is powered up. This will be the roadmap for us to
follow. Generally, the process is as below:

1.  Computer is powered on performs **P**owered-**O**n-**S**elf-**T**est ([POST](https://en.wikipedia.org/wiki/Power-on_self-test)).
    If this process is successful, usually you’ll hear a \*beep\*, but laptops
    don’t usually do this anymore, so unless you have a desktop PC, you won’t
    hear it. (On the other hand, if anything bad happens during this test,
    you’ll hear several \*beep\*s and if you’re on a desktop PC, there will be
    an array of light on the mainboard to tell you what went wrong)

2.  [BIOS](https://en.wikipedia.org/wiki/BIOS) is loaded and initializes the CPU and some basic I/O like keyboard and
    monitor. (Remember the blue screen that you get when you furiously hammer
    that Delete or F12 button when your computer starts up? Nowadays computers
    use UEFI, which provides many more functionalities like graphics, mouse even
    touch screen)

    BIOS is a piece of software that provides basic interfaces to control the
    hardware of the computer. This is usually installed on a ROM chip on the
    mainboard.

3.  BIOS will look at available drives for a magic number (0x55 and 0xAA at
    offset 510 and 511 respectively) to determine if that drive is bootable and
    load the first sector as the boot sector (usually 512 bytes).

4.  This boot sector will have several responsibilities:

    1.  Choose a partition to boot from (hard drive) or in the case of floppy
        disk, which drive.

    2.  Load the kernel on that partition/disk into memory.

    3.  Enter 32-bit Protected Mode.

    4.  Setup the environment for the kernel to work in.

    5.  Starts the kernel.

What we will cover
===================

The following is a list of things I’ve learned to do in the last few days and
I’ll document them in subsequent posts:

1.  Setting up development environment.

2.  First boot sector.

3.  Some useful utilities in Real Mode.

4.  Disk drive model and how to load memory from disk in Real Mode.

5.  Switch to Protected Mode.

6.  First simple kernel in Protected Mode.

I hope you will accompany me in this journey to learn about Operating System and
computers under the hood. I for one is very excited to dig into this topic and
share it with you.

Cheers,

Thai
