---
layout: post
title: Chattyloo
date: 2012-08-04
category:
tags: ['ruby', 'sinatra', 'heroku']
---

Look, I know the name is stupid. There just aren't a lot of domains names available containing the word "chat", so I'm stuck with this one!

[Chattyloo](http://chattyloo.com) is an asynchronous chat app written in Sinatra and persisted with Postgres or SQLite using Datamapper. There is no authentication scheme. Users arrive at the index and are redirected to a chat room with a unique URL that can be shared for others to join the room.

It makes use of Server-Sent Events (SSEs) over WebSockets for simplicity. Go have a look at the source code! The app is entirely open-source, please help improve it.

<div class='callout github'>
  The source code for this application is available <a href='http://github.com/jpsilvashy/chattyloo'>on Github</a>.
  {% include octocat.svg %}
</div>

Besides the fact that it's very simple, the app does have a few bells and whistles. Firstly, different sounds play when chat messages are sent and received. Hearts and :)'s are automatically rotated using CSS3 transforms. Lastly, the title element displays a status message when new messages are received.

## Usage

First clone the repository:

<pre>
git clone git@github.com:jpsilvashy/chattyloo.git
</pre>

Run the `bundle` command at the root of the project:

<pre>
cd chattyloo/
bundle
</pre>

Then you should be able to start the server:

<pre>
./chattyloo.rb
</pre>

Then just go to [localhost:4567](http://localhost:4567), and start chatting!

## Deploying to Heroku

You can deploy your own Chattyloo app! Here are the instructions for deploying to [Heroku](http://heroku.com).

First in the root creat a new heroku app:

<pre>
heroku apps:create mychatapp
</pre>

Then add the addon for the shared database, this will create a Postgres database with all the proper environment variables needed to connect to it.

<pre>
heroku addons:add shared-database
</pre>

Now deploy the app to the server and go check it out

<pre>
git push heroku master
</pre>
