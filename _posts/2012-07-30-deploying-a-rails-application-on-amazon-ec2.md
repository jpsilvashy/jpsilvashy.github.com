---
layout: post
title: Deploying a Rails Application on Amazon EC2
date: 2012-07-30
category: 
tags: []
---

<div class='leader_note'>
	This is post 2 of 2, in my series on deploying Rails applications. <a href="/posts/setting-up-ubuntu-with-ruby-193-on-amazon-ec2/">Go to part 1 &rarr;</a>
</div>

This is a continuation of my previous blog post [Setting up Ubuntu with Ruby 1.9.3 on Amazon EC2](/posts/setting-up-ubuntu-with-ruby-193-on-amazon-ec2/), I realize that a lot of people have experience with Amazon EC2 so I decided to break it into several parts, that way if you're already comfortable navigating AWS's menus, you can zip through or just skip the first part.

Let's jump right in. We're going to deploy our rails app using [Capistrano](https://github.com/capistrano/capistrano). So in your local rails app, let's get capistrano setup. Let's add `capistrano`, to our `Gemfile` in our application.

<pre>
gem 'capistrano'
</pre>

Run the `bundle` command to update your bundle file, we'll talk more about Bundler and the Gemfile later on but for now it's important to just look at it as a dependency manager for your app.

<pre>
bundle
</pre>

Next we need to use Capistrano's `capify` command to generate the files that we'll need to get the app deployed on our server.

<pre>
$ capify .
[add] writing './Capfile'
[add] writing './config/deploy.rb'
</pre>

So you'll see that it created two files, we'll be focusing on `deploy.rb` for now.








Open up your terminal and SSH to your machine.

<pre>
ssh ubuntu@50.11.131.12
</pre>

First things first, lets make sure we have some basic stuff, you'll need to have `(bundler)[http://gembundler.com]` installed, this is really the only gem we'll install this way, as Bundler itself will manage all our other gems.

<pre>
sudo gem install bundler
</pre>

Make sure you have the `bundle` binary available from your shell.

<pre>
$ bundle -v
Bundler version 1.1.5
</pre>

