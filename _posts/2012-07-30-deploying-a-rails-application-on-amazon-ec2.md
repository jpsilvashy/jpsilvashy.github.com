---
layout: post
title: Deploying a Rails Application on Amazon EC2
date: 2012-07-30
category:
tags: ['ruby', 'rails', 'ubuntu', 'aws']
---

<div class='callout'>
	This is part 2 of 3, in my series on deploying Rails applications.
</div>

This is a continuation of my previous blog post [Setting up Ubuntu with Ruby 1.9.3 on Amazon EC2](/posts/setting-up-ubuntu-with-ruby-193-on-amazon-ec2/), I realize that a lot of people have experience with Amazon EC2 so I decided to break it into several parts, that way if you're already comfortable navigating AWS's menus, you can zip through or just skip the first part.

## Setting up Capistrano

Let's jump right in. We're going to deploy our rails app using [Capistrano](https://github.com/capistrano/capistrano). So in your local rails app, let's get capistrano setup. Let's add `capistrano`, to our `Gemfile` in our application.

<pre class="prettyprint lang-ruby">
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

<pre class="prettyprint lang-ruby">
set :application, "blog"
</pre>

<pre class="prettyprint lang-ruby">
set :scm, :git
set :repository, "git@github.com:jpsilvashy/blog.git"
set :scm_passphrase, ""
</pre>

Also remember we're just using the `ubuntu` user, in a more serious situation I'd probably create a user just for deploying the app, this would allow us to sort of "silo" the users activty and access to stopping and starting other services, or worse running some malicous code on the server.

<pre class="prettyprint lang-ruby">
set :user, "ubuntu"
</pre>

Since, in our case, the server will house the application itself, the webserver, and the database, we have to tell capistrano that all our tasks related to deploying these services are on the same server. For now since we don't have our web server or database setup, just leave that value as `:app`, later on we'll add `:web`, and `:db`.

Additionally we need to provide a hostname, let's just say the domain name we have is "my_cat_blog.com", you could also use your IP address as a string here. We'll also add the path to where the app should be `deployed_to`.

<pre class="prettyprint lang-ruby">
server "my_cat_blog.com", :app, :primary => true
set :deploy_to, "/var/www/blog"
</pre>

[Here is the complete `deploy.rb`](https://gist.github.com/3205604). Remeber to update the variables to match your servers hostname or IP address.

## Testing Our Deploy Recipe

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

We want the default settings so when asked to enter a file in which to save the key, just press enter. In my case I'm not setting a passphrase for my keyfile, you may wish to do this if you are concerned with someone accessing your files and getting a hold of your keyfiles.

<pre>
ssh-keygen -t rsa -C "your_email@example.com"
</pre>

Since we're using Github (you don't have to use Github, you can host your code anywhere), we need to tell Github that the server is allowed to access my repositories.

<pre>
cat /home/ubuntu/.ssh/id_rsa.pub
</pre>

You should have file one long line that starts with `ssh-rsa`, and ends with your email address.

Let's go copy the files content to our clipboard and head over to [github.com/settings/ssh](https://github.com/settings/ssh), add the key and give it a name.

Now let's go back to our local computer and try to deploy our application code to the server. We'll try it `cold` first, this will basically mimic the actual deploy process, but not actual create any files or affect any services.

<pre>
cap deploy:cold
</pre>

The output should scroll by as though it were deploying the app and cloning the repository to the server.

<div class="callout tip">
	If you receive the error <code>Host key verification failed</code>, you might want to SSH to the remote machine and try cloning your repository from there first, this will ask you if you want to add Github to your list of known hosts. Choose yes, hopefully that solves the issue.
</div>

If everything seems to be working okay, let's go for it and deploy our code for real!

<pre>
cap deploy
</pre>

Everything should go smoothly this time, the last line of the output will probably state something like:

<pre>
`deploy:migrate' is only run for servers matching {:roles=>:db, :only=>{:primary=>true}}, but no servers matched
</pre>

But that's okay for us right now, we don't have a database setup yet! Let's login to our server and see if Capistrano did it's job.

<pre>
cd /var/www/blog
ls -la

lrwxrwxrwx 1 ubuntu ubuntu   37 Jul 30 09:01 current -> /var/www/blog/releases/20120730090056
drwxrwxr-x 4 ubuntu ubuntu 4096 Jul 30 09:00 releases
drwxrwxr-x 5 ubuntu ubuntu 4096 Jul 30 07:37 shared
</pre>

So you'll notice that the `current` directory is actually just a symlink to a timestamped folder name, this allows Capistrano to rollback to previously deployed code in case something goes wrong during the deployment.
