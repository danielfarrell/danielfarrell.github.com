---
layout: post
title: "Authentication as an Attached Resource"
description: ""
category: 
tags: []
---
{% include JB/setup %}

For some years we have been using a SSO authentication system here at work that was based on the auth_ntlm_winbind Apache module. It worked, but it tied our apps to Apache, and required some extra setup on the server. I wanted a system that would break authentication out of our apps and server, and would treat it as an attached resource.

## Options

The technologies I considered to solve this were along these lines:

1. ADFS/WS-Fed(Active Directory Federation Services)
2. SAML(SimpleSAMLphp)
3. Reverse Proxy with Kerberos authentication

## Solution

I started with ADFS, but we needed to upgrade our domain to make it work at that point. Then I looked at SimpleSAMLphp, and I got an experimental setup working on it(on our legacy Windows web server that we host old apps on) at one point. We had a few apps that were doing cross-origin calls, and the authentication strategies didn't seem to exactly fit us, so it got delayed.

I wrote a kerberos authenitcation reverse proxy based on Goliath as another option. But it still required a server setup piece, and then we had issues with Kerberos over our VPN(but not NTLM, don't ask), and that was the death of that solution.

I then went back to SimpleSAMLphp. I wrote a module for it(that will be open sourced at some point) to just use the Integrated Authentication from IIS as it's authentication. This means that any browser that supports NTLM/Kerberos SSO just sees a quick redirect back and forth on first loading one of our apps.  For those of us not using a supported system(ie, mac users who don't keep renewing their kerberos tickets) we only have a single site to save our username/password for instead of each individual site.

I know that treating authenitcation as an attached resource is a common strategy among those on the open web(as opposed to those of us on intranet), as FB and twitter are becoming the common authentication measures used. And for rubyists, [omniauth](https://github.com/intridea/omniauth) has made this super easy. I took this same approach to using SAML.  Getting SAML working on our apps was made pretty easy thanks to the [omniauth-saml](https://github.com/PracticallyGreen/omniauth-saml) gem. And because omniauth is great, switching to it also made our development user setup a lot better(previously it was based on an env var that you had to change and restart).

This setup let's us bring up new apps without having to worry about setting up any authentication.  It also will allow us to use a different web server(nginx is planned), and our chef scripts for a new server become simplied as well because we don't have to make auth_ntlm_winbind(and hence winbind, samba, etc) also work.

This treats authentication as an attached resource that we do not have to worry about in our apps, it's just a service that does it's thing.
