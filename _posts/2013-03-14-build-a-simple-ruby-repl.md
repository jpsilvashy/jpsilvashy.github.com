---
layout: post
title: Build a simple Ruby REPL
date: 2013-03-14
category: development
tags: ['ruby']
---

<div class='callout github'>
  Find the code for this post on <a href='http://github.com/jpsilvashy/repl'>on Github</a>.
  {% include octocat.svg %}
</div>

## What is a REPL?

A REPL is a [Read–eval–print loop](http://goo.gl/dWURA) interactive environment, however many other languages have a similar mechanism, like [IRB](http://goo.gl/BdZ5M) with Ruby for example.

Other nerds might call it an interactive toplevel or a language shell... but they should just knock it off, because that sounds fucking annoying.

## Why would I want one?

Everyone has their go-to tools, especially in your regular system shell. Even though most developers have access to powerful tools like [Rake](http://rake.rubyforge.org/), and [Thor](https://github.com/wycats/thor), you'll still often find yourself piping together `sed`, `grep`, and `awk` commands to do quick tasks.

But for more involved tasks that I find myself doing more often I'll build a rake task. Sometimes these things just don't cut it and you need to open up your system in a more controlled way.

I like to build out my toolchain for each project or system I work on. Often times that means making a command line interface to my application, this is the ideal case for a REPL.

## Example

I'm going to break it out in to three parts, first a method to handle the input.

When we handle the input, we would probably run some methods deep in our application or build a mini DSL for your methods you'd want to run.

<pre class="prettyprint lang-ruby">
def handle_input(input)
  result = eval(input)
  puts(" => #{result}")
end
</pre>

This is a lambda that runs the content of the first block after the input is chomped.

<pre class="prettyprint lang-ruby">
repl = -> prompt do
  print prompt
  handle_input(gets.chomp!)
end
</pre>

After evaling and returning, fire up the prompt lambda again, this loops after every input and exits with exit or a HUP.

<pre class="prettyprint lang-ruby">
loop do
  repl[">> "]
end
</pre>

Again, the project is <a href='http://github.com/jpsilvashy/repl'>on Github</a>, fork it, let's make it more simple! Also this is going to be a part of a poker game that we'll be building over the next couple weeks.
