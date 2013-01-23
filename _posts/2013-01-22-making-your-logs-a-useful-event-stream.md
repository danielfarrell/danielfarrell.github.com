---
layout: post
title: "Making Your Logs a Useful Event Stream"
tagline: "Minimally invasive and still useful?"
category: 
tags: []
---
{% include JB/setup %}

In the [logs](http://www.12factor.net/logs) section of the 12-Factor App we see patterns of how to treat our logs. The first pattern is to seperate log handling from the application by logging to STDOUT. The second pattern is to treat our logs as an event stream instead of a file.

## Pleasures

The pleasure here is that we can keep log handling out of our app, and have a single place to view our event data.

## Pains

Turns out that the combination of the two patterns is tough to pull off. If you cared just about the first pattern you can do a simple logger setup and then have your deployment environment send those to a file. If you just cared about the second pattern you could use for something like [MongoDB Logger](http://mongodb-logger.catware.org/) to get nice log handling. But the combination could be a pain.

## Solution

I was looking for something minimally invasive into our apps that would enable both of these patterns to work together. What I came up with was [Common Event Formatter](https://github.com/danielfarrell/common_event_formatter). It is a very simple gem to just is an alternative log formatter. It makes your application logs be in the CEE/Lumberjack format that is an emerging standard. How is that a solution you ask? Well, your logs become a CEE json stream that you send to STDOUT. That is a format that can be used with the latest versions of rsyslog/syslog-ng, and for our purposes also works great with [fluentd](http://fluentd.org/). We are moving to running our apps in production using unicorn and upstart. They can now run like so:

    cd path; bin/unicorn_rails -c config/unicorn.rb 2>&1 | fluent-cat app.tagname

Our logs end up in a mongo database in the CEE format. We are building an app for viewing them that we plan on open sourcing when it is done. This combination seems to hit both the patterns well without being very invasive into your codebase or adding large dependencies to your app.
