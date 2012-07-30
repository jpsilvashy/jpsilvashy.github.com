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

## Setting up Capistrano

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

So you'll see that it created two files, we'll be focusing on `deploy.rb` for now. Let's just remove everything in that file and start from scratch. First let's get the application name in there:

<pre>
set :application, "blog"
</pre>

<pre>
set :scm, :git
set :repository, "git@github.com:jpsilvashy/blog.git"
set :scm_passphrase, ""
</pre>

Also remember we're just using the `ubuntu` user, in a more serious situation I'd probably create a user just for deploying the app, this would allow us to sort of "silo" the users activty and access to stopping and starting other services, or worse running some malicous code on the server.

<pre>
set :user, "ubuntu"
</pre>

Since, in our case, the server will house the application itself, the webserver, and the database, we have to tell capistrano that all our tasks related to deploying these services are on the same server, let's just say the domain name we have is "my_cat_blog.com", you could also use your IP address as a string here. We'll also add the path to where the app should be `deployed_to`.

<pre>
server "my_cat_blog.com", :app, :web, :db, :primary => true
set :deploy_to, "/var/www/blog"
</pre>

## Testing Out Our Deploy Recipe

Let's see if our recipe works! From your project root on your local machine, run:

<pre>
cap deploy:setup
</pre>

This will connect to the server, and create all the directories you'll need when we actually deploy the app. Next we need to check to see if the permissions are okay with the directories in question, let's run:

<pre>
cap deploy:check
</pre>

You'll probably run into a few issues here, when setting up my server from scratch, this is what I recieve:

<pre>
The following dependencies failed. Please check them and try again:
--> You do not have permissions to write to `/var/www/blog'. (50.11.131.12)
--> You do not have permissions to write to `/var/www/blog/releases'. (50.11.131.12)
--> `git' could not be found in the path (50.11.131.12)
</pre>	

Let's take care of fixing the permissions for those files first. SSH to your server.

<pre>
ssh ubuntu@50.11.131.12
</pre>

Go to the directory you've set your `deploy_to` path. There we'll change the owner and group of the folders that threw the errors.

<pre>
cd /var/www/
sudo chown -R ubuntu ./
sudo chgrp -R ubuntu ./
</pre>

Okay! The permissions should be okay, now lets install Git.

<pre>
sudo apt-get install git
</pre>

Now on your local machine run `cap deploy:check` again, you should recieve something like this:

<pre>
You appear to have all necessary dependencies installed
</pre>

## Deploying Your Code

So we've almost got our application code deployed to our new web server, however, there are several missing pieces. There are a couple overarching issues.

- we have no web server
- we have no database

Before we address the web sever or database let's just get our ruby code on our server. That way, as we work on our application, we can continuously deploy the app. This deployment methodology is sometimes referred to as ["Deploy Early and Often"](http://programmer.97things.oreilly.com/wiki/index.php/Deploy_Early_and_Often). After this step, and the next post in the series we can get our web server and database running.

On your server you'll need to have `(bundler)[http://gembundler.com]` installed, this is really the only gem we'll install this way, as Bundler itself will manage all our other gems.

<pre>
sudo gem install bundler
</pre>

Make sure you have the `bundle` binary available from your shell.

<pre>
$ bundle -v
Bundler version 1.1.5
</pre>

Then we need to generate an SSH key, we'll give that to Github so our server can pull the latest version of our application code when we deploy the application.




