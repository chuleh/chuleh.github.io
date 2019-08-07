Hi, in this post we will continue with our `ansible playbooks` tutorials. We'll continue from our last lesson, that was the [intro to playbooks]({% post_url 2018-06-18-ansible-playbooks %}). If you haven't read that one yet, I suggest you go and read it before moving any further with this one
since most of the content we're going to see is related to the previous post.

So, in the last post I said we were going to cover to `templates`, `handlers`, `vars` and that's what we're going to do.
<!--more-->
In today's post we'll see:
- Templates
- Vars
- Handlers

Main role
=========
Let's continue from our [previous post]({% post_url 2018-06-18-ansible-playbooks %}). I suggest you check it out if you haven't yet, cause we're going to do stuff covered already there. For today's example I'll be using  the config file for my own website. We're going to create a new `role` called `nginx`. In this role, we're going to install `nginx`, create the configuration file for the site, create a symbolic link from `sites-available` directory to `sites-enabled` and finally `restart` nginx.
Of course, we're going to do all of this adding new stuff. We will be using `templates`, `variables` and `handlers`.

But let's not get ahead of ourselves, let's start with the `template`. Inside the `nginx` directory, let's create a `templates`, `handlers` and `vars` directory. If you haven't created the `nginx` directory, do it now. I will do all of this in my `tmp` directory.
{% highlight bash %}
/tmp » mkdir -p nginx/{tasks,templates,handlers,vars}
{% endhighlight %}
Now, `cd` into our `tasks` directory and let's create the `main.yml` file for nginx:

{% highlight yaml %}
---
- name: run apt-get update
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: install nginx
  apt:
    name: nginx
    state: present

- name: create config for site
  template:
    src: lcasaretto.com.j2
    dest: /etc/nginx/sites-available/{% raw %}{{ site.name }}{% endraw %}

- name: symlink new site to sites-enabled/
  file:
    src: /etc/nginx/sites-available/{% raw %}{{ site.name }}{% endraw %}
    dest: /etc/nginx/sites-enabled/{% raw %}{{ site.name }}{% endraw %}
    state: link

- name: check nginx config
  command: nginx -t
  notify: restart nginx
{% endhighlight %}

Everything should be pretty straightforward so far, until we hit the line where it says `template`. See it? That's how you declare a template. Let's take a deeper look into it.


Templates
=========
`Templates` is an ansible module that allows us to use our own text files, like config files, and replace them on the destination server.
`Templates` are processed by the `Jinja2` formatting language, meaning that this opens a whole scenario of variables that we can pass to our templates.
So, to declare a template file, you just type down `template`, the `source` and the `destination` on the remote server. Ansible is smart and will look for our template file in our `templates` directory.

If you paid attention, you'll see that it also says `{% raw %}{{ site.name }}{% endraw %}`, that's a variable and we'll cover it later.
As I said starting today's post, I'll be using my own website for today's example. So I'll create a file called `lcasaretto.com.j2`. Pay attention to the `j2` extension, that's our `Jinja2` template file.
{% highlight text %}
server {
  listen 80;
  server_name {% raw %}{{ site.name }} {{ site.serveralias }}{% endraw %};
  root {% raw %}{{ site.rootdir }}{% endraw %};
  access_log  /var/log/nginx/{% raw %}{{ site.name }}{% endraw %}-access.log;
  error_log  /var/log/nginx/{% raw %}{{ site.name }}{% endraw %}-error.log;

  gzip             on;
  gzip_comp_level  3;
  gzip_types       text/html text/plain text/css image/*;

}
{% endhighlight %}
So that's the configuration file for my website. Let's break it down a little bit. It says that it's going to listen on port 80. The `server_name`
has two variables: `{% raw %}{{ site.name }} {{ site.serveralias }}{% endraw %}` which we will cover next. The same goes for `access` and `error` log.

Variables
=========
Now it's time to cover the `variables`. Variables are just that, `vars`, pieces of text that can be reused making your automation easier.
Remember we created the `vars` directory? `cd` into it and create a `main.yml` file which will contain all of our variables.
{% highlight yaml %}
site:
  name: lcasaretto.com
  serveralias: www.lcasaretto.com
  rootdir: /mnt/efs/lcasaretto.com/_site/
{% endhighlight %}
As you can see, the file starts with `site:`. Why? Because the vars we named started with `site.variable`. So we have the `name`, `serveralias` and `rootdir` variables defined, which are the ones we were calling earlier in our `template` file.


Handlers
========
Now it's time to create the `handlers` file. So what's a `handler`? It's like regular task, but it only happens when there's a `notify` directive in the playbook.
We have already created the `handlers` directory, `cd` into it and create a `main.yml` file.
{% highlight yaml %}
---
- name: restart nginx
  service:
    name: nginx
    state: restarted
{% endhighlight %}
If you go `tasks/main.yml` you'll see that our last line has a `notify` calling this `handler`.


Now, all that remains is running `ansible-playbook`. In my case, it will return `ok` on some stuff because `nginx` is already installed, and the configuration files are present, as well. Also, I already have `lcasaretto1`  defined in my `/etc/ansible/hosts` file.
{% highlight yaml %}
» ansible-playbook site.yml -l lcasaretto1 -t nginx

PLAY [lcasaretto] ******************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************
ok: [lcasaretto1]

TASK [nginx : run apt-get update] **************************************************************************************************************************************************************************
changed: [lcasaretto1]

TASK [nginx : install nginx] *******************************************************************************************************************************************************************************
ok: [lcasaretto1]

TASK [nginx : create config for site] **********************************************************************************************************************************************************************
ok: [lcasaretto1]

TASK [nginx : symlink new site to sites-enabled/] **********************************************************************************************************************************************************
ok: [lcasaretto1]

TASK [nginx : check nginx config] **************************************************************************************************************************************************************************
changed: [lcasaretto1]

RUNNING HANDLER [nginx : restart nginx] ********************************************************************************************************************************************************************
changed: [lcasaretto1]

PLAY RECAP *************************************************************************************************************************************************************************************************
lcasaretto1                : ok=7    changed=3    unreachable=0    failed=0
{% endhighlight %}



So that's pretty much everything, make sure you go by each of the steps carefully and slowly since we have covered a lot of stuff.
Stay tuned for the next post, in which we'll cover how to make decisions with ansible.

:wq
