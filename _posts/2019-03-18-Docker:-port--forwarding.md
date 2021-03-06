Hi, in today's post we will continue where we left off with our `docker` series. If you haven't read the previous posts, I recommend doing so. There's the [Intro to Docker]({% post_url 2019-02-20-intro-to-docker %}) post and [Docker Part 2]({% post_url 2019-02-21-Docker:-working-with-containers %}).

In part 3 we will talk about port-forwarding. We will create a basic webserver using the `Nginx` image.

Pulling the Nginx image
=======================

First, let's create an `Nginx` directory where we are going to work and then pull the `Nginx` image from the `docker` hub.
This may take some time depending on your connection, but it's usually pretty fast.

{% highlight raw %}
~/docker [chuleh] » mkdir nginx
~/docker [chuleh] » cd nginx
docker/nginx [chuleh] » docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
f7e2b70d04ae: Pull complete
08dd01e3f3ac: Pull complete
d9ef3a1eb792: Pull complete
Digest: sha256:98efe605f61725fd817ea69521b0eeb32bef007af0e3d0aeb6258c6e6fe7fc1a
Status: Downloaded newer image for nginx:latest
{% endhighlight %}

Your output should be similar to mine.

Running the container
=====================

Now it's time to run the container. We've already covered how to do that, but in this case we're going to add some extra flags. We're going to use the `-p` flag to tell `docker` we want to map the `host port` to a `container port`. Remember this: `host` to the left, `container` to the right.
In my case, I'm going to map port `8080` to port `80`.
Type in the following command `docker run -p 8080:80 nginx`
{% highlight raw %}
docker/nginx [chuleh] » docker run -p 8080:80 nginx
{% endhighlight %}

The container should be running now. Now open another `terminal` or go to `localhost:8080` on your `browser`.
I will open another tab on my terminal and `curl` the webserver.

{% highlight raw %}
~ » curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.15.9
Date: Mon, 18 Mar 2019 15:00:22 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 26 Feb 2019 14:13:39 GMT
Connection: keep-alive
ETag: "5c754993-264"
Accept-Ranges: bytes
{% endhighlight %}

I got a `200`, which means that's ok. The webserver is up. Now go back to your `docker` tab, you should see the hit on your log.
{% highlight raw %}
docker/nginx [chuleh] » docker run -p 8080:80 nginx
172.17.0.1 - - [18/Mar/2019:15:00:22 +0000] "HEAD / HTTP/1.1" 200 0 "-" "curl/7.54.0" "-"
{% endhighlight %}

Just to double check, I will run `docker container ls -a`. It will show the `container` that's running.
{% highlight raw %}
~ » docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
9bbbc3c3cbf1        nginx               "nginx -g 'daemon of…"   7 minutes ago       Up 7 minutes        0.0.0.0:8080->80/tcp   boring_shaw
{% endhighlight %}

Now go back to the `Nginx` tab and hit `ctrl-c`. I will stop the container. Once again, let's double check with `docker container ls -a`.
{% highlight raw %}
~ » docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
9bbbc3c3cbf1        nginx               "nginx -g 'daemon of…"   10 minutes ago      Exited (0) 5 seconds ago                       boring_shaw
{% endhighlight %}

Running the container detached
==============================

Running the container detached is a good way to free up some space in your `terminal`. To do so, we will have to add another flag, in this case, the `-d` flag, which tells docker to run `detached`, meaning the container will run in the `background`.
Type in `docker container run -p 8080:80 -d nginx`. It will print out the `container id`.
{% highlight raw %}
docker/nginx [chuleh] » docker container run -p 8080:80 -d nginx
5ae043c5d589e11a5e473e7a547128d964c7f01c9cc15bf9ca460481c8948c92
{% endhighlight %}

Let's check if it's really running with a `curl` or by going with your `browser` to `localhost:8080`.
{% highlight raw %}
docker/nginx [chuleh] » curl -I localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.15.9
Date: Mon, 18 Mar 2019 15:19:27 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 26 Feb 2019 14:13:39 GMT
Connection: keep-alive
ETag: "5c754993-264"
Accept-Ranges: bytes
{% endhighlight %}

We can also check it by running `docker containerls -a`.
{% highlight raw %}
docker/nginx [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                  NAMES
5ae043c5d589        nginx               "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes                0.0.0.0:8080->80/tcp   peaceful_jang
9bbbc3c3cbf1        nginx               "nginx -g 'daemon of…"   24 minutes ago      Exited (0) 13 minutes ago                          boring_shaw
{% endhighlight %}
Your output should be similar to this. I've got the `nginx` container from the previous example `exited` and the `container` from this example `running`.

Attaching to the container
==========================
What now? Well, we've got our `container` running, it's time to attach to it and execute some commands.
To attach to your container type `docker exec -i -t <container id> bash` and you should be inside your container.

{% highlight raw %}
docker/nginx [chuleh] » docker exec -i -t 5ae043c5d589 bash
root@5ae043c5d589:/#
{% endhighlight %}

Now we can run any commands inside our `container`. Be mindful that since the containers come with the bare minimum installed, some commands may not be available unless you install the software. Type `exit` or press `ctrl-d` to exit the container.
Now type in `docker container ls -a`.

{% highlight raw %}
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS                  NAMES
9145872cf703        nginx               "nginx -g 'daemon of…"   2 minutes ago       Up 2 minutes                0.0.0.0:8080->80/tcp   condescending_meninsky
{% endhighlight %}

As you can see, we exited the `container` but it's still running. Since we're not going to use it anymore, let's stop it with `docker container stop <container id>`.

{% highlight raw %}
docker/nginx [chuleh] » docker container stop 9145872cf703
9145872cf703
{% endhighlight %}

And double check it's stopped with `docker container ls -a`.
{% highlight raw %}
docker/nginx [chuleh] » docker container ls -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
9145872cf703        nginx               "nginx -g 'daemon of…"   4 minutes ago       Exited (0) 4 seconds ago                        condescending_meninsky
{% endhighlight %}


And that's it for now. Stay tuned, in the next tutorial we will	 talk about `volumes` and `creating an image`.

:wq
