Hi, in today's post we're going to continue with our `docker` series. If you're new to `docker` I suggest reading at least my [Intro to Docker]({% post_url 2019-02-20-intro-to-docker %}).
In this lecture we're going to talk about `bind mounts` and `volumes`.
We're going to use the `nginx` image from our [previous lesson]({% post_url 2019-03-18-Docker:-port--forwarding %}) and do stuff with it, so my recommendation is if you haven't done that tutorial yet, go do it first.

What's the point of of this? Well, `bind mounts` and `volumes` are perfect for when you want to stop and delete containers without losing data.

Bind mounts
====================
A `bind mount` is rather simple. It takes a host path, eg: `/data` and mounts it inside your container, eg: `/usr/data`.
It is very simple and fast to use, the only downside is that you need to specify it at runtime. That means you need to specify it when you type our `docker container run` command.

Let's take a look at it. I'm going to work on my `nginx` directory from the previous lesson. By now you should already have your `nginx` image. If you're like me that you regularly `prune` your computer, just download it again with `docker pull nginx`.

{% highlight raw %}
docker/nginx [chuleh●] » docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
Digest: sha256:98efe605f61725fd817ea69521b0eeb32bef007af0e3d0aeb6258c6e6fe7fc1a
Status: Image is up to date for nginx:latest
{% endhighlight %}

Next, I will create a directory named `content` where I will store a very basic `index.html` file.
{% highlight raw %}
docker/nginx [chuleh●] » mkdir content
docker/nginx [chuleh●] » cd content
nginx/content [chuleh●] » echo "Hello World" > index.html
{% endhighlight %}

Now it's time to run the container with our data, for this we're going to use the `-v` flag when we run our `docker container`. Remember from the past lesson we need to specify the port with the `-p` flag and I will run it with the `-d` flag for it to run detached.

The command should look similar to this `docker container run -p 8080:80 -v /Users/chuleh/docker/nginx/content/:/usr/share/nginx/html:ro -d nginx`.
Of course, change the path to match the one on your end.
{% highlight raw %}
docker/nginx [chuleh●] » docker container run -p 8080:80 -v /Users/chuleh/docker/nginx/content/:/usr/share/nginx/html:ro -d nginx
b8d993a08c463e2fcc2e497ca8f54ec6e4087a0edd66f4e3ed9aaeb0ab3e2df3
{% endhighlight %}

Also notice I typed `:ro`, this is to make the volume `read-only` making the container unable to edit the files on the host.

If everything went correctly, I should be able to `curl` localhost and get the `Hello world` message back.
{% highlight raw %}
docker/nginx [chuleh●] » curl localhost:8080
Hello World
{% endhighlight %}

And that's it, you've got a container running with your custom data. Now we can stop the container and the data will remain in our local directory.
{% highlight raw %}
docker/nginx [chuleh●] » docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
b8d993a08c46        nginx               "nginx -g 'daemon of…"   12 minutes ago      Up 12 minutes       0.0.0.0:8080->80/tcp   dreamy_bohr
docker/nginx [chuleh●] » docker stop b8d993a08c46
b8d993a08c46
{% endhighlight %}


Volumes
================
Volumes are entities inside `docker`. We're going to create a volume named `data` from the `cli` with the following command: `docker volume create data`.
{% highlight raw %}
docker/nginx [chuleh●] » docker volume create data
data
{% endhighlight %}
You see `docker` creates the `volume` and outputs its name. If you paid attention, you'll see we didn't specify where the data is mounted on the host. Do not worry, we're going to use the `inspect` command to have an in-depth look at the volume.
Type `docker volume inspect data`
{% highlight raw %}
docker/nginx [chuleh●] » docker volume inspect data
[
    {
        "CreatedAt": "2019-03-22T18:03:07Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/data/_data",
        "Name": "data",
        "Options": {},
        "Scope": "local"
    }
]
{% endhighlight %}
We can see that the data is stored in `/var/lib/docker/volumes/data/_data` on the host.

Now, we only need to run the container with the `-v` flag again.
{% highlight raw %}
docker/nginx [chuleh●] » docker container run -v data:/usr/share/nginx/html -p 8080:80 -d nginx
312142394f02a7d5ce0949b49ed644eea7f95eed8ab116818cf2279c208d20bf
{% endhighlight %}

Data is empty so there's no point in doing a `curl` to the localhost:8080 as it will return the default nginx index.html.

Cleaning up
===========
Stop your container with `docker stop`. Then we need to check the volumes that are scattered around with `docker volume ls`.
{% highlight raw %}
docker/nginx [chuleh●] » docker volume ls
DRIVER              VOLUME NAME
local               content
local               data
{% endhighlight %}

You can see I have two volumes. I can delete them with `docker volume rm` or I can `prune` them with `docker volume prune`. In my case I will use `prune` since I don't care about these two volumes.




Wrap up
=======
- `-v /path/to/something:/path/in/container` is a bind mount.
- `-v path:/path/in/container` is a volume.

And that's pretty much it. Make sure you're comfortable with this, play around with the commands for in the next lesson we're going to take a more in-depth look at docker and `Dockerfile`, allowing us to create our own images.

:wq
