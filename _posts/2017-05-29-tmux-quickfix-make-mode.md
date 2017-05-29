---
layout: post
title: "The tmux quickfix plugin: Make mode feature"
description: ""
category: tmux
tags: [linux, tmux, make]
comments: True
---


Make mode is the new feature came in this plugin on commit [a96ed3c84c](https://github.com/fmount/tmux-quickfix/commit/a96ed3c84ccce81c427a77e1968f3aa0dad030ce). <br>
It's an extension of the previous features, but is also built on two main components:

1. **Project**: it represents the target dir on which we can execute commands; to make this variable
   available on the system, it has been registered in the metadata environment (defined locally for each session) as **@quickfix-project**

2. **Make command**: as described for the project variable, this is just the command to be executed
   on project target dir. It is defined as **@quickfix-make** and users can change its value according
   to the kind of project they're working on.

This is a new mode of work and as usual, metatada are setted up and queried on every new execution. <br>
Modifying these values:

    tmux set-option @quickfix-project "/target/dir/"

and

    tmux set-option @quickfix-make "make command"

users can obtain the desidered behaviour related to the category of their project. </br>
An example is show by the following video.


[![asciicast](https://asciinema.org/a/bn8holc9f3ic21k8f83yppqm5.png)](https://asciinema.org/a/arg8rk97mptlp6qlkoz3zkf13?autoplay=1)

