---
layout: post
title: "Windows Processes and Threads"
description: "Discussion about process and thread on Windows."
category: operatingsystem
tags: [Operating System, Thread, Process]
---

Multitasking in Operating System
================================

Software on a computer, basically, are a series of instructions that the CPU can
run. If we ignore multi-core CPU’s, how does your Operating System run multiple
programs at the same time so that you can listen to Spotify, browse Facebook
while writing your term paper if the programs are just a series of instructions?
This is the problem of multitasking, or scheduling in an Operating System.

In this blog post, I will discuss Windows’ Process and Thread, what they are and
how they work, as well as a brief comparison to their Linux counterparts.

CPU Scheduling
==============

Most modern Operating Systems perform preemptive multitasking. What that means
is that the scheduler can suspend a task/thread (more on this later) from
executing without the thread’s permission and let another task/thread execute.
The suspended thread will be scheduled to perform at a later time. This
scheduling act, performed quick enough, will give the illusion that multiple
programs are running at the same time, when in fact they are taking turn using
the CPU to execute their instructions. In Windows, the quantum/interval of
execution is from 10ms to 120ms, while on Linux it’s 10ms to 200ms.

Process
=======

On a Windows system, if you used the Task Manager tool, you likely have noticed
that the main tab is the “Processes” tab, and this seems to display the programs
running on your computer. You can usually think of a process as a program
running, except for some special cases (multiple processes for a software, or a
process hosting multiple “program”). On a deeper level, a process contains a
virtual address space, executable code, a handle table (containing information
about which system resource the process is using) and statistics about the
process. A process contains one or more threads, but cannot be empty. The first
thread in a process is called a *primary thread.*

Processes can communicate using several methods:

-   **Message passing**: Message passing can be done using Windows Message. This
    can be a default Windows Message (WM\_\*) or a custom message. Using a
    custom message requires both processes to know what the message is and what
    it contains. Message passing can also be done using Mailslot. Mailslot is a
    datagram broadcasted to multiple processes, locally or over a network, it is
    quite similar to a network datagram.

-   **Shared memory:** Another method of communication between processes is
    using shared memory. Shared memory can be in a few forms: File Mapping is a
    mechanism that lets a process treat a file as if it’s a part of the
    process’s virtual address space. 2 processes doing this to a file will be
    able to communicate through this file. Another form of shared memory is
    through the Windows Clipboard. The Clipboard is maintained by the Windows
    Kernel, and the processes communicating using the Clipboard only has to
    agree on what format the data is in. An abstraction over a shared memory
    communication is Pipe, which can be Anonymous Pipe, which is used for 1 way
    communication between parent and child processes, or Named Pipe, which
    allows 2-way communication between unrelated processes, locally or over
    networks.

-   **Remote Procedure Call:** RPC is another method of inter-process
    communication, this makes the communication between processes looks like a
    function call. Expanding on this is COM (Component Object Model), which lets
    processes share objects (DCOM – Distributed COM, allows sharing COM objects
    over networks as well).

-   **Socket:** And of course, processes can communicate using sockets, which
    allows communication locally and over networks.

Thread
======

A thread is a part of a Process, and is the basic scheduling unit in Windows.
The scheduler will pick threads from any process to run on the CPU at any time.

All threads in a process share the virtual address space and the handle table.
Each thread, however, has its own local storage and thread context. The thread
context contains its registers and stacks (kernel and user). The thread context
is what the system will save and load when a thread is being rescheduled.

A thread can start other threads and wait on other threads, maintaining a
parent/child relationship. Inter-thread communication is performed using typical
multithreading synchronization methods (mutex, locks, semaphores…).

A brief description of Linux counterparts
=========================================

Over in Linux land, the multitasking is done slightly differently. Linux only
has a single unit of execution, a task. However, task in Linux can be
independent of each other, not sharing address space or system resources (this
will make a task the equivalence of a Windows process with a single thread of
execution). Tasks can also share their address space and system resources, which
make them the equivalence of Windows threads in a process.

Wrapping up
===========

Above are the basics about Process and Thread in Windows, though the same
concepts will apply to Linux or any other modern Operating System, with some
different vocabularies and nuances. I wrote this blog post to provide some
background knowledge for my upcoming post(s) about debugger and how it works.

As always, I’d love to get feedbacks, so if I missed or wrote anything
incorrectly, please let me know!

Thai
