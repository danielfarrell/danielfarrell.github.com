---
layout: post
title: "The pleasure and pain of environment variables"
description: ""
category: 
tags: []
---
{% include JB/setup %}

After reading [the config section](http://www.12factor.net/config) of The 12-Factor App I set about evangelising it in our office. We started using env vars for our config, which was great for keeping config out of code and making things clean and flexible. The pain came with trying to do development as a team and deployment with them. Let me explore it with you.

## Pleasures

Keeping configuration out of our codebases made things very flexible. Plus, there is something that just feels right about letting the deployment environment be in charge of those things.

## Development Pains

This pain is from keeping everyone on the same page with what config is required for the app to run. I add a feature that requires a new config variable, and then the next time a co-worker needs to work on that app it dies from him not setting that locally in his development environment. We started putting required variables into the README file, but turns out, despite it's name, no one reads it, it's a pain.

## Deployment Pains

And there are some deployment pains as well. We have some apps that run under Passenger, and we have to set those variables in the apache config. And we have other apps that deploy with a Foreman export to run under Upstart, which need the variables in a file at export time. Plus we have cron jobs that run command line programs for these apps that need the proper variables set for the shell. There are ways we worked around these issues, but they were a pain.

## Solution

After a co-dev went on a rant one day about the pain involved, I set out to come up with a solution to this. From that [redis_vars](https://github.com/danielfarrell/redis_vars) was born. It allows you to have a central store of environment variables on a redis server. If you get everyone on your team to use it and share a redis database(redistogo for distributed teams) it will solve the development pains. But you can also use it to have a central store of environment variables for production. We are moving to running our apps with unicorn, and starting them in production with redis_vars. One of our Procfile's looks like this:

    web: redis_vars bin/unicorn_rails -c config/unicorn.rb

Command line jobs run like this:

    cd path; redis_vars bin/rake cron:task

And everything is managed from a simple command line interface for adding/removing them.
