---
title: 'New gem: self-notifo'
enki_id: 5
tags: [ruby, rails, notifo, gems]
---
I've never created a Ruby gem before, although I use other people's gems a lot. So I decided recently that I might have a go at making one myself. I needed to find something nice and small to start off with.<!--more-->

### Some background

A while ago I came across a web startup called [Notifo](http://notifo.com/). They're one of the companies funded by the well regarded [Y Combinator](http://ycombinator.com/) venture firm. Notifo basically does one thing and does it pretty well as far as I can tell: they allow people to send notifications when stuff happens. You can join up to Notifo and create a 'service' that other Notifo users can subsequently sign up to and be notified when something happens. One example is the [push.ly](http://push.ly/) service that pushes a notification to you via Notifo when someone mentions your username on [Twitter](http://twitter.com/).

Notifo users need to download one of Notifo's apps in order to receive notifications. At the moment they have an iPhone app, an app that hooks into Growl for OS X and browser add-ons for Chrome, Safari and Firefox. I gather they are working on Android and Black Berry apps as well as something for Linux.

### Some foreground

I wasn't so much interested in the Notifo 'service' side of things. I just wanted to create a really simple way that I could send myself notifications (to my iPhone) from my own Rails apps when something interesting happens. Enter [self-notifo](http://github.com/gaelian/self-notifo). Full instructions for use are at the previous link. You can install with:

```bash
$ sudo gem install self-notifo
```

This gem in its current form is quite tied to Rails and the normal convention in this case seems to be that you would usually release your code as a Rails plugin rather than a gem. But for a number of reasons - not the least of which being that I just wanted to write a gem - I went with a gem rather than a Rails plugin.

If you're looking for something in Ruby that does take into account the Notifo 'service' side of things as well and isn't tied to Rails specifically, [try this library](http://github.com/jot/notifo).

**Disclaimer:** I have no affiliation with Notifo whatsoever, I just like the cut of their jib.
