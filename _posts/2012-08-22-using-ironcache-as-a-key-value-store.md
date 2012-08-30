---
layout: post
title: Using IronCache as a Key Value Store
date: 2012-08-22
category:
tags: ['ruby', 'sinatra']
---

While IronCache is a great option for caching, it can also be used to persist data more permanently. There are dozens of great uses for a persistent key/value store, however, we'll be building a simple chat app with Sinatra using IronCache as our datastore.

First let's talk about a traditional cache. Take Memcached for example, it's an in memory, non-durable, key/value store. It's great! It's probably the most popular object caching system there is. Until now, Memcached has always been my go-to solution.

Initially I built my mini-chat app [Chattyloo](http://chattyloo.com) using Postgres. Postgres is a **big hammer**, for quite a simple problem. I know I needed persistence and durability, but I didn't need such a serious Object Relationship Mapper. Moreover I didn't want to manage a database, and that was the biggest factor to move over to IronCache.

Even though IronCache is an object caching system, it is at it's heart, MongoDB. But wait, isn't MongoDB's database write lock it's weakest point? True, it blows! But it's not MongoDB's fault, blame Spidermonkey or V8's single threaded process. The good thing is that this isn't an issue if you have multiple shards, the bad thing is that you have to have multiple shards. When MongoDB is setup with multiple instances and shards, it flies.

Now this is one relationship here, but it's actually an advantage in the case of a chat app. I needed to have one relationship, as chat rooms have chats. The way around this is to store all your chats as a single value for that room. I've created an ORM-like finder:

<pre class="prettyprint lang-ruby">
def self.find_or_initialize(name)
  if settings.ironcache.cache(name).get('messages').nil?
    self.new(name)
  else
    self.find(name)
  end
end
</pre>

Now this is a sinatra app so `settings` is a hash, and `ironcache` is just an instance of `IronCache::Client`, see here: https://github.com/jpsilvashy/chattyloo/blob/master/app.rb#L22

When a new chat is POST'ed, the we insert it like so:

<pre class="prettyprint lang-ruby">
# Inserts a new message into the channel
def insert(message)
  self.messages.push message
  self.messages = self.messages.last(settings.channel_message_limit)
  self.save
end
</pre>

The `last()`, trims off the messages so we don't grow the size of the object too large, as they are all loaded at once.

Lastly, chats are retrieved like so:

<pre class="prettyprint lang-ruby">
def self.find(name)
  @name = name
  @messages = JSON.parse(settings.ironcache.cache(@name).get('messages').value)

  Channel.new(@name, @messages)
end
</pre>

This finder will return the channel with the messages. Now in your views you can iterate over the `@messages` for a given instance of `Channel`.

If you want to check out the final product, head over to [chattyloo.com](http://chattyloo.com). The project is entirely open source on Github as well, check out [Chattyloo on Github](http://github.com/jpsilvashy/chattyloo).

