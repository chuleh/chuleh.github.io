Hi, in today's lecture we will continue with our `Dockerfile` tutorial. In our [previous lecture]({% post_url 2019-03-25-Docker:-what-is-a-Dockerfile %}) we created a `Dockerfile` for our `redis-server` and saw how to `build` and `run` it.

Now, we're going to create a `node.js` web application that says `Hello world`. It's a simple web app, nothing too fancy as I don't know much about `node.js`, the point here is that you grasp the knowledge of how a `Dockerfile` is built.

The node.js web app
===================
As always, I will work on my `docker` directory and create a new one called `nodeapp` and cd into it.

{% highlight raw %}
~/docker [chuleh●] » mkdir nodeapp && cd $_
docker/nodeapp [chuleh●] »
{% endhighlight %}

Inside of it, we will create three files. Two are for our `node.js` app, the remaining one will be the `Dockerfile`. Let's start with the web app. Open up your text editor and create a file called `package.json`, and paste the following code:

{% highlight raw %}
{
  "dependencies": {
    "express": "*"
  },
  "scripts": {
    "start": "node index.js"
  }
}
{% endhighlight %}


The second file is actually our web app. Create a new file called `index.js` and paste this code:
{% highlight raw %}
const express = require('express');

const app = express();

app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(8080, () => {
  console.log('Listening on port 8080');
});
{% endhighlight %}

That's it, we now have the two needed files to run a web application that will return `Hello world`.

Writing the Dockerfile
======================
Now that we've got our `node.js` app covered, it's time to focus and what really matters, the `Dockerfile`.

As with every single `Dockerfile`, the first line will be `FROM`. But in this case we're going to do a little tweak, we will tell `Docker` we want to use the `node` image, with the `alpine` tag.
`Alpine` in the docker world usually means that it is an image that comes installed with the bare minimum to run. It basically is the most lightweight possible image.

So the first line will look like this `FROM node:alpine`.

Now we need to `COPY` our application into the container. Copy works just the same as the UNIX command so there's nothing new here.

Type in `COPY ./ ./`, this will copy everything from the relative path to the container.

Next we need to tell the image to install `node`. So we're going to use the `RUN` command, the one that replicates the shell command we would run.

`RUN npm install` is the line that follows.

Finally, we need to tell our container what will be the command to execute when it starts.

`CMD ["npm", "start"]` is the final line we will type.

So your `Dockerfile` should look like this:
{% highlight raw %}
FROM node:alpine

COPY ./ ./

RUN npm install

CMD ["npm", "start"]
{% endhighlight %}

Building the image
==================
The process is the same, now we have to build our image. For that we will run the `docker build` command. Like I did in the previous lesson, I will tag the image with the `-t` flag.

{% highlight raw %}
docker/nodeapp [chuleh●] » docker build -t chuleh/nodeapp .
Sending build context to Docker daemon  4.096kB
Step 1/4 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/4 : COPY ./ ./
 ---> 39ecd43407f4
Step 3/4 : RUN npm install
 ---> Running in 0b095f8d21f6
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN !invalid#2 No description
npm WARN !invalid#2 No repository field.
npm WARN !invalid#2 No license field.

added 48 packages from 36 contributors and audited 121 packages in 2.35s
found 0 vulnerabilities

Removing intermediate container 0b095f8d21f6
 ---> 4937fe21e8bd
Step 4/4 : CMD ["npm", "start"]
 ---> Running in 8d9a47d434f7
Removing intermediate container 8d9a47d434f7
 ---> 4cc0fa9f67d9
Successfully built 4cc0fa9f67d9
Successfully tagged chuleh/nodeapp:latest
{% endhighlight %}

Your output should be similar to mine. Pay no attention to the WARN lines. Those are related to the `node.js` app, not to the `Dockerfile`.

As usual I'll double check if the image has been built with `docker image ls`.

{% highlight raw %}
docker/nodeapp [chuleh●] » docker image ls
REPOSITORY            TAG                 IMAGE ID            CREATED              SIZE
chuleh/nodeapp        latest              4cc0fa9f67d9        About a minute ago   78.3MB
{% endhighlight %}

Running the container
=====================
Now it's time to run the container with the image we created. You already know how to do this, `docker container run`.

{% highlight raw %}
docker/nodeapp [chuleh●] » docker container run chuleh/nodeapp

> @ start /
> node index.js

Listening on port 8080
{% endhighlight %}

The output on your end should be exactly the same as mine.

Testing the app
===============
So now that we've got our app running, we should be able to open a browser and go to `localhost:8080` or `curl` and get the `Hello World` message, right? Well, **no**. Let's see what happens.

Open a new tab on your terminal and type `curl localhost:8080`.

{% highlight raw %}
docker/nodeapp [chuleh●] » curl localhost:8080
curl: (7) Failed to connect to localhost port 8080: Connection refused
{% endhighlight %}

You see the connection is refused.

What's happening here? Well, `curl` is making a request to our local network on port `8080`, but there's nothing running on our local machine on port `8080`. [Just like we learned]({% post_url 2019-03-18-Docker:-port--forwarding %}), we need to do some port-forwarding to our container.

Go back to the tab where your `node app` is running and stop it with `ctrl-c`. Now, we will run the container once again with the `-p` flag.

{% highlight raw %}
docker/nodeapp [chuleh●] » docker container run -p 8080:8080 chuleh/nodeapp

> @ start /
> node index.js

Listening on port 8080
{% endhighlight %}

Now open a new tab on your terminal and `curl localhost:8080`.

{% highlight raw %}
docker/nodeapp [chuleh●] » curl localhost:8080
Hello World%
{% endhighlight %}
And that's it, we've got our `node.js` webapp running and docker is doing all the `port-forwarding` for us.

As always, hack around the `Dockerfile` and stay tuned for in the next post we will dive even deeper. We'll look at new commands and write the `node.js` app that will connect with our `redis-server`.

:wq
