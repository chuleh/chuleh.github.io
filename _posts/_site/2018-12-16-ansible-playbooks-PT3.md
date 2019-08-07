Hi, in this post we're going to take a deeper look into `playbooks`. In our previous posts we covered our [intro to playbooks]({% post_url 2018-06-18-ansible-playbooks %}),
where we learned how to create simple `playbooks` and our `inventory`. In our [second post]({% post_url 2018-06-27-ansible-playbooks-pt2 %}) we learned about `templates`, `variables`
and `handlers`. We went over how my website is configured and took a look at its `template`.

So, in the last post I said we were going to tell `Ansible` how to make decisions, for that, we're going to spin up 2 `vms`. One with `CentOS` and another one with `Ubuntu`.
<!--more-->
For this, we're going to use `Vagrant`. I have already covered in [previous posts]({% post_url 2018-06-14-intro-to-vagrant %}) how to set up `vms` with `Vagrant`.
These two `vms` already have configured the user I will run `Ansible` with. In today's post we're going to install `Apache`. As you may know, in `CentOS` this package is called `httpd`
whereas in `Debian` related distros, this is called `apache2`.

Playbook
========
Now we're going to create the structure in which we're going to work. I will `cd` into my `/tmp/` directory and create the following directories: `tasks`, `vars`.
This is going to be a very basic example showing you just how to install `apache`, it won't include any `template`.
{% highlight bash %}
~ » cd /tmp
/tmp » mkdir -p roles/apache/{tasks,vars}
{% endhighlight %}


Tasks
=====
If you've followed my previous tutorials, you'll know here is where we're going to create the main file which is going to orchestrate our run.
For this, we're going to use the built-in variable `ansible_os_family`. Remember it is a `variable`, so it'll go between curly braces. The `name` can be anything you want.
I just like to put simple names to it so I'll name it `Ansible receiving variables`. So far, your file should look like this:

{% highlight yaml %}
---
- name: Ansible receiving variables
  include_vars: "{% raw %}{{ansible_os_family}}{% endraw %}.yml"

{% endhighlight %}

Now we'll use the `include` directive. Telling ansible we're going to include two files which will be located in our `files`directory.
Once again, `Ansible` is smart enough to know where to look for the files. We'll also use the `when` statement, telling `ansible` that this the `include` will happen when the `when` criteria
is met.
Your file should look something like this:
{% highlight yaml %}
---
- name: Ansible receiving variables
  include_vars: "{% raw %}{{ansible_os_family}}{% endraw %}.yml"

- include: debian_apache_install.yml
  when: ansible_os_family == "Debian"

- include: rhel_apache_install.yml
  when: ansible_os_family == "RedHat"
{% endhighlight %}

Include files
=============
Now it's time to create the two `include` files. I will start with `debian_apache_install.yml`. These two files are included inside the `tasks` directory.
{% highlight yaml %}
---
- name: Install debian packages
  apt: name={% raw %}{{ item }}{% endraw %} state=present
  with_items:
    - "{% raw %}{{ deb_packages }}{% endraw %}"
{% endhighlight %}

This should be pretty legible to you if you've been following my tutorials.
We're calling the `items` variable and I have created the `deb_packages` variable which we will define later in the `vars` directory.

Now it's time to create the `rhel_apache_install.yml` file.
{% highlight yaml %}
---
- name: Install RHEL packages
  yum: name={% raw %}{{ item }}{% endraw %} state=present
  with_items:
    - "{% raw %}{{ rhel_packages }}{% endraw %}"
{% endhighlight %}



Var
====
We have created the `include` files in the previous section, now it's time to create the `var` files. `cd` into the `vars` directory and let's start hacking.
We're going to create three files: `Debian.yml`  and `RedHat.yml`, the two we defined in our main `tasks` file. And we also need to create a `main.yml`, which will be empty.

Let's start with the easy one. Your `main.yml` is going to be empty, so just type the three dashes inside of it.
It's gonna look just like this
{% highlight bash %}
apache/vars » cat main.yml
---
{% endhighlight %}
Easy, just type in the three dashes and close the file.

Now we have to create the `Debian.yml` file. Inside, it will have the variable we defined previously and the package we want to install. Since it's debian, it will be `apache2`.
{% highlight yaml %}
---
deb_packages:
  - apache2
{% endhighlight %}

It's the RHEL file turn. Since this is RedHat, the package name will be `httpd`.
{% highlight yaml %}
---
rhel_packages:
  - httpd
{% endhighlight %}


Run the playbook
================
Now it's time to run the playbook. I already have my `inventory` created. This file includes the `hosts` and `tags`I will use to run `ansible-playbook`. I will not explain it, since
I've already covered this [here]({% post_url 2018-06-18-ansible-playbooks %}).
Let's take a look at the run.

{% highlight bash %}
tmp » ansible-playbook site.yml -l vagrant -t apache
[...]
PLAY [vagrant] *********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************************************************************************************************
ok: [webserver3]
ok: [webserver1]

TASK [apache : Ansible receiving variables] ****************************************************************************************************************************************************************
ok: [webserver1]
ok: [webserver3]

TASK [apache : Install debian packages] ********************************************************************************************************************************************************************
skipping: [webserver3]
changed: [webserver1] => (item=[u'apache2'])

TASK [apache : Install RHEL packages] **********************************************************************************************************************************************************************
skipping: [webserver1]
changed: [webserver3] => (item=[u'httpd'])

PLAY RECAP *************************************************************************************************************************************************************************************************
webserver1                 : ok=3    changed=1    unreachable=0    failed=0
webserver3                 : ok=3    changed=1    unreachable=0    failed=0
{% endhighlight %}

As you can see, it skips `webserver3` when it runs the `Install debian packages` part of the play and installs `apache2` on `webserver1`.
Then, it skips `webserver1` and installs `httpd` on `webserver3`.

This playbook should give you a solid foundation on how `Ansible` works when it has to make decisions. This playbook will also work if you rename it and add more packages, as for example `BIND`
which has different names depending on the distro you're running. Hack around and play with it.

Stay tuned, next playbook we'll see an example for a client I'm working with which includes `files`.

:wq

