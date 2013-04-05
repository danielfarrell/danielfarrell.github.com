---
layout: post
title: "Running Unicorn with Upstart"
description: "Herd those unicorns"
category: 
tags: []
---
{% include JB/setup %}

If you are using unicorn to run your ruby apps then you've probably experimented with how to run it on your server.

## Pains

You've probably tried the the normal foreman export, but it's template is lacking in a couple of ways:

1. You must set your environment variables in the command string
2. You can't use a `initctl reload appname` because that is the sh that starts things

Your upstart script looks something like this:

    start on starting appname
    stop on stopping appname
    respawn

    exec su - app -c 'cd /var/www/appname; export PORT=5200; export REDIS_URL=redis://localhost:6379/2;  export RAILS_ENV=production;  export QUEUE=appname;  bin/unicorn -c config/unicorn.rb >> /var/www/appname/shared/log/web-1.log 2>&1'

## Pleasure

I've found a way around those pain points using start-stop-daemon.  My upstart script looks like this:

    start on starting appname
    stop on stopping appname
    respawn

    env RBENV_ROOT="/usr/local/rbenv"
    env PATH="/usr/local/rbenv/shims:/usr/local/rbenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    env PORT=5200
    env REDIS_URL=redis://localhost:6379/2
    env RAILS_ENV=production
    env QUEUE=appname

    exec start-stop-daemon --start -c app -d /var/www/appname -x /var/www/appname/bin/unicorn -- -c /var/www/appname/config/unicorn.rb >> /var/log/appname/web.log 2>&1

This gets the env definitions out of the startup string, and also gives upstart the proper pid to do a reload. Thought I'd share.
