Hi, in this post we will take a more in-depth look at `Ansible playbooks`. We're going to continue from from where we left off in our [intro to ansible]({% post_url 2018-06-15-intro-to-ansible %}). If you haven't checked that post yet, I suggest you go and read it before moving any further with this one.

In today's post we're going to write a `playbook` as in the previous session, but we will enhance it. Rather than running it as a single playbook, we're going to create a `yml` file with all our hosts and run the `playbook` as a `role`.
<!--more-->

For today's lesson, I will be running 3 `vms` with `ubuntu`. The same ones from our [vagrant]({% post_url 2018-06-14-intro-to-vagrant %}) post.
They're already set up with my user and password, and I'm already included in the `sudoers` file.

Let's get our hands dirty. We're going to learn to:
- Edit our inventory
- Create a role
- Create a main includer for our hosts file
- Running our playbook

Editing the inventory
=====================
Open up your terminal and edit your `/etc/ansible/hosts` file. You should already have this file if you followed my previous [ansible]({% post_url 2018-06-15-intro-to-ansible %}) post.
{% highlight bash %}
~ » sudo vim /etc/ansible/hosts
{% endhighlight %}
Next, I'm going to create a `group` inside our inventory. I will name it `[home]` and add my `webservers` and `database` to it. I will use the `ansible_host` next to each of them, to specify the `ip address`. Your file should look something like this:
```
[home]
webserver1  ansible_host=192.168.0.20
webserver2  ansible_host=192.168.0.30
dbserver  ansible_host=192.168.0.40
```
Of course, change the `ip` address to match the ones you're using

To check that all the `vms` are responding, I'll just ping the `group` now, instead of each of the `vms`.
{% highlight bash %}
~ » ansible home -m ping
webserver1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
webserver2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
dbserver | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
{% endhighlight %}
Cool, right? Now we can run `ad-hoc` commands to the group and `ansible` will know what we're talking about.

Creating a role
===============
What is a `role`? Let's see what `ansible` says: Roles are ways of automatically loading certain vars_files, tasks, and handlers based on a known file structure. Ok, what does this mean? First it means that roles need a known __file structure__. That would be, of course, the `roles` directory, which is our today's topic.
So, as usual, let's create a `test` directory and inside we will create a `roles` directory. Finally, we will `cd` into it.
{% highlight bash %}
~ » mkdir -p test/roles
~ » cd test/roles
test/roles »
{% endhighlight %}
Now, we will create our new role. Let's try something different, let's create a role that will install the apps that don't come by default in `ubuntu`. I will call this role `apps`, so I will create the `apps` directory. Remember the definition said it needed a `tasks` in their file structure? Well, we'll also create a `tasks` directory, and inside of it we will create our playbook, which has to be named `main.yml`. Let's do it.
{% highlight bash %}
test/roles » mkdir -p apps/tasks && cd $_
apps/tasks » vim main.yml
{% endhighlight %}
Now it's time to start hacking, inside the `main.yml` we will write our playbook. As with every `yaml` file, it will start with __three dashes__. Then we will give our task a `name`, in my case I want to run `apt-get update` if it's been more than an hour. It should look something like this:
{% highlight yaml %}
---
- name: run apt-get update
  apt: update_cache=yes cache_valid_time=3600
  {% endhighlight %}
  See that we haven' defined `hosts` or `become` in our `role` like we did with our `apache2` playbook. That's because we're going to do it in our next step, when we create our main includer.

  Now it's time to install the apps, for this we will use the `apt` module and call the `item` variable. Next, we will define `with_items` that are going to be the `apps` that will fill our `item` variable. The playbook, should look something like this:
{% highlight yaml %}
- name: install apps
  apt: name={% raw %}{{ item }}{% endraw %} state=present
  with_items:
    - screen
    - rsync
    - htop
    - iotop
{% endhighlight %}
Mind the `curly braces` when you type it, otherwise it will fail. Our complete playbook should look like this:
{% highlight yaml %}
---
- name: run apt-get update
  apt: update_cache=yes cache_valid_time=3600

- name: install apps
  apt: name={% raw %}{{ item }}{% endraw %} state=present
  with_items:
    - screen
    - rsync
    - htop
    - iotop
 {% endhighlight %}
 We have successfully created our first role! Now it's time to run it.

Creating the main includer file
================
Why haven't we included the `hosts` directive or the `become` in our playbook? That's because we're not hardcoding anything into it. This means that we can use the same role in other hosts we might have in our inventory and it will just work! This is known as `reusable` code.

Now it's time to run our `playbook` with the `role` we specified. For this, we're going to create a file where we will specify the `hosts` and the
`tags`. `cd` back to the test directory and create a new `yml` file. I will call it `test.yml`.
{% highlight bash %}
apps/tasks » cd ../../..
~/test » vim test.yml
{% endhighlight %}
We'll start with our __three dashes__ and start typing our `hosts`, this has to be the same name you defined in your inventory. And we will tell ansible we want to become `root` with the `become` directive:
{% highlight yaml %}
---
- hosts: home
  become: True
  {% endhighlight %}
Now here is where the magic happens, let's define our `role` and `tag`. We will type `roles:`, press `enter` and define our role. This is a list, so __be careful__ with the indentation:
{% highlight yaml %}
  roles:
    - { role: apps, tags: ['apps'] }
    {% endhighlight %}
See how I defined our role `apps`? That's the role we just created. The `tag` I used was the same as the `role` name, but it could've been anything you wanted. Let me show you, I will add another `tag` and call it `watermelon`.
So this is is what my includer file looks like:
{% highlight yaml %}
---
- hosts: home
  become: True
  roles:
    - { role: apps, tags: ['apps','watermelon'] }
    {% endhighlight %}

Running our playbook
===================
Now it's time to run our playbook. We will use `ansible-playbook` command followed by `test.yml` which is our main includer file with all the hosts. Then we will use the `-l` flag which means `limit`, saying that we want to specify the `hosts` that will be affected by our playbook and the `-t` flag, which stands for `tag`. Remember I tagged it as `watermelon`? I'm going to run it like that.
{% highlight bash %}
~/test » ansible-playbook test.yml -l home -t watermelon

PLAY [home] ************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************
ok: [webserver1]
ok: [webserver2]
ok: [dbserver]

TASK [apps : run apt-get update] ***************************************************************************************************************************************************************************
changed: [webserver2]
changed: [dbserver]
changed: [webserver1]

TASK [apps : install apps] *********************************************************************************************************************************************************************************
ok: [webserver1] => (item=[u'screen', u'rsync', u'htop', u'iotop'])
ok: [webserver2] => (item=[u'screen', u'rsync', u'htop', u'iotop'])
ok: [dbserver] => (item=[u'screen', u'rsync', u'htop', u'iotop'])

PLAY RECAP *************************************************************************************************************************************************************************************************
dbserver                   : ok=3    changed=1    unreachable=0    failed=0
webserver1                 : ok=3    changed=1    unreachable=0    failed=0
webserver2                 : ok=3    changed=1    unreachable=0    failed=0ç
{% endhighlight %}
Perfect! `Ansible` ran without flaws. In my case the apps were already installed, so nothing's changed.

Now in this way we can specify the `-l` flag and affect only the resource we want. Go ahead and give it a shot, try running `ansible-playbook test.yml -l webserver1 -t apps` and see what happens. I will only affect `webserver1` leaving `webserver2` and `dbserver` untouched.

This has been a long post and we've gone through a lot of new stuff, so make sure to understand everything we wrote. Create new `roles`, add new `hosts` to your `inventory`, play with `ansible`, that's the best way to learn.

Hope you enjoyed it, stay tuned cause in the next session we will learn about `templates` and `handlers`.

:wq
