Hi, in this post we're going to learn about [Docker](https://www.docker.com/){:target="_blank"} and containers.
We're going to install docker and work with the `hello world` example and learn some basic commands and terminology.


Installing Docker
=================
Installing `docker` is pretty straight-forward, although it may vary depending on which OS you're on. I suggest going [here](https://docs.docker.com/install/){:target="_blank"} and following the guide.


Hello World
===========
Let's check that we have `docker` running by checking the version:
{% highlight bash %}
~ » docker --version
Docker version 18.09.1, build 4c52b90
{% endhighlight %}
Of course, the version might change, but you should get some output.
Before going any further, let's check two basic concepts:

- Images

What's an image? An image is the file system and configuration for our app, which is used to create containers.

- Containers

What's a container? A container is a running instance of a `docker` image. They actually run the app. It contains an app and all of its dependencies.

Remember these two concepts, they are key to understanding images and containers.
Now let's get back on our terminal and run the `hello-world` example with `docker run hello-world`.

{% highlight bash %}
~ » docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
{% endhighlight %}

Your output should be similar to the one above, if that's so, it means your docker installation is working properly.
Read the output, it's actually very helpful, it lets you know what happened behind the scenes. `Docker daemon` actually did a `docker image pull` to download the image and then it did a `docker container run` to show you the `hello-world` output.

Try running `docker run hello-world` again. It should be faster since the `image` has already been pulled.


Pulling your first image
========================
Now that everything's setup, it's time to start hacking. Let's pull an `Ubuntu` image from the repos.

Run the following commmand `docker pull ubuntu`.
This will download the latest `image` from the repos.
{% highlight bash %}
~/docker [chuleh] » docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
Digest: sha256:7a47ccc3bbe8a451b500d2b53104868b46d60ee8f5b35a24b41a86077c650210
Status: Downloaded newer image for ubuntu:latest
{% endhighlight %}


Now type `docker image ls`, you should see the `hello-world` and the `Ubuntu` image.
{% highlight bash %}
~/docker [chuleh] » docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              47b19964fb50        2 weeks ago         88.1MB
hello-world         latest              fce289e99eb9        7 weeks ago         1.84kB
{% endhighlight %}


Creating your first container
=============================
Now we're going to run a container based on this image.
To do so, we'll use the command `docker container run <container> <command>`. Let's list the contents of our container with `docker container run ubuntu ls -l`.
{% highlight bash %}
~/docker [chuleh] » docker container run ubuntu ls -l
total 64
drwxr-xr-x   2 root root 4096 Feb  4 21:05 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  340 Feb 20 22:35 dev
drwxr-xr-x   1 root root 4096 Feb 20 22:35 etc
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   8 root root 4096 May 23  2017 lib
drwxr-xr-x   2 root root 4096 Feb  4 21:03 lib64
drwxr-xr-x   2 root root 4096 Feb  4 21:02 media
drwxr-xr-x   2 root root 4096 Feb  4 21:02 mnt
drwxr-xr-x   2 root root 4096 Feb  4 21:02 opt
dr-xr-xr-x 184 root root    0 Feb 20 22:35 proc
drwx------   2 root root 4096 Feb  4 21:04 root
drwxr-xr-x   1 root root 4096 Feb  6 03:37 run
drwxr-xr-x   1 root root 4096 Feb  6 03:37 sbin
drwxr-xr-x   2 root root 4096 Feb  4 21:02 srv
dr-xr-xr-x  13 root root    0 Feb 20 22:35 sys
drwxrwxrwt   2 root root 4096 Feb  4 21:05 tmp
drwxr-xr-x   1 root root 4096 Feb  4 21:02 usr
drwxr-xr-x   1 root root 4096 Feb  4 21:04 var
{% endhighlight %}
So when you ran the `run` command, believe it or not, a `container` was created. The `docker daemon` created the container and then ran the command inside of it, finally, the output is streamed through the `docker client`.

Let's try something, run the following command `docker container ls`. This will list all your active containers:
{% highlight bash %}
~/docker [chuleh] » docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
{% endhighlight %}
Ok, so what happened here? We created a container and listed the contents of it, however when we list the `containers` the output shows none.
This is not a bug, this is correct, the container ran with the command we gave it and then the process stopped.

Now let's try `docker container ls -a`.
This will show all of the containers we ran so far.
{% highlight bash %}
~/docker [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
f69327766694        ubuntu              "ls -l"             4 minutes ago       Exited (0) 4 minutes ago                       elegant_bohr
{% endhighlight %}
Notice the `STATUS` column, it shows the containers just exited minutes ago.


Run containers interactively
================================
We can also run commands inside our container from the container's `CLI`.
Try running `docker run -i -t ubuntu bash`, this will open an interactive shell session within our container.
{% highlight bash %}
~/docker [chuleh] » docker run -i -t ubuntu bash
root@d3cdbc126cd4:/#
{% endhighlight %}
Cool, now we can do lots of stuff inside our containers.
Let's create a `hello-world` file and exit.
{% highlight bash %}
root@d3cdbc126cd4:/# echo "hello world" > hello-world
root@d3cdbc126cd4:/# exit
exit
{% endhighlight %}


Name your container
==================
As you may have noticed when running `docker container ls`, all containers have some funny name. This name is generated randomly by `docker`.
We can name our container with the `--name` flag. I will name mine `ubuntu-test`.
{% highlight bash %}
~/docker [chuleh] » docker container run --name ubuntu-test ubuntu ls -l
total 64
drwxr-xr-x   2 root root 4096 Feb  4 21:05 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  340 Feb 20 23:09 dev
drwxr-xr-x   1 root root 4096 Feb 20 23:09 etc
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   8 root root 4096 May 23  2017 lib
drwxr-xr-x   2 root root 4096 Feb  4 21:03 lib64
drwxr-xr-x   2 root root 4096 Feb  4 21:02 media
drwxr-xr-x   2 root root 4096 Feb  4 21:02 mnt
drwxr-xr-x   2 root root 4096 Feb  4 21:02 opt
dr-xr-xr-x 183 root root    0 Feb 20 23:09 proc
drwx------   2 root root 4096 Feb  4 21:04 root
drwxr-xr-x   1 root root 4096 Feb  6 03:37 run
drwxr-xr-x   1 root root 4096 Feb  6 03:37 sbin
drwxr-xr-x   2 root root 4096 Feb  4 21:02 srv
dr-xr-xr-x  13 root root    0 Feb 20 22:35 sys
drwxrwxrwt   2 root root 4096 Feb  4 21:05 tmp
drwxr-xr-x   1 root root 4096 Feb  4 21:02 usr
drwxr-xr-x   1 root root 4096 Feb  4 21:04 var
{% endhighlight %}

And now let's check it with `docker container ls -a`
{% highlight bash %}
~/docker [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                          PORTS               NAMES
987ddbb75b46        ubuntu              "ls -l"             50 seconds ago      Exited (0) 49 seconds ago                           ubuntu-test
7dcca0aee2d3        ubuntu              "bash"              11 minutes ago      Exited (0) About a minute ago                       pensive_khorana
d3cdbc126cd4        ubuntu              "bash"              16 minutes ago      Exited (0) 13 minutes ago                           festive_robinson
f69327766694        ubuntu              "ls -l"             35 minutes ago      Exited (0) 35 minutes ago                           elegant_bohr
43fa84ae8b90        ubuntu              "/bin/bash"         36 minutes ago      Exited (0) 36 minutes ago                           modest_burnell
{% endhighlight %}
As you can see in the `NAMES` column, the name of the container has been changed.




That's it for today. Play around with `docker image` and `docker container`, they're your bread and butter and most likely the commands you will be using the most.

Stay tuned for in the next lesson we'll be covering `deletion` of images and containers.

:wq
