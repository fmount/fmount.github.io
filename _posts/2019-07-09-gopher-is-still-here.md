---
layout: post
title: "Gopher is still alive - Welcome to port 70"
description: ""
category:
read: 5
tags: [linux, rfc1436, networking, gopher, c]
comments: true
---

It's been a while since I've written my last post.<br>
Actually I was absent because in the meantime I've got a new job, I'm going to
make some family's changes (new house, new life), so all of my efforts were (and
are) focused on doing my best in my new job, handle all the things that the life 
reserved to me, and yes, it's been a quite hard find a moment to stop myself, 
thinking and writing.<br>
Finally I'm here, trying to find some time to write a little post on things I 
love and I really hope it can be a rule for the next months.<br>
Even though I love what I do day by day, there are few side projects I think 
they deserved to be told and yes, this post is about one of them.<br>
To make a long story short, few months ago reading how the [Redis](http://antirez.com/news/127)
development was going I was inspired by a new, old topic: the implementation of 
the **gopher protocol**.<br>
It's not a joke, and I know today gopher is not widely used anymore (HTTP rules), 
but I've seen in it an opportunity to learn a bit of history and why not, the 
opportunity to make a pure C implementation of a Gopher server to revive it's 
[RFC1436](https://tools.ietf.org/html/rfc1436) and figure out how internet (the 
internet people know today) was at the very beginning, when the need was to empower
people to publish and share information, getting rid of all it was related 
to the style and the cosmetic issues on presenting information to the end users.<br>
Anyway, [c_gopherd](https://github.com/fmount/c_gopherd) tries to be a tiny, 
minimal re-implementation of a gopher server and tries to make alive again that 
world in which you just had text based documents, hyperlink connections to other 
resources, experimenting a different way to do things.<br>
If you want to have more context on this, I suggest reading few useful documents:

1. [A bit of history around Gopher](https://mncomputinghistory.com/gopher-protocol)
2. [The Gopher protocol in brief](https://dev.to/dotcomboom/the-gopher-protocol-in-brief-1d88)
3. [Would you like to build your own Gopher{hole,space}?](https://sdf.org/?tutorials/gopher)


The c_gopherd project
---

You can simply build it using the provided Makefile.<br>
According to the defined _$PREFIX_, the default _all_ target generates a bin/ 
directory containing the result of the compilation.<br>
The _config_ target is useful to generate a _lib/defaults.h_ with all the user
defined parameters.

    $ make help

will show all the available targets.

    $ make config

will generate the defaults.h according to the defaults.def.h

    $ make

will compile and put the result in $(PREFIX) dir.


How can use it
---

    $ ./c_gopherd -h
    Usage ./bin/c_gopherd [-h host] [-p port] [-s srv_path] [-v]

More context on the project structure
---

In the /example project dir there is a little [Gophermap](https://github.com/fmount/c_gopherd/blob/master/example/gophermap)
file.<br>
Customizing the **GOPHER_ROOT** directory (aka *GROOT*) or passing the `-s path`
option, you can serve it as an *HTML equivalent index*.<br>
In this way you can display your custom Gopher menu instead of just a directory
listing.<br>
A good approach enabled by the use of the gophermap is represented by the ability
to create nested [gopherholes](https://gopher.zone/posts/tutorial-for-absolute-beginners)
indexed with their own gophermap in a structure like the following:

    example/
      gophermap
      user1/
        gophermap
        user_1_content/
      user2
        gophermap
        user_2_content/
      content1/
      content2.txt
      ...
      ...


How the gophermap is built?
---

According to the RFC and the evolution of the protocol, the following table specifies
the format and the content types allowed.<br>
The technical specification for Gopher, RFC 1436, defines 14 item types.
A one-character code indicates what kind of content the client should expect.<br>
Item type 3 is an error code for exception handling. Gopher client authors improvised
item types h (HTML), i (informational message), and s (sound file) after the
publication of RFC 1436, and I think the informational message is very useful
to organize better a gophermap and present the resources in a cleaner way.<br>

| Type   |      Description      |
|:----------:|:-------------|
| 0 | text file |
| 1 | Gopher submenu   |
| 2 | CCSO Nameserver |
| 3 | Error code returned by a Gopher server to indicate failure |
| 4 | BinHex-encoded file (primarily for Macintosh computers)   |
| 5 | DOS Files |
| 6 | uuencoded file |
| 7 | Gopher full-text search   |
| 8 | Telnet |
| 9 | Binary file |
| + | Mirror or alternate server (for load balancing or in case of primary server downtime)   |
| g | GIF file |
| I | Image file |
| T | Telnet 3270 |
| i | informational message |
| h | HTML file |
| s | Sound file |

An example of gophermap can be found [here](https://github.com/fmount/c_gopherd/tree/master/example).


Conclusion
---

I really enjoyed writing the first version of this server and I hope I can continue
soon implementing my [todo list](https://github.com/fmount/c_gopherd/blob/master/TODO)
items, but one of the most important things is that this project has given me the opportunity
to work again in C, learning about a very important piece of our history and share it with you.<br>
So, have fun with Gopher, and **welcome to port 70!** :D
