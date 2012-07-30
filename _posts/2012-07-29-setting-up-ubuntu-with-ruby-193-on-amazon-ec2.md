---
layout: post
title: Setting up Ubuntu with Ruby 1.9.3 on Amazon EC2
date: 2012-07-29
category: 
tags: []
---

If you haven't deployed a Ruby application on Amazon Web Services EC2 yet, you're really missing out. It's largely a simple and enjoyable process. I'm always impressed at how quickly servers can be provisioned on EC2. This is a guide on how I typically get a new project going on EC2.

## Provisioning the Server on EC2

Head over to [aws.amazon.com](http://aws.amazon.com) to get started. Log-in then go to the [EC2 panel](https://console.aws.amazon.com/ec2/), click Launch Instance. Choose the Classic Wizard, scroll and choose Ubuntu Server 12.04 LTS (Note that it selects the 64 bit version by default, we'll use that one). LTS (Long Term Support) releases are always the better choice because the'll receive security and bug fixes for 5 years.

I typically choose to launch a Small Instance (m1.small) when I'm just getting a project started. A Small Instance will cost about $60/mo. Click continue, next you'll be on the Storage Device Configuration, I typically leave the volume size at 8GiB. If my server is going to have some sort of persistent datastore, like MySQL or MongoDB, I'd probably bump the size of the volume up accordingly.

Click Continue again, leave a name for your instance, I'm naming mine `blog`, and continue to the next step. I usually create a new key pair for each server just to have tighter control of access. It really depends on how your manage access to your servers. In our case, we'll name it `blog`, and download the key pair, don't lose this file, it'll probably be named `blog.pem`. Click continue.

Create a new Security Group. Let's just name it the same as the instance, blog (you're also required to leave a description). Since we'll be using a web server (HTTP), we'll need port `80` open, as well as `22` so we can SSH to the server.

The last page of the setup process is just a summary of the instance that's about to be provisioned, if everything looks good, *launch it!*

Before we get started let's assign our server an Elastic IP address, click Elastic IPs in the left navigation menu. Allocate a new address then associate it with your EC2 Instance. 

Next click Instances in the navigation menu, and take a look at your instance, it's state should now say **running**.

## Setting up the Server

Now fire up your terminal. Let's first configure our SSH client so we don't have to provide our credentials all the time. I usually keep all my key files in the same directory. So open up `~/.ssh/config`, mine looks like this for our blog project:

<pre class="prettyprint">
Host 50.11.131.12
  User ubuntu
  IdentityFile /Users/jpsilvashy/Projects/aws_keypairs/blog.pem
</pre>

Lastly before you can SSH to your new server change the permissions of the key pair file to 600.

<pre class="prettyprint">
chmod 600 /Users/jpsilvashy/Projects/aws_keypairs/blog.pem
</pre>

Next, SSH to the server as the `ubuntu` user.

<pre class="prettyprint">
ssh ubuntu@50.11.131.12
</pre>

Okay, you should be logged into the server now! Good job! Now it's time to get our hands dirty. We'll be using `apt-get`, to setup most of the packages we'll need. Let's first make sure we have all the latest updates for Ubuntu.

<pre class="prettyprint">
sudo apt-get update
sudo apt-get upgrade
</pre>

Next, let's get some of the basic packages that we'll need installed, this will help with the remainder of the setup.

<pre class="prettyprint">
sudo apt-get install git build-essential
</pre>

So to clear things up, that'll install Git, and the complication tools you'll need to build the other packages, like GCC.

## Installing Ruby

Lastly, we'll need to get Ruby setup, but before we can actually install Ruby there are a few basic libraries that you are likely to need.

<pre class="prettyprint">
sudo apt-get install openssl libreadline6 libreadline6-dev \
zlib1g zlib1g-dev libssl-dev libyaml-dev libxml2-dev \
libxslt-dev autoconf libc6-dev ncurses-dev
</pre>

That takes care of the basic dependencies, now we'll install Ruby **1.9.3**, now this point is really important, even thoght the package name in the apt repository is `ruby1.9.1`, you'll actually install Ruby 1.9.3. The reason it's named this is a little complicated, but be rest-assured nothing is wrong here and this is the correct name of the package to you want to install.

<pre class="prettyprint">
sudo apt-get install ruby1.9.1
</pre>

Lastly, be sure you have the currect version:

<pre class="prettyprint">
$ ruby -v
ruby 1.9.3p0 (2011-10-30 revision 33570) [x86_64-linux]
</pre>

There you go! If the version is `ruby 1.9.3p0` or newer, you're all setup.