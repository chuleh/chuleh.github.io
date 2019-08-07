Hi, in this post we're continuing our series about `Docker`, if you're completely new to `Docker`, I suggest reading my [Intro to Docker]({% post_url 2019-02-20-intro-to-docker %}) where you will learn the very basics of `Docker`.

Today we will learn about deletion of images, we will create a new container, mess it up and spin a new one in a matter of seconds.

Starting a new container
========================
If you followed my first lesson, you should already have the `Ubuntu` image we pulled from the `Docker hub`. Keep it, cause we're going to use it now to create a new container.
We're going to run the new container interactively, we're going to `rm -rf /` it and then we'll start a new container.
Type `docker container run -i -t ubuntu bash`
{% highlight bash %}
~/docker [chuleh] » docker container run -i -t ubuntu bash
root@a058c5234e4b:/#
{% endhighlight %}

Now it's time to mess up the container. We're going to `rm -rf /` and delete all of its contents. *MAKE SURE YOU'RE IN THE CONTAINER* when typing this.
Type `rm -rf / --no-preserve-root` and you'll mess up your container. You'll see a lot of output while everything is being deleted.
Just to make sure, try typing a few commands, you'll see that they're not found.
{% highlight raw %}
root@a058c5234e4b:/# ls
bash: /bin/ls: No such file or directory
{% endhighlight %}

Good, now just exit the container with either `ctrl-d` or by typing `exit`.
Let's check our container exited with `docker container ls -a`.
{% highlight bash %}
~/docker [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
a058c5234e4b        ubuntu              "bash"              7 minutes ago       Exited (127) 7 seconds ago                       silly_lumiere
ad6759ec86c7        ubuntu              "/bin/bash"         9 minutes ago       Exited (0) 9 minutes ago                         musing_swanson
{% endhighlight %}

So far so good. Now let's run another container based on the same image.
Type `docker run container -i -t ubuntu bash`.  You should have a fresh new container working.
Type in `ls -la` and you should see everything ready to work again on your container.
{% highlight bash %}
~/docker [chuleh] » docker container run -i -t ubuntu bash
root@8a6fe93f040b:/# ls -la
total 72
drwxr-xr-x   1 root root 4096 Feb 21 15:32 .
drwxr-xr-x   1 root root 4096 Feb 21 15:32 ..
-rwxr-xr-x   1 root root    0 Feb 21 15:32 .dockerenv
drwxr-xr-x   2 root root 4096 Feb  4 21:05 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  360 Feb 21 15:32 dev
drwxr-xr-x   1 root root 4096 Feb 21 15:32 etc
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   8 root root 4096 May 23  2017 lib
drwxr-xr-x   2 root root 4096 Feb  4 21:03 lib64
drwxr-xr-x   2 root root 4096 Feb  4 21:02 media
drwxr-xr-x   2 root root 4096 Feb  4 21:02 mnt
drwxr-xr-x   2 root root 4096 Feb  4 21:02 opt
dr-xr-xr-x 183 root root    0 Feb 21 15:32 proc
drwx------   2 root root 4096 Feb  4 21:04 root
drwxr-xr-x   1 root root 4096 Feb  6 03:37 run
drwxr-xr-x   1 root root 4096 Feb  6 03:37 sbin
drwxr-xr-x   2 root root 4096 Feb  4 21:02 srv
dr-xr-xr-x  13 root root    0 Feb 21 15:24 sys
drwxrwxrwt   2 root root 4096 Feb  4 21:05 tmp
drwxr-xr-x   1 root root 4096 Feb  4 21:02 usr
drwxr-xr-x   1 root root 4096 Feb  4 21:04 var
{% endhighlight %}


Deleting containers
====================
Our containers take up some space, it's usually minimal, though. But they still do take some space so now it's time to delete them.
First let's check the space they're consuming with `docker container ls -a -s`.
{%highlight bash %}
~/docker [chuleh] » docker container ls -a -s
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES               SIZE
8a6fe93f040b        ubuntu              "bash"              8 minutes ago       Exited (0) 4 minutes ago                          amazing_pare        7B (virtual 88.1MB)
a058c5234e4b        ubuntu              "bash"              18 minutes ago      Exited (127) 10 minutes ago                       silly_lumiere       0B (virtual 88.1MB)
ad6759ec86c7        ubuntu              "/bin/bash"         20 minutes ago      Exited (0) 20 minutes ago                         musing_swanson      0B (virtual 88.1MB)
{% endhighlight %}

So I've got 3 containers stopped. I will delete the first one with `docker container rm`. The command can take either the `container id` or the `container name`. So we will delete it and then check again if the container has been deleted with `docker container ls -a -s`.
{% highlight bash %}
~/docker [chuleh] » docker container rm 8a6fe93f040b
8a6fe93f040b
~/docker [chuleh] » docker container ls -a -s
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES               SIZE
a058c5234e4b        ubuntu              "bash"              23 minutes ago      Exited (127) 16 minutes ago                       silly_lumiere       0B (virtual 88.1MB)
ad6759ec86c7        ubuntu              "/bin/bash"         25 minutes ago      Exited (0) 25 minutes ago                         musing_swanson      0B (virtual 88.1MB)
{% endhighlight %}

You can also run containers with auto deletion adding the `--rm` flag so they will be automatically removed after they exit.
Type in `docker container run --rm -i -t ubuntu bash` and type some command.
{%highlight bash %}
~/docker [chuleh] » docker container run --rm -i -t ubuntu bash
root@d3c6aad1b015:/# ls -la
total 72
drwxr-xr-x   1 root root 4096 Feb 21 16:07 .
drwxr-xr-x   1 root root 4096 Feb 21 16:07 ..
-rwxr-xr-x   1 root root    0 Feb 21 16:07 .dockerenv
drwxr-xr-x   2 root root 4096 Feb  4 21:05 bin
drwxr-xr-x   2 root root 4096 Apr 24  2018 boot
drwxr-xr-x   5 root root  360 Feb 21 16:07 dev
drwxr-xr-x   1 root root 4096 Feb 21 16:07 etc
drwxr-xr-x   2 root root 4096 Apr 24  2018 home
drwxr-xr-x   8 root root 4096 May 23  2017 lib
drwxr-xr-x   2 root root 4096 Feb  4 21:03 lib64
drwxr-xr-x   2 root root 4096 Feb  4 21:02 media
drwxr-xr-x   2 root root 4096 Feb  4 21:02 mnt
drwxr-xr-x   2 root root 4096 Feb  4 21:02 opt
dr-xr-xr-x 181 root root    0 Feb 21 16:07 proc
drwx------   2 root root 4096 Feb  4 21:04 root
drwxr-xr-x   1 root root 4096 Feb  6 03:37 run
drwxr-xr-x   1 root root 4096 Feb  6 03:37 sbin
drwxr-xr-x   2 root root 4096 Feb  4 21:02 srv
dr-xr-xr-x  13 root root    0 Feb 21 15:24 sys
drwxrwxrwt   2 root root 4096 Feb  4 21:05 tmp
drwxr-xr-x   1 root root 4096 Feb  4 21:02 usr
drwxr-xr-x   1 root root 4096 Feb  4 21:04 var
{% endhighlight %}

After exiting, type `docker container ls -a` and the container shouldn't be there.

Deleting images
===============
Now it's time to delete the images. Let's check what images we have by running `docker image ls`.

{% highlight raw %}
~/docker [chuleh] » docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              47b19964fb50        2 weeks ago         88.1MB
hello-world         latest              fce289e99eb9        7 weeks ago         1.84kB
{% endhighlight %}

We can safely delete the `hello-world` image since we're not going be using it anymore.
Time to run `docker image rm`. Again, it can take `image id` or the `image name`.

{% highlight bash %}
~/docker [chuleh] » docker image rm fce289e99eb9
Untagged: hello-world:latest
Untagged: hello-world@sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Deleted: sha256:fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
Deleted: sha256:af0b15c8625bb1938f1d7b17081031f649fd14e6b233688eea3c5483994a66a3
{% endhighlight %}

Let's check if it disappeared with `docker image ls`.
{% highlight raw %}
~/docker [chuleh] » docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              47b19964fb50        2 weeks ago         88.1MB
{% endhighlight %}

Perfect, the `hello-world` image is gone.

Pruning
=======
Pruning is a good way for deleting several images or containers in one command.
Let's say we want to delete all the containers we've got on our system, you would run `docker container prune`.
{% highlight bash %}
~/docker [chuleh] » docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
a058c5234e4b19c45bfa7f346795ab32d5e72a33b74a68c1c0d9a55689b51337
ad6759ec86c7f5a7ad66af4673f23e3651940ad1b455146b1341a6f7f9053d23

Total reclaimed space: 0B
~/docker [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
{% endhighlight %}

For images, you would run `docker image prune`.

And if you want a total cleanup, you would run `docker system prune`. That will delete images, containers, volumes, networks and so forth.


So that's it for today, stay tuned cause in the next lesson we will take a more in-depth approach at images and port-forwarding.

:wq
