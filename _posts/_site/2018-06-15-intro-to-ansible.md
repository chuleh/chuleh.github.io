Hi, in this post we're going to talk about [Ansible](https://www.ansible.com/){:target="_blank"}. So, what's `Ansible`?
Well, as the website says, `Ansible` is an IT orchestration engine that automates configuration management, application
deployment and many other IT needs. What does this mean? This means that the days of logging via `ssh` into servers to
install and configure them are gone! Through `Ansible` we will be able to install all of our apps, configure our files
 and do all sorts of tasks we can think of.
<!--more-->

Today we're going to learn how to:
- Setting up the host VM
- Install Ansible
- Run ad-hoc commands
- Create a simple playbook

Installing Ansible
==================
We're going to cover how to install `Ansible` on `GNU\Linux` and `macOS`.
##### Centos
{% highlight bash %}
sudo yum install ansible -y
{% endhighlight %}
##### Fedora
{% highlight bash %}
sudo dnf install ansible -y
{% endhighlight %}
##### Ubuntu
Ubuntu is a bit trickier but don't worry, I got you covered. We're going to install it through `Ansible` PPA's.
{% highlight bash %}
sudo apt-get update
sudo apt-get install software-properties-common -y
sudo apt-add-repository ppa:ansible/ansible
sudo apt-get update
sudo apt-get install ansible -y
{% endhighlight %}
##### Debian
On Debian you need to edit your `sources.list`. Type the following:
{% highlight bash %}
sudo vim /etc/apt/sources.list
{% endhighlight %}
And add the following to your `sources.list`:
```
deb http://ppa.launchpad.net/ansible/ansible/ubuntu trusty main
```
And then type following:
{% highlight bash %}
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 93C4A3FD7BB9C367
sudo apt-get update
sudo apt-get install ansible -y
{% endhighlight %}
##### macOS
On macOS it is recommended to install it via `pip`. Make sure you have `pip` installed by typing `which pip`. If you don't have `pip` installed, just type:
{% highlight bash %}
sudo easy_install pip
{% endhighlight %}
And once `pip` is installed, type the following:
{% highlight bash %}
sudo pip install ansible
{% endhighlight %}

Setting up the host VM
======================
A few things before we start. The `host` server __must__ have `Python` installed, otherwise `Ansible` will fail. For
this tutorial I'm going to run an `Ubuntu vm` following my [intro to vagrant]({% post_url 2018-06-14-intro-to-vagrant %})
post, so `Python` is already installed and it already has a `static ip`. Also, I have already created a new user inside the `vm`
which is the one I will be using for this post. If you haven't done any of these two, either `vagrant ssh` into your `vm` and install `Python`
and `adduser` your new user or follow [this tutorial]({% post_url 2018-06-14-intro-to-vagrant %}). This new user __must__ be in the sudoers file with permissions to run sudo without asking for password. I'll cover this quickly below if you haven't done it yet, `vagrant ssh` into your new `vm`. Then `sudo -i` to become `root`. Afterwards `adduser <username>` and create the new `user`. Type `adduser <username> sudo` and `visudo`.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ sudo -i
root@vagrant-ubuntu-trusty-64:~# adduser chuleh
[...]
Is the information correct? [Y/n] y
root@vagrant-ubuntu-trusty-64:~# adduser chuleh sudo
Adding user `chuleh' to group `sudo' ...
Adding user chuleh to group sudo
Done.
root@vagrant-ubuntu-trusty-64:~# visudo
{% endhighlight %}
Inside your `sudoers` file, scroll down to the line where we have the group `sudo` and change it so the line looks like this:
```
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```
Good, now we're all set up to run `Ansible`.

Now it's time to tell `Ansible` about this host `server`.
 Let's edit or create the following file: `/etc/ansible/hosts`.
 {% highlight bash %}
 sudo vim /etc/ansible/hosts
 {% endhighlight %}
 And lets add our `vm`. Mine, in the previous post was named `nginx_vm`, so I'll just name it the same in `hosts` file.
 It should look something like this
 ```
 nginx_vm  ansible_host=192.168.0.50
 ```
Finally, `ssh-copy-id` you `ssh-key` to the `vm`.
{% highlight bash %}
~ » ssh-copy-id -i chuleh@192.168.0.50
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/chuleh/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
chuleh@192.168.0.50's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh 'chuleh@192.168.0.50'"
and check to make sure that only the key(s) you wanted were added.
{% endhighlight %}

Doing all of that was a pain, wasn't it? Well, those days are gone with `Ansible`. Let's check some `ad-hoc commands` before moving onto `playbooks` for automation.

Ad-hoc commands
===============
What are `ad-hoc` commands? They're commands you run once and you forget about them. These are used for doing tasks on the remote `vm` from your `cli`.

The syntax is as follows: `ansible <host> -m <module> -a "<arguments>"`.
Let's see an example. In the following, I will ping the `vm` to see if it's online:
{% highlight bash %}
~ » ansible nginx_vm -m ping
nginx_vm | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
{% endhighlight %}
Cool! The `vm` is responding to our ping. We used the `ping module` and the `vm` replied back to us. Now how about doing some more fun stuff. Let's install `apache` on the remote `vm`. For this, we're going to use the `apt` module. If your `vm` is running `CentOS`, you can use the `yum` module.
Since this `module` requires `root` privileges, we will use the `--become` option.
{% highlight bash %}
~ » ansible nginx_vm -m apt -a "name=nginx state=present" --become
nginx_vm | SUCCESS => {
    "cache_update_time": 1528839938,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],

    [...]

{% endhighlight %}

I changed my mind. Let's remove `nginx` and install `Apache`. For this, we will tell the `vm` we want `nginx` to be `stopped` with the `service` module. And then we will tell the `vm` we want it `absent`.
{% highlight bash %}
~ » ansible nginx_vm -m service -a "name=nginx state=stopped"
nginx_vm | SUCCESS => {
    "changed": true,
    "name": "nginx",
    "state": "stopped"
}
~ » ansible nginx_vm -m apt -a "name=nginx state=absent" --become
nginx_vm | SUCCESS => {
    "changed": true,
    "stderr": "",
    "stderr_lines": [],

    [...]

~ » ansible nginx_vm -m apt -a "name=apache2 state=present" --become
nginx_vm | SUCCESS => {
    "cache_update_time": 1528839938,
    "cache_updated": false,
    "changed": true,

    [...]
   {% endhighlight %}

Let's check if this is true by running `curl` against our `vm`.
{% highlight bash %}
~ » curl -I 192.168.0.50
HTTP/1.1 200 OK
Date: Fri, 15 Jun 2018 17:47:00 GMT
Server: Apache/2.4.7 (Ubuntu)
Last-Modified: Fri, 15 Jun 2018 17:33:23 GMT
ETag: "2cf6-56eb19cdebd1a"
Accept-Ranges: bytes
Content-Length: 11510
Vary: Accept-Encoding
Content-Type: text/html
{% endhighlight %}

Ansible playbooks
=================
Running this every single time would be a hassle, wouldn't it? Well, here's where `playbooks` kick in. They are a way to automate, in a `yaml` file, all the `tasks` you want to run on a vm. In this playbook, we're going to install apache2 and start it, just like we did from the `cli`.
Let's go to our test directory and start hacking!

We will name this playbook `example.yml`
{% highlight bash %}
~ » mkdir test && cd $_
~/test » vim example.yml
{% endhighlight %}
We'll start our `playbook` with __three dashes__. Followed by the `host` where the playbook will run. Just like we did in the `cli`, we will use the `become` command to run as `root`.
{% highlight yaml %}
---
- hosts: nginx_vm
  become: True
  {% endhighlight %}
Now we will begin our `playbook` with a `task` command and then we will write the instructions to be executed.
{% highlight yaml %}
---
- hosts: nginx_vm
  become: True

  tasks:
    - name: run apt-get update
      apt:
        update_cache: yes

    - name: install apache2
      apt:
        name: apache2
        state: present

    - name: start apache2
      service:
        name: apache2
        state: started
        {% endhighlight %}
Finally, we will run our playbook with the `ansible-playbook` command.
{% highlight bash %}
~/test » ansible-playbook example.yml
{% endhighlight %}
If everything went correct your playbook should run and the output should look something like this:
```
PLAY RECAP *************************************************************************************************************************************************************************************************
nginx_vm                   : ok=4    changed=1    unreachable=0    failed=0
```


So we wrote our first playbook but let me be honest. This is just a glimpse of what `Ansible` can do. This playbook is rather useless. Why? Well, we have hardcoded our `hosts` so it will only run on our `vm`. But don't worry, stay tuned cause in the next post we're going to run `Ansible` with `roles` and `tags`.

:wq
