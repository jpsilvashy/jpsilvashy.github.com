---
layout: post
title: Ruby Poker Server on Sinatra (Part 1)
date: 2013-03-14
category: development
tags: ['http', ruby']
---

<div class='callout github'>
  An example application for this post is available <a href='http://github.com/jpsilvashy/poker-server'>on Github</a>. The application itself is deployed <a href='http://glacial-crag-1279.herokuapp.com'>on Heroku</a>.
  {% include octocat.svg %}
</div>

So this is part one in a series on building a mini web application as a service. We're going to make a multiplayer poker game server. Give it a try with curl right now (or I suppose you can use a browser like Chrome which should render the json response body in the browser, Firefox may ask you to download it). Let's start a new game:

<pre>
http://glacial-crag-1279.herokuapp.com/
</pre>

You'll get redirected to a url with a lady's name and some other random words. You'll also recieve the deck state with the face-suits as strings in an array. This was just a fun way to make semi-private games with your friends.

<pre>
http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport
=> ["7h","Ah","2s","9h","Ks","Kd","Ts","4s","Ac","Qd","3d","As","6s","Kc","5d","Tc","7c","4d","Jc","5c","8s","Kh","3h","7s","5h","3c","Td","6d","Qs","2h","4h","8d","Th","Jh","9c","6h","2c","3s","8h","4c","Jd","Js","9d","7d","Ad","2d","8c","5s","9s","6c","Qh"]
</pre>

[emily-fuscia-westport](http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport), good name, here is the deck of cards:

So it's important to recognize this is a simple API so we'll use curl to deal the deck and play an ad-hoc game. Deal out a hand:

<pre>
curl http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport/deal.json
=> "Qc"
</pre>

The result is a two character string representation like so:

You can also shuffle the deck:

<pre>
curl http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport/shuffle.json
</pre>

How many cards in the deck?

<pre>
curl http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport/size.json
=> 51
</pre>

Burn a card from the deck, send the card face-suit as the card param.

<pre>
curl http://glacial-crag-1279.herokuapp.com/emily-fuscia-westport/burn.json?card=Qc
=> ["7h","Ah","2s","9h","Ks","Kd","Ts","4s","Ac","Qd","3d","As","6s","Kc","5d","Tc","7c","4d","Jc","5c","8s","Kh","3h","7s","5h","3c","Td","6d","Qs","2h","4h","8d","Th","Jh","9c","6h","2c","3s","8h","4c","Jd","Js","9d","7d","Ad","2d","8c","5s","9s","6c","Qh"]
</pre>

After each command the current deck state will be returned, except for `deal`.

