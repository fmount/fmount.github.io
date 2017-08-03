---
layout: post
title: "Dockerizing Taskwarrior"
description: ""
category: fun
tags: [raspberry, task, fun]
comments: True
---

In my path towards dockerizing everything in my laptop, I wrote another Dockerfile
to build my taskd-server container service.
And yeah, if you don't know what this little jewel is, Taskwarrior is just your todo list
manager, flexible, fast, efficient; it captures tasks, shows you the list and removes tasks
from that list, but it also has many great features, for example it could become a sophisticated
data query tool that can help you stay organized, and get through your work.<br>
As I said taskw has several strengths and the one I like very much is the ability to organize 
tasks in a **user-defined** hierarchy (by project, by date, by priority); when your list become 
so long, I think you can appreciate the query system, so the good news is that you can write
your own functions (combining the task-cli commands) to display things as you prefer, for instance 
this little peace of code show how you can combine the *ls* and the _summary_ commands to display tasks
by project name:

```sh

if [[ -z "$1" ]]; then
    task list && task summary
else
    task list project:$1 && task summary project:$1
fi

```

But now it's time to talk about the server side level.<br>
First of all, you can find the Dockerfile on my [github](https://github.com/fmount/dockerized/tree/master/taskd-server)
but you can just skip this check pulling the container from the dockerhub:
~~~
$ docker pull fmount/taskd
~~~
The default image generates a container that runs the server on localhost:53589.<br>
In order to expose the service, you need to modify the taskd server conf and run it
on **0.0.0.0:53589**, so the conf definitively looks like:

    confirmation=1
    extensions=/usr/local/libexec/taskd
    ip.log=on
    log=/tmp/taskd.log
    pid.file=/tmp/taskd.pid
    queue.size=10
    request.limit=1048576
    root=/var/taskd
    server=0.0.0.0:53589
    trust=strict
    verbose=1
    client.cert=/var/taskd/client.cert.pem
    client.key=/var/taskd/client.key.pem
    server.cert=/var/taskd/server.cert.pem
    server.key=/var/taskd/server.key.pem
    server.crl=/var/taskd/server.crl.pem
    ca.cert=/var/taskd/ca.cert.pem


The image also stores configuration and data in **/var/taskd**, which you should persist somewhere,
so you can run it mounting your own data to the taskd server:

	$ docker run --name "the_magical_tomato" -p 53589:53589 \
		-v /home/user/taskd-srv:/var/taskd:Z fmount/taskd:1.2.0

or you can start the docker container without any volume:

    $ docker run --name "the_magical_tomato" -p 53589:53589 fmount/taskd:1.2.0

and then copy all client conf from the /var/taskd:

    $ docker cp the_magical_tomato:/var/taskd/client.cert.pem
    $ docker cp the_magical_tomato:/var/taskd/client.key.pem
    $ docker cp the_magical_tomato:/var/taskd/ca.cert.pem

But ... before copying the file **client\*** and **ca.cert** mentioned above, you need first to create a tenant and the
related user(s) to sync from the server, so you need to attach to the container:

	$ docker exec -it the_magical_tomato /bin/bash

and add a Group/user:

	$ taskd add org MyOrganization
	$ taskd add user 'MyOrganization' 'myuser'

Finally, just generate the correct keys for the user client, so going to:

	$ cd /opt/taskd/pki

generate the **client.\*** launching:

	$ ./generate.client 'myuser'

This will generate a new key and cert, named **myuser.cert.pem** and **myuser.key.pem**. It is not important 
that 'myuser' was used here, just that it is something unique, and valid for use in a file name. 
It has no bearing on security.

Once the container is running and you see the exposed port (ss command is your friend), you can finally configure
your client adding to taskrc the following:

	taskd.certificate=/home/user/.task/client.cert.pem
	taskd.key=/home/user/.task/client.key.pem
	taskd.ca=/home/user/.task/ca.cert.pem
	taskd.server=server_address:53589
	taskd.credentials=Private/user/uuid
	
Here some useful doc I found to build the container and configure my clients:

* [https://taskwarrior.org/docs/taskserver/user.html](https://taskwarrior.org/docs/taskserver/user.html)
* [https://taskwarrior.org/docs/taskserver/taskwarrior.html](https://taskwarrior.org/docs/taskserver/taskwarrior.html)
* [https://taskwarrior.org/docs/taskserver/troubleshooting-sync.html](https://taskwarrior.org/docs/taskserver/troubleshooting-sync.html)


Finally, a last interesting note on this post is related to a tmux layout I made in order to show some infos
about the status of task activities; following this [GIST](https://gist.github.com/fmount/1bff0871e1982a87def2914a4c30b756)
you can find the tmux layout provided and, according to my last project [Tmux layout plugin](https://github.com/fmount/tmux-layout), 
you can load it putting it in the ~/.tmux/plugins/tmux-layout/layouts.
