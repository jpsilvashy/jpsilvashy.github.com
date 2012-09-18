---
layout: post
title: Changing your hostname on Ubuntu
date: 2012-06-20
category: development
tags: ['ubuntu']
---

If you're using Amazon AWS, your instance will likely have a hostname which matches the initial hostname and IP issued when you provisioned your server.

So on your server you'll need to update `/etc/hostname`.

<pre>
sudo vi /etc/hostname
</pre>

You'll see there is only one line in this file, go ahead and change it to the new hostname you'd like.

<pre>
myserver
</pre>

Then you'll also need to update `/etc/hosts`, under the first line which should look something like this, add your hostname and the localhost IP address.

<pre>
127.0.0.1 localhost
127.0.0.1 myserver

...
</pre>

Then I typically reboot the machine.

<pre>
sudo reboot
</pre>

When you log back in use the command `hostname` to check that your changes have taken effect.