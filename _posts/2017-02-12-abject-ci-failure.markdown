---
layout: post
title:  "Abject CI failure"
date:   2017-02-12 00:00:00 -0500
---

I had some free time, so I figured I'd look around for free <a href="https://www.thoughtworks.com/continuous-integration">CI servers</a>. I really believe in continuous integration, and I haven't addressed it so far because I just didn't feel I had that much actual integration going on. Generally, the stuff I've been doing is much more infrastructure-y than feature-y, so the tests would simply be a "does the framework start" sort of deal - which now that I'm thinking about it, probably would have been a good idea to do from the start. I was initially thinking I was going to have to deploy my own server, but <a href="https://travis-ci.org/">Travis CI</a> is free and integrates directly with GitHub (which is where I'm hosting the repository for this project). After bumbling around through their <a href="https://docs.travis-ci.com/user/languages/javascript-with-nodejs">documentation</a> having no idea what I was doing (I continue to have no idea what I'm doing) I got this far:

![First Travis CI failure]({{ site.url }}/images/TravisFailure.PNG)

It's not glamorous, but at least its running something. As with most software projects, getting something to fail is most of the hard part. Fixing it up so it succeeds is much easier once you have something failing consistently. I had no earthly clue what I was doing, and I didn't really want to spend my day reading documentation just to tackle this. After googling around a bit, I happened across a post <a href="https://medium.com/from-the-couch/angular-2-with-travis-ci-922040e01937">here</a> which was extremely helpful. Doing the ol' copy paste got me quite a bit further in the build:

![The more failing the better!]({{ site.url }}/images/MoreAbjectFailure.PNG)

Scrolling up through the thousands of lines of log output was enlightening. Turns out I haven't the faintest idea how to test Angular code. I haven't really worked with it much, so it's just one of those things I never got around to. I had generally ignored it, as I haven't gotten into anything particularly complicated client-side, but because of that I kind of just assumed `ng test` would pass. After all, I hadn't added any assertions or anything, so what is it that was failing?

Everything. Everything was failing terribly. I'm going to throw the brakes on trying to move forward with this project until I have some basic testing of both the client side and the server side. It looks like Travis CI will be an ally in fixing up this project before it gets to be unsalvageable.