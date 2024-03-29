---
layout: post
title:  "Xplornet and its confounded double NAT"
date:   2017-02-11 00:00:00 -0500
categories: jekyll update
---

# Contents
{:.no_toc}

*  
{:toc}

### Who is Xplornet ###

You can check [their site](https://www.xplornet.com/who-we-are/), but they're a nation-wide Internet service provider in Canada. They generally focus on rural areas and offer both satellite and fixed wireless products (previously 4G, but now LTE). If you have experience with satellite Internet, you're likely aware it's at just about the bottom of the list for desirable connections. Its typically expensive, has low bandwidth, restrictive data caps, and high latency. Contrast that with Xplornet's fixed wireless offerings which historically have been expensive, have relatively low bandwidth, restrictive data caps, but acceptable latency! We jumped at the opportunity to upgrade from 4G to LTE - the pitch was "Up to<sup>1,2,3</sup> 25 Mbps down and 1 up with a 500GB data cap for 99$/month". If you live in an urban area you may find that surprising, but that is in fact a pretty decent deal. We were paying about 10$/month less for 10/1 connection and a 100GB data cap. So we switched (eventually, after Xplornet's labored roll out).

### One static IP, please ###

A couple months after switching, I wanted to expose a small web server as part of a hobby project. I looked at the Xplornet website and found that they offer static IP addresses - [see](https://www.xplornet.com/our-internet-packages/service-add-ons/static-ip-addresses/)? They seem to have forgotten to mention that static IPs were not something they could offer with their LTE service, only with their now obsolete WiMAX. There was no amount of money (even the 10$/month was expensive enough...) which could purchase a static IP from Xplornet on their LTE equipment. So, how to host a server? The go to solution in situations like this is a [dynamic DNS](https://en.wikipedia.org/wiki/Dynamic_DNS)(DDNS) - for most purposes its functionally equivalent so long as the DNS is updated every time the IP address changes. I've personally used (and enjoyed) [FreeDNS](https://freedns.afraid.org/), but there are other providers like [NoIP](http://www.noip.com/). There are yet other providers that are not free, but I don't see a compelling reason to use them. So, problem solved, right? Update the DNS entries every time Xplornet issues a new IP address and we're off to the races! Not a chance. Xplornet provides users with an IP address through network infrastructure that involves a double NAT.

### What is a double NAT? ###

Someone who actually knows about it can explain it better (e.g. [here](http://www.practicallynetworked.com/networking/fixing_double_nat.htm)). Essentially, with finite IP addresses and an ever-growing number of devices, the number of devices that are directly accessible from the Internet is a small subset of all Internet connected devices. For many home Internet users, their ISP gives them a dynamically allocated IP address and that conventionally ends up associated with the user's router. The router provides the means to translate addresses for incoming and outgoing packets - this is where you get your computers IP address that usually looks like `192.168.1.10`. Machines can "see out" via the router, but public Internet users can only see so far as the router (wild generalizations happening here). This means that if you want to host a website or run a server that is accessible from anything other than your LAN, you need to work around the NAT that the router is offering you. Everyday users run into this with the concept of port forwarding for video games or other software products - the router sends traffic that shows up on a given port directly to the target computer.

The issue with what Xplornet does is that it has its own router that serves many users routers. One public IP address hits Xplornet's router, is split into multiple private subnets (one layer of NAT), and those subnets are provided to end users who then use NAT (the second layer) to route to all the devices in their house. Generally, this doesn't cause any issues and makes better use of the increasingly scarce IPv4 addresses, but if you actually want to have anything visible from the public Internet, it's a nightmare. There's no direct connection from the equipment that in/around your house to the public Internet. So without cooperation from Xplornet, there is *no way* to accomplish the same effect as a static IP address with similar methods.

Feel free to check out other people running into this problem (and far more people fundamentally misunderstanding it) over on the [RedFlagDeals site](http://forums.redflagdeals.com/how-do-i-get-around-double-nat-network-situation-1857549/).


### What to do? ###

A double NAT means you can't use a DDNS or port forwarding or any of the other common suggestions people have for resolving what they think the issue is. So what are the options?

#### Free ####

What I believe to be the easiest is a "local tunnel" solution. There are plenty out there, one of the more popular is [ngrok](https://ngrok.com/). It has a [free tier](https://ngrok.com/product#pricing) and as long as you don't need multiple ports on a single domain (like I did), it gets you 4 tunnels and is actually super handy. Something as simple as:

{% highlight bash %}
ngrok http 80
{% endhighlight %}

Gets you:   

{% highlight bash %}
ngrok by @inconshreveable                                                 
                                                                          
Session Status                online                                      
Version                       2.1.18                                      
Region                        United States (us)                                               
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://d75b33b2.ngrok.io -> localhost:80
Forwarding                    https://d75b33b2.ngrok.io -> localhost:80
                                                                          
Connections                   ttl     opn     rt1     rt5     p50     p90 
                              0       0       0.00    0.00    0.00    0.00
{% endhighlight %}

Similarly, [Pagekite](https://pagekite.net/) is easy to get up and running. The difference there is the free tier of ngrok doesn't expire, while Pagekite's does.

Neither of these ([so far as I know](https://github.com/pagekite/PyPagekite/issues/59)) offers any way to get many ports on the same domain (or subdomain). You can have many ports fed from a single machine, but they're made accessible on separate subdomains. This part didn't work for me.
<br><br>
#### As close as you can get to free ####

I think there are two workable options. One would be to set up a publicly accessible [VPN](https://en.wikipedia.org/wiki/Virtual_private_network) server, the other is to set up a publicly accesible SSH server. All else held equal, I imagine the VPN server would be the best option, but I've never done that before. SSH servers are present by default on many Linux distributions, so most of the work is done.

With that said, my answer here is: reverse SSH tunnel. Cloud computing has driven the price of [VPSs](https://en.wikipedia.org/wiki/Virtual_private_server) into the ground. Whether it be with [DigitalOcean](https://www.linode.com/), [AWS](https://aws.amazon.com/), or [Linode](https://www.linode.com/) (see comparison [here](https://joshtronic.com/2016/12/01/ten-dollar-showdown-linode-vs-digitalocean-vs-lightsail/)), you can pick up a pretty beefy machine for 10$ per month of runtime. DigitalOcen even offers a 5$ tier, which is pretty astounding. With that taken into consideration, it's cheaper (or nearly the same price) to get a cloud instance than it is to pay for something like ngrok or Pagekite. You need barely anything computing resource wise to run this kind of set up.

There are two pieces to this, I'll refer to them as the "public computer" (cloud instance") and the "private computer" (the one you're wanting to expose).

#### Public computer ####

The public computer set up is pretty straightforward:

1. Already have, or sign up for a cloud instance with [DigitalOcean](https://www.linode.com/), [AWS](https://aws.amazon.com/), [Linode](https://www.linode.com/) or similar

1. Provision the machine with some kind of Linux (e.g. Ubuntu 16.04)

1. Know enough about securing a publicly accessible machine that you don't need to be told how to (you were already exposing something to the internet, so presumably you have some idea). You can look at [this guide](https://www.linode.com/docs/security/securing-your-server) for a quick intro.

1. SSH into it `ssh user@cloud-instance-ip.totallyreal.com`

1. Make sure it's up to date - `sudo apt update && sudo apt upgrade`

1. Here's the tricky part. It took me more than an hour to find out why I could only see the tunneled ports from the remote machine. Feel free to read up on the [documentation](https://www.freebsd.org/cgi/man.cgi?sshd_config(5)). You have to open up your cloud machine so that the SSH tunnels are exposed to the world at large (hopefully not everybody, you have a firewall set up - right?). This is possible by modifying `/etc/ssh/sshd_config` - note, you'll likely have a `ssh_config` as well. That file is for the cloud machine's clients, we need to change the settings for the cloud machine as a server - `sshd_config`. Anyways,

    1. `sudo sed -i '$a GatewayPorts clientspecified' /etc/ssh/sshd_config`
    1. Restart the SSH server: `sudo service ssh restart` 
    <br><br>

And that's it. The public computer is ready to act as a gateway to your private computer. All it needs to do is stay accessible.

#### Private computer ####
    
1. First things first, you're going to want to ensure your private machine can communicate with the public machine securely. The easiest way to do this is with [SSH keys](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys) - they'll allow the computers to connect without having to type in a username and password everytime. If anything here is confusing, follow the better written guide [here](https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys).

    1. Ensure you have SSH installed (as well as autossh, we'll be using that): `sudo apt install ssh autossh`
    1. Ensure you're logged in as the user you'd like to connect to the public computer with
    1. Create a private/public key pair to use for authentication with the public computer
        1. `ssh-keygen`
        1. You can use the defaults for all the prompts or change them as you wish
        1. This will yield a key pair - `~/.ssh/id_rsa` (the private part that you never share with anyone) and `~/.ssh/id_rsa.pub` (the public part, designed to be shared)
    1. Add the public key that corresponds to your private machine to the "authorized keys" on the public machine
        1. `ssh-copy-id user@cloud-instance-ip.totallyreal.com`
2. You can now test the connection with your own configuration, for me it was running on port `7080`:
    1. `ssh -R "[::]:7080:localhost:7080" -N root@23.239.2.79`
    1. This sets up a reverse tunnel from the public machine's port `7080` to the private machine's port `7080`
3. Using SSH directly works, but for a more robust and persistent connection we'll use [auttossh](http://www.harding.motd.ca/autossh/). It's basically just a managed SSH connection, perfect for this use case.
    1. To make this easy, we'll use a [SSH client config file](https://linux.die.net/man/5/ssh_config). In `~/.ssh/config`, put the details of your connection. Mine, for example, looked something like this:

{% highlight bash %}
HOST unifi-video-tunnel
    HostName            li123-45.members.linode.com
    User                toby
    Port                22
    IdentityFile        /home/toby/.ssh/id_rsa
    ServerAliveInterval 30
    ServerAliveCountMax 3
    RemoteForward       :7080 localhost:7080
    RemoteForward       :7443 localhost:7443
    RemoteForward       :7445 localhost:7445
    RemoteForward       :7446 localhost:7446
{% endhighlight %}

4. To add yet another layer, everything will be easier with a service wrapping around the autossh session. In Ubuntu 16, that would be a systemd service.
    1. Add a `.service` file that reflects the application to `/etc/systemd/system/` - I called mine `autossh-unifi-video.service`
    1. Flesh out the service to invoke autossh for you

{% highlight bash %}
[Unit] 
Description=Make Unifi Video available remotely
After=network.target 

[Service] 
User=toby
ExecStart=/usr/bin/autossh -M 0 -N unifi-video-tunnel

[Install] 
WantedBy=multi-user.target]
{% endhighlight %}    

5. As with any service, get it moving:
    1. `systemctl daemon-reload`
    1. `systemctl enable autossh-unifi-video.service`
    1. `systemctl start autossh-unifi-video.service`

And that's pretty much it! The reverse tunnel should be working, and everything you've mapped should be exposed to the public.

### Improvements

- Add a separate user for autossh
- Use a VPN solution instead of this
