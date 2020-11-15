---
published: false
---
## Introduction

I've been meaning to start self-hosting a lot of the services I use on a regular basis by myself, but haven't really gotten around actually doing it. Every few months I would read an article about how companies are using data from users who use their products and it would make me want to switch, but it was enough of a hassle that I never actually took the plunge. Until this weekend.

A few days ago Google announced that they were removing the unlimited storage feature of Google Photos. I actually use that feature _a ton_. It also got me thinking of all the great [products](https://killedbygoogle.com/) that Google has also shut down over the years. If they decided to shut down Google Photos, Drive, Mail, etc. I would be ruined as basically all my digital data is backed up or stored in Google's ecosystem. It has been since I first created my Google Account for Gmail back when I was around 10 or so (with my parents' permission since I was under the age of 13, of course, if anyone at Google is reading this...)

However, it works quite well, and everything works cohesively together in a way you can't really duplicate with a mishmash of different products from different companies spanning different ecosystems (Google, Apple, Microsoft, etc). Another reason I was reluctant to self-host.

On top of that, there's the reliability and security aspect of it. I know my access to my account or products isn't going to go down very often if I'm hosted with one of these companies, since they all have at least three 9s of uptime, and it is very unlikely that they get hacked or lose my data since they have tons of very talented programmers and engineers working on these products to ensure everything stays safe.

I haven't really solved the cohesiveness issue, and only time will tell for the reliability (but i do have snapshots and backups in place), but I am pretty sure I have the kinks ironed out mostly for the security on on my own self-hosted stack. I'm happy so far with what I have setup, but I will look into expanding into more tools once I know my current setup is stable.

## Overview

I started brainstorming what tools I normally use on a daily basis. I looked for free self-hosted alternatives to them that I could easily deploy on my own. I have wanted to learn Docker for a while now, and this seemed like a good way to practice, so I decided to run all these apps from container images that already exist on DockerHub.

I wanted a nice way to manage my Docker containers from the web without having to login to my server each time to stop or restart a container, so I decided to use Portainer for that.

I also used NextCloud for a Google Suite alternative (Drive, Photos, etc). Bitwarden_rs is used for the password manager as a LastPass replacement. I went with Bitwarden_rs instead of the official Bitwarden build since it includes all the premium features for free and uses less system resources.

Everything is also setup to work behind an Nginx reverse proxy so I don't have to host all these services on different VPSes each.

I started with a Ubuntu 20.04 server, assigned my DNS with the Cloudflare CDN that I use for everything on my site, and got to work.

