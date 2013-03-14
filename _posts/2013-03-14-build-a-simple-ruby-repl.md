---
layout: post
title: Build a simple Ruby REPL
date: 2013-03-14
category: development
tags: ['ruby', 'ubuntu', 'aws']
---

<div class='callout github'>
  Find the code for this post on <a href='http://github.com/jpsilvashy/repl'>on Github</a>. The application itself is deployed <a href='http://x-messages.herokuapp.com'>on Heroku</a>.
  {% include octocat.svg %}
</div>

## What is a REPL?

A REPL is a [Read–eval–print loop](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop). The term is most usually used to refer to a Lisp interactive environment, however many other languages have a similar mechanism, like IRB with Ruby for example.

Other nerds might call them an interactive toplevel or a language shell... but they should just knock it off, because that sounds dumb.

## Why would I want one?

Everyone has their go-to tools, especially in your regular system shell. Even though most developers have access to powerful tools with Rake, and Thor, you'll still often find yourself piping together `sed`, `grep`, and `awk` commands to do quick tasks.

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
