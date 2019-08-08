Hi, in today's post we will take a look at a new command: `docker-compose`.
For this, we will create new nodejs app that will act as a visitor counter from the old days.

We will do a very basic docker-compose example, but should be enough to showcase the power of it.

Writing the nodejs app
======================
The nodejs app is pretty basic as I don't have much knowledge of nodejs, it's code I came across the internet just by doing a google search.
As always, I will work on my docker directory and create a new one called node-visits-app.
In that same directory, we will create two files, index.js and package.json.
Let's start with the index.js, open up your text editor and paste the following code:
```bash
const express = require('express');
const redis = require('redis');

const app = express();
const client = redis.createClient({
  host:'redis-server',
  port: 6379
});
client.set('visits', 0);

app.get('/', (req, res) => {
  client.get('visits', (err, visits) => {
    res.send('Number of visits is ' + visits);
    client.set('visits', parseInt(visits) + 1);
  });
});

app.listen(8081, () => {
  console.log('Listening on port 8081');
});
```
Just copy and paste the code, I promise you it will work.

Onto the package.json file. Again, in your text editor create a new file called package.json and paste the following code:
```bash
{
  "dependencies": {
    "express": "*",
    "redis": "2.8.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

Writing the Dockerfile
======================
Good, now that we have the two files for the node app we can focus on our Dockerfile. It will be very similar to the one we've been working with.
This should be pretty straight forward, but just in case, let's go over it once again.
The first line will be FROM, this is mandatory. There we will specify the base image we will use, in this case it's node:alpine.
Next we will specify our WORKDIR, in my case I'll use '/app', but remember this can be anything you want.
Then, as we learned in our [previous lesson]({% post_url 2019-04-01-Docker:-WORKDIR-and-rebuilds %}) we need to copy our package.json into the WORKDIR, so type in COPY package.json .
RUN npm install comes next.
After the installation, we'll copy all the files to the WORKDIR with COPY . .
And finally, we start our app with CMD ["npm", "start"]

Your Dockerfile should look like this:
```bash
FROM node:alpine

WORKDIR '/app'

COPY package.json .

RUN npm install

COPY . .

CMD ["npm", "start"]
```

Running the app
===============
Let's run the app and see what happens. If you paid attention to the code, you may have read that line 2 on our index.js says `const redis = require('redis');`
This means that our app needs redis to run properly. Most likely our app will fail when we try to run it, but don't worry, I got you covered. I'm just showing you some common mistakes you might run into when creating your own apps.
Let's build the image with docker build.
```bash
~/docker/node-visits-app  chuleh ✗                                              5d ✖
▶ docker build -t chuleh/node-visits-app .
Sending build context to Docker daemon  4.096kB
Step 1/6 : FROM node:alpine
 ---> 09084e4ff58d
Step 2/6 : WORKDIR '/app'
 ---> Using cache
 ---> e4c764a3b068
Step 3/6 : COPY package.json .
 ---> Using cache
 ---> 56da628b91ed
Step 4/6 : RUN npm install
 ---> Using cache
 ---> fa9df3429e73
Step 5/6 : COPY . .
 ---> dd290ca9fd71
Step 6/6 : CMD ["npm", "start"]
 ---> Running in a0b535fd2b51
Removing intermediate container a0b535fd2b51
 ---> 13384ec271c7
Successfully built 13384ec271c7
Successfully tagged chuleh/node-visits-app:latest
```

Once the image has been built, it's time to run it.

```bash
~/docker/node-visits-app  chuleh ✗                                              5d ✖
▶ docker run chuleh/node-visits-app

> @ start /app
> node index.js

Listening on port 8081
events.js:173
      throw er; // Unhandled 'error' event
      ^

Error: Redis connection to redis-server:6379 failed - getaddrinfo ENOTFOUND redis-server redis-server:6379
    at GetAddrInfoReqWrap.onlookup [as oncomplete] (dns.js:58:26)
Emitted 'error' event at:
    at RedisClient.on_error (/app/node_modules/redis/index.js:406:14)
    at Socket.<anonymous> (/app/node_modules/redis/index.js:279:14)
    at Socket.emit (events.js:197:13)
    at emitErrorNT (internal/streams/destroy.js:91:8)
    at emitErrorAndCloseNT (internal/streams/destroy.js:59:3)
    at processTicksAndRejections (internal/process/next_tick.js:76:17)
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @ start: `node index.js`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @ start script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /root/.npm/_logs/2019-04-02T17_15_23_563Z-debug.log
```
As it was expected, our app failed. Why's this? Let's take a look at the log.
It seems that in wants to start, but then we come across this line:
`Error: Redis connection to redis-server:6379 failed - getaddrinfo ENOTFOUND`.
Clearly, our app need the redis server to connect to. Let's take a look at that in the next section.

Writing the docker-compose file
==========================
As we saw before, our nodejs app can't run without redis. Now you might be thinking that we could create a new redis app and link them together through the CLI. And you wouldn't be wrong, but let me tell you something, the syntax to link containers from the CLI is awful.
That's why we'll look into `docker-compose`. This a command that already comes with Docker, so it should be installed by default in your system. What docker-compose allows us to, is to automate the writing from the CLI specifying it into a file which, as you may have guessed, it's called `docker-compose.yml`.
This docker-compose file will do all the networking for us and will run the containers as well.
So what we are going to do is the following:
- tell docker to run our redis server
- tell docker to run our nodejs app
- map the ports to have access to the app.

All of this, will be done using the docker-compose syntax.

Open up your text editor and create a new file called `docker-compose.yml`.
Let's start hacking, the first line will always be `version`. This tells the version of docker-compose we'll use.
```yaml
version: '3'
````
Thats your first line.
Now we want to tell docker what it should run. That means to tell docker to run our containers. Remember, we'll run redis and the nodejs app. We're going to use the word `services` to tell docker to run our containers. Then, we'll type in a name related to the service and finally we'll tell docker what image to use.
Type in the following:
```yaml
version: '3'
services:
  redis-server:
    image: 'redis'
```
I named it redis-server, for a good reason. If you looked at the code from the nodejs app, it says: `host:'redis-server'`. This is the hostname the app will use to connect to. If you change the name of the service, you need to change the name on the nodejs app as well.
Now it's time for the node app. Be **very** careful with the indentation, the next service we'll run is the nodejs app, this should be aligned at the same level of `redis-server`.
We're going to add another directive, the `build` one. We want docker to use the Dockerfile we created with our app, so for that, we'll tell docker `build: .`, that means to use the Dockerfile in the current directory.
```yaml
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    build: .
```
And finally, we need to map the ports for our app. So we'll use the `ports` directive and write an `array`. This would be the same as running our container with docker run -p localport:containerport.
```yaml
version: '3'
services:
  redis-server:
    image: 'redis'
  node-app:
    build: .
    ports:
     - "4000:8081"
```
I used port 4000 as the local port, but it could've been any other one.

Running our docker-compose.yml
===============
Now it's time to run our docker-compose file. Believe it or not, that's all there is to it. Just by defining our containers inside the docker-compose.yml, docker will create a network between our two containers son they can communicate with each other.
To run our docker-compose file, just type `docker-compose up`. Of course, this has to be done in the directory where the `docker-compose.yml` file is located, otherwise it will give an error.

```bash
~/docker/node-visits-app  chuleh ✗                                              5d ⚑
▶ docker-compose up
Starting node-visits-app_node-app_1     ... done
Starting node-visits-app_redis-server_1 ... done
Attaching to node-visits-app_redis-server_1, node-visits-app_node-app_1
redis-server_1  | 1:C 02 Apr 2019 19:15:38.821 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
redis-server_1  | 1:C 02 Apr 2019 19:15:38.821 # Redis version=5.0.4, bits=64, commit=00000000, modified=0, pid=1, just started
redis-server_1  | 1:C 02 Apr 2019 19:15:38.821 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
redis-server_1  | 1:M 02 Apr 2019 19:15:38.824 * Running mode=standalone, port=6379.
redis-server_1  | 1:M 02 Apr 2019 19:15:38.825 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
redis-server_1  | 1:M 02 Apr 2019 19:15:38.825 # Server initialized
redis-server_1  | 1:M 02 Apr 2019 19:15:38.825 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
redis-server_1  | 1:M 02 Apr 2019 19:15:38.826 * DB loaded from disk: 0.002 seconds
redis-server_1  | 1:M 02 Apr 2019 19:15:38.826 * Ready to accept connections
node-app_1      |
node-app_1      | > @ start /app
node-app_1      | > node index.js
node-app_1      |
node-app_1      | Listening on port 8081
```

If we take a look at the console output, we can see two things: first, our database says `ready to accept connections` and our nodejs app says `Listening on port 8081`.

I will open another tab on my terminal and see curl localhost:4000. Remember, it's the local port the one you curl, not the container port.

```bash
▶ curl localhost:4000
Number of visits is 0
```

Seems to be working. I'll curl another time and it should say 1.
```bash
▶ curl localhost:4000
Number of visits is 1
```

Perfect. We've got our app running and the networking between the two containers has been taken care of by docker. Go back to the docker-compose tab and press ctrl-c to bring down the app. If you ran it detached, just type `docker-compose down`.


And that's our docker-compose. This ends today's lesson. As always, stay tuned for in the next lesson we will look at how docker integrates in our workflow, we'll create a complete CI/CD pipeline with docker included.

`:wq`
