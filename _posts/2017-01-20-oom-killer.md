---
layout: post
title: "OOM Killer: Who are you!?!?"
description: ""
category: 
tags: [linux, process, memory]
comments: true
---

The OOM killer exists because the Linux kernel, by default, can commit to supplying more 
memory than it can actually provide.
Overcommitting memory and fearing the OOM killer are not necessary parts of the Linux 
experience, however. Simply setting the sysctl parameter **vm/overcommit\_memory** to 
2 turns off the overcommit behavior and keeps the OOM killer forever at bay.

But who is this oom killer? And how it works?

First of all we can do an example: let's consider the **fork()** system call, which copies all 
of a process's memory for the new child process. In fact, all it does is to mark the memory 
as _"copy on write"_ and allow parent and child to share it. Should either change a page shared 
in this way, a true copy is made. In theory, the kernel could be called upon to copy all of 
the copy-on-write memory in this way; in practice, that does not happen. 
If the kernel reserved all of the necessary virtual memory (which includes swap space), some 
of that space would certainly go unused. Rather than waste that space - and fail to run programs 
or memory allocations that, in practice, it could have handled - the kernel overcommits itself and 
hopes for the best.  

But how this is ruled?

I write this post just to _copy and paste_ this citation coming from [HERE](https://lwn.net/Articles/104185/),
that describe who is the oom killer and how it was designed :D

_"An aircraft company discovered that it was cheaper to fly its planes with less fuel on board. 
The planes would be lighter and use less fuel and money was saved.
On rare occasions however the amount of fuel was insufficient, and the plane would crash. 
This problem was solved by the engineers of the company by the development of a special OOF
(out-of-fuel) mechanism. In emergency cases a passenger was selected and thrown out of the plane. 
(When necessary, the procedure was repeated.) 
A large body of theory was developed and many publications were devoted to the problem 
of properly selecting the victim to be ejected. Should the victim be chosen at random? 
Or should one choose the heaviest person? Or the oldest? Should passengers pay in order 
not to be ejected, so that the victim would be the poorest on board? And if for example 
the heaviest person was chosen, should there be a special exception in case that was the 
pilot? Should first class passengers be exempted? Now that the OOF mechanism existed, 
it would be activated every now and then, and eject passengers even when there was no 
fuel shortage. The engineers are still studying precisely how this malfunction is caused."_
