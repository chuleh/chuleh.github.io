Hi, in this post we're going to talk a little bit about [Vagrant](https://www.vagrantup.com/){:target="_blank"}.
So, what is `Vagrant`? Well, as the website says, `Vagrant` is a tool for building and managing `virtual machines` (`vm` from now). What does this mean? Let's say you want to fire up a new `vm`, you'd go through all the hassle of downloading an .iso from the OS you wanted. Then you'd open up [VirtualBox](https://www.virtualbox.org/){:target="_blank"}, create a new `vm`, select the network interface, and just click next, next, next and next again.

Well, here is where `Vagrant` kicks in. <!--more-->Vagrant let's you automatize this process by writing a `Vagrantfile`. In this `Vagrantfile` we set up our new `vm` and tell Vagrant how we want it to be deployed through a set of parameters.

Ok, time to get our hands dirty. First well need two things:
- Vagrant
- VirtuaBox

Installing Vagrant
==================
I assume you have `VirtualBox` installed already. If not, go to [VirtualBox](https://www.virtualbox.org/){:target="_blank"}, download it and install it. Next, we're going to install `Vagrant`. Enter [here](https://www.vagrantup.com/downloads.html){:target="_blank"} and download the binary that suits your OS.
Also, `Vagrant` can be installed from the `cli` by using `yum install vagrant -y`, `apt-get install vagrant -y` or `brew install vagrant -y`.

Once Vagrant is installed, let's create a `Vagrant` directory with `mkdir Vagrant` and cd into it with `cd Vagrant`.
I already have a `Vagrant` directory, so I'm skipping the previous step :).

Creating a new VM with Vagrant
=============================
Enter [here](https://app.vagrantup.com/boxes/search){:target="_blank"} and you will see a list of `vms` we can deploy. For this example, I'm going to be using an `ubuntu` one. The one named __ubuntu/trusty64__. Make sure to remember the name of `vm` you chose, since we will use it in our next step.

Ok, so I will start my `terminal` and I will create a new directory inside our Vagrant directory and cd into it.
{% highlight bash %}
~/vagrant [vag-chuleh] » mkdir ubuntu64
~/vagrant [vag-chuleh] » cd ubuntu64
{% endhighlight %}
Inside this new directory, is where we're going to initialize `Vagrant`. We'll do this by typing `vagrant init ubuntu/trusty64`.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh] » vagrant init ubuntu/trusty64
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
{% endhighlight %}
If everything went correct, we should have our `Vagrantfile`. Now it's time to init our `vm`. We will do this by typing `vagrant up`.
This will take a while depending on your internet connection since it has to download the `vm`.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Box 'ubuntu/trusty64' could not be found. Attempting to find and install...
    default: Box Provider: virtualbox

    [...]

    default: Guest Additions Version: 4.3.36
    default: VirtualBox Version: 5.2
==> default: Mounting shared folders...
    default: /vagrant => /Users/chuleh/vagrant/ubuntu64
{% endhighlight %}
If everything went ok, your output should look something similar to the one above, which means we have a `headless vm` running ready to use. Now we can `ssh` into the new `vm` with the command `vagrant ssh`.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » vagrant ssh
[...]
vagrant@vagrant-ubuntu-trusty-64:~$
{% endhighlight %}
Now we're ready to do everything we want such as we would do with any `vm`.

Customizing the vagrantfile
===========================
But there's so much more we can than just firing up a new `vm`. Let's customize our `vagrantfile`.
Open up your `vagrantfile` in a text editor and let's start hacking.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » vim Vagrantfile
{% endhighlight %}
By default, `vagrant` runs your `vm` in `NAT` mode, so its `ip address` will look something like `10.0.x.x/24`. Let's change that, let's put it in `bridge mode` so it will have a ` static ip` matching our `LAN`.
Type the following in your `vagrantfile`:
{% highlight ruby %}
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "public_network", ip: "192.168.0.50"
 end
 {% endhighlight %}
 The first two lines you should already have them. The one we're adding is `config.vm.network "public_network", ip: "new static ip"`.
 If you don't know what your local `ip` is, type `ifconfig -a` in your terminal.
 Now we need to tell `Vagrant` about our changes. For that, we'll run `vagrant reload`.
 Now when you reload, your output should look something similar to this
 {% highlight bash %}
 vagrant/ubuntu64 [vag-chuleh●] » vagrant reload
==> default: Attempting graceful shutdown of VM...
==> default: Checking if box 'ubuntu/trusty64' is up to date...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Available bridged network interfaces:
1) en0: Wi-Fi (AirPort)
2) en5: USB Ethernet(?)
3) p2p0
4) awdl0
5) en3: Thunderbolt 1
6) en1: Thunderbolt 2
7) en4: Thunderbolt 3
8) en2: Thunderbolt 4
9) bridge0
==> default: When choosing an interface, it is usually the one that is
==> default: being used to connect to the internet.
    default: Which interface should the network bridge to?
{% endhighlight %}
I'm going to choose number `1` since that's my `nic`. But it could change on your end. Once we select our interface, `Vagrant` should continue to run. To check if the `static ip` was assigned, we can `ping` the ip.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » ping 192.168.0.50
PING 192.168.0.50 (192.168.0.50): 56 data bytes
64 bytes from 192.168.0.50: icmp_seq=0 ttl=64 time=0.449 ms
64 bytes from 192.168.0.50: icmp_seq=1 ttl=64 time=0.554 ms
^C
--- 192.168.0.50 ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 0.449/0.502/0.554/0.053 ms
{% endhighlight %}
Perfect, the `ip` is responding to our ping. But doing all that would be a hassle and we want to automate as much as possible. So, edit again your `vagrantfile` and append the following to `config.vm.network "public_network", ip: "192.168.0.50`.
{% highlight ruby %}
bridge: "your interface"
{% endhighlight %}
So your file should look something like this
{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "public_network", ip: "192.168.0.50", bridge: "en0: Wi-Fi (AirPort)"
end
{% endhighlight %}
Just like we did before, type `vagrant reload` for changes to take effect.

Provisioning apps to our VM
===========================
We can do all sorts of stuff to our `Vagrantfile`. Let's give this `vm` a name and let's install `Nginx` on it.
Edit your `Vagrantfile` and type the following
{% highlight ruby %}
  config.vm.provision :shell, path: "nginx.sh"
  config.vm.provider "virtualbox" do |v|
  v.name = "nginx_vm"
  end
{% endhighlight %}
Your config file should look something like this
{% highlight ruby %}
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "public_network", ip: "192.168.0.50", bridge: "en0: Wi-Fi (AirPort)"
  config.vm.provision :shell, path: "nginx.sh"
  config.vm.provider "virtualbox" do |v|
  v.name = "nginx_vm"
end
end
{% endhighlight %}
So now we're telling `vagrant` that we want to give a name to our `vm` and we also want it to run a script at startup. Let's create the `nginx.sh` script.
{% highlight bash %}
#!/bin/bash
apt-get update && apt-get install nginx -y
nginx -t
systemctl start nginx
systemctl enable nginx
{% endhighlight %}
Make sure to include the `-y` flag! Vagrant is not interactive, it will __fail__ if it needs user input. As usual, run `vagrant reload` for changes to take effect.

Destroying the VM
=================
Once you're done, destroying the `vm` is as simple as typing `vagrant halt` to stop it and `vagrant destroy` to destroy it.
{% highlight bash %}
vagrant/ubuntu64 [vag-chuleh●] » vagrant halt
==> default: Attempting graceful shutdown of VM...
vagrant/ubuntu64 [vag-chuleh●] » vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
{% endhighlight %}

And that's it. Your `vm` is gone.


This was just a quick view to all the stuff you can do with `vagrant`.
Hope you enjoyed it!

:wq
