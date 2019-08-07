Hi, in today's post we're going to take a look at something completely new: `Dockerfile`. Until now we have been using community images that just worked, but when you're going to use `docker` in production, you'll most likely want to run your own images. Here's when the `Dockerfile` comes into play.
But before diving any further into `Dockerfile`, let's do a little recap about what we've learned.

What you know so far
====================
- By now you should know how to [start and stop containers]({% post_url 2019-02-21-Docker:-working-with-containers %})
- Know how to [get a shell in a container]({% post_url 2019-02-21-Docker:-working-with-containers %})
- Have a solid understanding of [port-forwarding and running containers detached]({% post_url 2019-03-18-Docker:-port--forwarding %})
- Get the difference between a [bind mount and a volume]({% post_url 2019-03-22-Docker:-bind-mounts-and-volumes %})

I **can't stress** enough how important it is for you to have a solid understanding of the previous posts. Everything we've learned so far has been to create a solid foundation for what we're going to learn today and the series of posts to come. That is `Dockerfile` and `docker-compose`.

Dockerfile
==========
So what's a `Dockerfile`? It's basically a plain text file that holds a configuration telling our container how it should behave. Through our `CLI`, the `docker client` will connect with the `docker server`, the one that does all the heavy lifting for us. It will take the `Dockerfile`, go through all the lines of configuration written inside of it and in return will build us a usable image.

Let's take a look at the flow of a `Dockerfile`. First of all, notice how I always type `Dockerfile`, with a capital D. That's how the file will be named, this is mandatory. You can change the name of it, but for now let's stick to `Dockerfile`.

The main line of a `Dockerfile` will always be `FROM`. This is mandatory by `docker`. This is where we will specify the `base image`.

Then we will have other commands like `RUN`. This is where you will build up the image you're creating. The syntax for a `RUN` instruction, is to type down the shell command.

We also have `COPY`. As you may have guessed, this copies local files into the container.

`CMD` is to tell the image what command will run when it starts up. There can only be one `CMD` in a `Dockerfile`.

There are several more commands inside a `Dockerfile`, but for now let's just stick to these:
- FROM
- COPY
- RUN
- CMD

Writing your first Dockerfile
=============================
We're going to write a `Dockerfile` for `node.js` app that will count hits on our website. For this, we will need a `redis-server` and the `node.js` app itself.
In this first lesson we will only cover the `Dockerfile` for the `redis-server`.

As usual, I will work on my `docker` directory and create a new one called `visitor-counter`.
{% highlight raw %}
$ mkdir visitor-counter
{% endhighlight %}
Inside we will create our `Dockerfile`. As I explained before, the first line will be `FROM`, specifying the base image we will use.
In this case I won't use `ubuntu` as it's too heavy, we will use `alpine`.
{% highlight raw %}
FROM alpine
{% endhighlight %}

Now it's time to `RUN` the command to install `redis`. As you may have guessed, we will type in `RUN` and add a shell command from `alpine`.
{% highlight raw %}
RUN apk add --update redis
{% endhighlight %}

Finally, we will type `CMD` instructing our `Dockerfile` to run the `redis-server`.
{% highlight raw %}
CMD ["redis-server"]
{% endhighlight %}

Your file should look like this:
{% highlight raw %}
FROM alpine

RUN apk add --update redis

CMD ["redis-server"]
{% endhighlight %}

Building the image
==================
So now we have our `Dockerfile` ready. It's time to build it. For this we're going to run a new command, `docker build`. I will also use the `-t` flag to `tag` the image with my `username/name-of-the-image`. And finally we type a `.`, a dot. Telling `docker` we'll build in the current directory.

A quick note, tagging the image is usually writen like this: `username/name-of-the-image:version`. This is useful for when you have several images of the same service, but in my case I will just have one `redis-server`.
If you don't specify the version, it will default to `latest`.

{% highlight raw %}
docker/visitor-counter [chuleh●] » docker build -t chuleh/redis-server .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM alpine
 ---> 5cb3aa00f899
Step 2/3 : RUN apk add --update redis
 ---> Running in 1f1ce6c253e7
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/main/x86_64/APKINDEX.tar.gz
fetch http://dl-cdn.alpinelinux.org/alpine/v3.9/community/x86_64/APKINDEX.tar.gz
(1/1) Installing redis (4.0.12-r0)
Executing redis-4.0.12-r0.pre-install
Executing redis-4.0.12-r0.post-install
Executing busybox-1.29.3-r10.trigger
OK: 7 MiB in 15 packages
Removing intermediate container 1f1ce6c253e7
 ---> 5dcdaa0db544
Step 3/3 : CMD ["redis-server"]
 ---> Running in 94611db3a24d
Removing intermediate container 94611db3a24d
 ---> e9a44d9ad7e6
Successfully built e9a44d9ad7e6
Successfully tagged chuleh/redis-server:latest
{% endhighlight %}

If everything went correct, your output should be similar to mine. We can double check if the image has been built by typing `docker image ls`.
{% highlight raw %}
docker/visitor-counter [chuleh●] » docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
chuleh/redis-server   latest              e9a44d9ad7e6        2 minutes ago       8.06MB
{% endhighlight %}


Running the container
=====================
 Now, all that's left is to run the container. We already know how to do that, `docker container run`.
 {% highlight bash %}
docker/visitor-counter [chuleh●] » docker container run chuleh/redis-server
1:C 23 Mar 09:16:04.866 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 23 Mar 09:16:04.866 # Redis version=4.0.12, bits=64, commit=1be97168, modified=0, pid=1, just started
1:C 23 Mar 09:16:04.866 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 23 Mar 09:16:04.868 * Running mode=standalone, port=6379.
1:M 23 Mar 09:16:04.868 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
1:M 23 Mar 09:16:04.868 # Server initialized
1:M 23 Mar 09:16:04.868 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
1:M 23 Mar 09:16:04.868 * Ready to accept connections
{% endhighlight %}

Again, your output should be similar to mine. Pay attention to the last line, it says `Ready to accept connections`. Meaning our container ran successfully.


That'll be all for today's lesson. Stay tuned cause in the next lecture we're going to create a `node.js` app that will be the foundation for our next projects.

:wq
