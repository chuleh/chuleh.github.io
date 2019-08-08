Hi, in today's post we will continue from where we left off with our Docker series. We're going to work with the Dockerfile from our [previous lesson]({% post_url 2019-03-28-Docker:-diving-deeper-into-Dockerfile %}) and work with the nodeapp container.
We will also take at rebuilds and how to best avoid them.

Dockerfile WORKDIR
==================

Fire up the container from the previous post and list the content inside of it.
```bash
docker/nodeapp [chuleh] » docker container run -i -t chuleh/nodeapp sh
/ # ls -l
total 88
-rw-r--r--    1 root     root            67 Mar 24 23:17 Dockerfile
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 bin
drwxr-xr-x    5 root     root           360 Mar 25 00:26 dev
drwxr-xr-x    1 root     root          4096 Mar 25 00:26 etc
drwxr-xr-x    1 root     root          4096 Mar 18 21:20 home
-rw-r--r--    1 root     root           192 Mar 24 22:52 index.js
drwxr-xr-x    1 root     root          4096 Mar  4 15:43 lib
drwxr-xr-x    5 root     root          4096 Mar  4 15:43 media
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 mnt
drwxr-xr-x   51 root     root          4096 Mar 24 23:18 node_modules
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 opt
-rw-r--r--    1 root     root         13052 Mar 24 23:18 package-lock.json
-rw-r--r--    1 root     root            96 Mar 24 22:51 package.json
dr-xr-xr-x  182 root     root             0 Mar 25 00:26 proc
drwx------    1 root     root          4096 Mar 25 00:26 root
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 run
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 sbin
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 srv
dr-xr-xr-x   13 root     root             0 Mar 25 00:26 sys
drwxrwxrwt    1 root     root          4096 Mar 18 21:53 tmp
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 usr
drwxr-xr-x    1 root     root          4096 Mar  4 15:43 var
```
As you can see, we have several files scattered around the filesystem. This is usually a **bad** practice. Why's that? Well, imagine you have /lib directory for your project. When you tell your Dockerfileto do COPY ./ ./ it will overwrite your filesystem's /lib directory and will probably mess up your container.

Fortunately there's a solution for this: WORKDIR. This is a new command we will start using from now on. WOKDIR is pretty simple, you just type WORKDIR and the path that will be your new working directory.

Start your text editor and open the Dockerfile from our previous lesson. Inside of it, right below the FROM command we will type in our WORKDIR. I will use /app. Docker's smart enough to create the directory for us if it doesn't exist.

So your Dockerfile now should look like this
```bash
FROM node:alpine

WORKDIR /app

COPY ./ ./

RUN npm install

CMD ["npm", "start"]
```

I'm using /app, but you could use /usr/app or any other name that comes to your mind.

Rebuilding the image
====================
If you're still inside the container, exit. It's time to rebuild the image. What do you think is going to happen when we build our image again? Let's find out.

```bash
docker/nodeapp [chuleh●] » docker build -t chuleh/nodeapp .
Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> e4c764a3b068
Step 3/5 : COPY ./ ./
 ---> 4f60c019e639
Step 4/5 : RUN npm install
 ---> Running in b09a202d51c8
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN app No description
npm WARN app No repository field.
npm WARN app No license field.

added 48 packages from 36 contributors and audited 121 packages in 2.92s
found 0 vulnerabilities

Removing intermediate container b09a202d51c8
 ---> 54e7a1431308
Step 5/5 : CMD ["npm", "start"]
 ---> Running in d5fd1ccb87f3
Removing intermediate container d5fd1ccb87f3
 ---> 6ab55ea3f0ef
Successfully built 6ab55ea3f0ef
Successfully tagged chuleh/nodeapp:latest
```

As you can see, since we made a change to the Dockerfile, every other command will be run again meaning we will rebuild our image from scratch.

Starting the container
======================
Now that we have our new image, it's time to run our container again, this time I will do it detached. By now you should already know how to do this.
I will run the container and curl localhost to see if it's working.

```bash
docker/nodeapp [chuleh●] » docker container run -p 8080:8080 -d chuleh/nodeapp
402e7b40dcb21a45680f4567ecb3771bf29ecaa4b12be70a76b44ed0f582dc02
docker/nodeapp [chuleh●] » curl localhost:8080
Hello World
```

Now it's time to check if our WORKDIR directive worked. To do this we will use docker exec to get a shell inside our container. First I will check the ID of my container with docker ps and then will attach to it.

```bash
docker/nodeapp [chuleh●] » docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
402e7b40dcb2        chuleh/nodeapp      "npm start"         2 minutes ago       Up 2 minutes        0.0.0.0:8080->8080/tcp   keen_lumiere
docker/nodeapp [chuleh●] » docker exec -i -t 402e7b40dcb2 sh
/app #
```

Notice how my prompt immediately changed to /app. List the contents of the directory, you should have all of your files in there. And if you go one directory back into the rootdir, the files shouldn't be there since we rebuilt the image.

```bash
/app # ls -l
total 32
-rw-r--r--    1 root     root            81 Mar 25 00:38 Dockerfile
-rw-r--r--    1 root     root           192 Mar 24 22:52 index.js
drwxr-xr-x   51 root     root          4096 Mar 25 00:43 node_modules
-rw-r--r--    1 root     root         13052 Mar 25 00:43 package-lock.json
-rw-r--r--    1 root     root            96 Mar 24 22:51 package.json
/app # cd ../
/ # ls -l
total 60
drwxr-xr-x    1 root     root          4096 Mar 25 00:43 app
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 bin
drwxr-xr-x    5 root     root           340 Mar 25 00:51 dev
drwxr-xr-x    1 root     root          4096 Mar 25 00:51 etc
drwxr-xr-x    1 root     root          4096 Mar 18 21:20 home
drwxr-xr-x    1 root     root          4096 Mar  4 15:43 lib
drwxr-xr-x    5 root     root          4096 Mar  4 15:43 media
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 mnt
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 opt
dr-xr-xr-x  183 root     root             0 Mar 25 00:51 proc
drwx------    1 root     root          4096 Mar 25 00:55 root
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 run
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 sbin
drwxr-xr-x    2 root     root          4096 Mar  4 15:43 srv
dr-xr-xr-x   13 root     root             0 Mar 25 00:51 sys
drwxrwxrwt    1 root     root          4096 Mar 18 21:53 tmp
drwxr-xr-x    1 root     root          4096 Mar 18 21:53 usr
drwxr-xr-x    1 root     root          4096 Mar  4 15:43 var
```

If you pay attention to detail, you'll see that in our /app directory we have a new one called node_modules. We will look into that.

Rebuilds and how to avoid them
==============================
There's one last thing I'd like us to look at before moving any forward. Start your text editor and open up the index.js file. If you copy-pasted the file just like I wrote it down in the previous post, line 6 should be your Hello World line.

Change the Hello World message to whatever you want. I will type Bye World (super original). Save the file and exit.
Now, if you curl localhost:8080 again, what do you think will happen? Nothing.

```bash
docker/nodeapp [chuleh●] » curl localhost:8080
Hello World
```

What's going on? Every time we create an image, we're taking a snapshot of the filesystem. So we took a snapshot of index.js after it got copied over, meaning we're running an older version of index.js in our container. This also means that changes are not automatically reflected in our container.
So how do we get our changes running in the container? The main thing you'd be thinking is to rebuild. Let's try doing that and then we will look into a better solution.

I will stop the container and rebuild it. Then I will start the container and curl localhost:8080.
```bash
docker/nodeapp [chuleh●] » docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
402e7b40dcb2        chuleh/nodeapp      "npm start"         22 minutes ago      Up 22 minutes       0.0.0.0:8080->8080/tcp   keen_lumiere
docker/nodeapp [chuleh●] » docker container stop 402e7b40dcb2
402e7b40dcb2
docker/nodeapp [chuleh●] » docker build -t chuleh/nodeapp .
Sending build context to Docker daemon  4.096kB
Step 1/5 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/5 : WORKDIR /app
 ---> Using cache
 ---> e4c764a3b068
Step 3/5 : COPY ./ ./
 ---> 7992fcee12f9
Step 4/5 : RUN npm install
 ---> Running in 4312d94d0865
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN app No description
npm WARN app No repository field.
npm WARN app No license field.

added 48 packages from 36 contributors and audited 121 packages in 2.49s
found 0 vulnerabilities

Removing intermediate container 4312d94d0865
 ---> 3b9a2fc4af19
Step 5/5 : CMD ["npm", "start"]
 ---> Running in 098bd6172512
Removing intermediate container 098bd6172512
 ---> 1f65862b6858
Successfully built 1f65862b6858
Successfully tagged chuleh/nodeapp:latest
```
As you can see, the image has been rebuilt from scratch.

Now let's run the container and curl localhost.
```bash
docker/nodeapp [chuleh●] » docker container run -p 8080:8080 -d chuleh/nodeapp
7e53f12eb2dcaf51670aeb7891f358503d734493454b3446cf9b8babf1506473
docker/nodeapp [chuleh●] » curl localhost:8080
Bye World
```

The change is in effect. This was rather fast, but going through all of this when you're working in larger projects is really a pain. Let's find out a better way to tell docker we want to make some changes without going through all the fuss of rebuilding.

Editing the Dockerfile (again)
==============================
We know that the Dockerfile is a series of instructions saying how we want our container to behave. We also know, or at least you'll know now, that node needs the package.json to run. That's all it cares about.
So what if we changed the order in which files are copied? Start your text editor and open up the Dockerfile. We're going to split the COPY instruction, into two.

Go to the COPY line and type in the following
```bash
COPY ./package.json ./
```

Below you should have the RUN npm install line. And below that, type the following line COPY ./ ./. Your Dockerfile should look like this.
```bash
FROM node:alpine

WORKDIR /app

COPY ./package.json ./

RUN npm install

COPY ./ ./

CMD ["npm", "start"]
```
What's going to happen now? We can make all the changes that we want to our files and it won't invalidate the building process. The only time npm install will be run again, is if we change any of the steps above it. So what have you learned? **Order matters** in a Dockerfile.

Rebuilding the image
====================
Now, let's rebuild the image and see what happens. First I'll stop the container, then I will rebuild the image.
```bash
docker/nodeapp [chuleh●] » docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
7e53f12eb2dc        chuleh/nodeapp      "npm start"         17 minutes ago      Up 17 minutes       0.0.0.0:8080->8080/tcp   tender_tereshkova
docker/nodeapp [chuleh●] » docker container stop 7e53f12eb2dc
7e53f12eb2dc
docker/nodeapp [chuleh●] » docker build -t chuleh/nodeapp .
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> e4c764a3b068
Step 3/6 : COPY ./package.json ./
 ---> af8e04c73fbd
Step 4/6 : RUN npm install
 ---> Running in 509db6ac296b
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN app No description
npm WARN app No repository field.
npm WARN app No license field.

added 48 packages from 36 contributors and audited 121 packages in 2.672s
found 0 vulnerabilities

Removing intermediate container 509db6ac296b
 ---> 883ab345b1c9
Step 5/6 : COPY ./ ./
 ---> f736f9c1ff1f
Step 6/6 : CMD ["npm", "start"]
 ---> Running in 64168adf8003
Removing intermediate container 64168adf8003
 ---> 937494f32de7
Successfully built 937494f32de7
Successfully tagged chuleh/nodeapp:latest
```

The build ran successfully, but it still had to run all the commands again. This was expected as we edited the Dockerfile.
Now, let's edit the index.js again. Where I wrote Bye world, I will type how you doing? and will rebuild the image. What do you think is going to happen this time? Let's find out.
```bash
docker/nodeapp [chuleh●] » docker build -t chuleh/nodeapp .
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/6 : WORKDIR /app
 ---> Using cache
 ---> e4c764a3b068
Step 3/6 : COPY ./package.json ./
 ---> Using cache
 ---> af8e04c73fbd
Step 4/6 : RUN npm install
 ---> Using cache
 ---> 883ab345b1c9
Step 5/6 : COPY ./ ./
 ---> 1b1e3012dccc
Step 6/6 : CMD ["npm", "start"]
 ---> Running in edb4e9c77e00
Removing intermediate container edb4e9c77e00
 ---> 1ce73d7510bc
Successfully built 1ce73d7510bc
Successfully tagged chuleh/nodeapp:latest
```
You can see that the output is filled with Using cache lines and that in ran way faster than it ran in the previous time. That's because the only changed that we made, was the COPY ./ ./ step.

Let's run the container again and curl it to see if the changes were made.
```bash
docker/nodeapp [chuleh●] » docker container run -p 8080:8080 -d chuleh/nodeapp
4de0b6265a475f22c09b8a639d48c6e3fb3167fd95fda4398c4f1a1d0955adc3
docker/nodeapp [chuleh●] » curl localhost:8080
How you doing?
```


And that's it. We've got our image running way faster than it was before with just a few tweaks. Remember, **order matters**.
As always, stay tuned for in the next lesson we'll take a look at a different node.js app and we will take a first look at docker-compose.

`:wq`
