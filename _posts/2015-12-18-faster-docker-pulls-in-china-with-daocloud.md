---
layout: post
title: Faster Docker pulls in China with DaoCloud
---

## Background

I've been recently spending a little of my spare time trying out Docker.
As it happens, part of that spare time's been in China, where `docker pull`
becomes just a _little_ bit slow[^dockerslow].

Given the nature of what we have, a local Chinese mirror for the Docker Hub repo
would be really nice to have, right[^mirror]? Some web searching gets me to
[talk slides](http://www.slideshare.net/Docker/dockercon-sf-2015-docker-community-in-china)
about the Docker community in China. One of the slides mentions a mirror -- bingo!

The mirror's provided by DaoCloud, a Chinese cloud services provider. Helpfully,
they provide it [for free](http://blog.daocloud.io/daocloud-mirror-free/)!
The catch: you'll need an id for the mirror, but that doesn't take much time.

## Getting your DaoCloud mirror id

First head over to the [sign up page](https://account.daocloud.io/signup).
Without an understanding of Chinese, you can probably guess what each field is
meant to be for. Provide a username, email, password, and fill out the CAPTCHA,
and you're done (password lengths are capped at 30 characters, which wasn't
immediately obvious).

![DaoCloud signup](/blog/images/daocloud-signup.png)

You'll be then taken to the dashboard, and receive an account verification email.
Just head straight over to the [mirror section](https://dashboard.daocloud.io/mirror).
As of writing they call it "Docker 加速器 2.0", or "Docker Accelerator 2.0",
and have a neat little hero thing.

![DaoCloud mirror 2.0 home](/blog/images/daocloud-mirror2-home.png)

We're here to skip the extra download of their DaoCloud Toolbox (unless it
interests you), though. Head over down to those tabs where it says
"2.0 操作手册" / "1.0 操作手册", and flip it over to 1.0, and you'll see your id
in the code snippets.

![DaoCloud mirror 1.0 instructions](/blog/images/daocloud-mirror2-instructions1.png)

Grab it, and let's keep going (or you can use their instructions if you prefer).

## Setting it up

### Docker Toolbox and boot2docker

If you're using Docker through Docker Toolbox or `boot2docker`[^boot2docker],
get to your Docker machine (e.g. `boot2docker ssh`, `docker-machine ssh default`,
Docker Quickstart Terminal), and `sudo vi /var/lib/boot2docker/profile` (or
use your favourite editor).

Then append the below line, replacing `YOUR_ID` with the id you found,
modifying your existing `EXTRA_ARGS` param if you have one:

    EXTRA_ARGS="--registry-mirror=http://YOUR_ID.m.daocloud.io"

Save and restart your machine (e.g. `boot2docker restart`,
`docker-machine restart default`) and get back in once it's up. A quick
`docker pull ubuntu` and we're getting much faster speeds here!

{% include retinaimg.html src="/blog/images/docker-pull-suchja-wine.png" alt="A Docker pull with the DaoCloud mirror" %}

*This was just minutes in. Not pictured: How excruciatingly slow it was not long ago.*

### Linux installations

Generally your configuration would be in `/etc/default/docker`. Instead of
`EXTRA_ARGS`, set your `DOCKER_OPTS`:

    DOCKER_OPTS="$DOCKER_OPTS --registry-mirror=http://YOUR_ID.m.daocloud.io"

Restart your Docker service (e.g. on Ubuntu `sudo service docker restart`),
and likewise a quick `docker pull ubuntu` gets you on your way!

## An alternative approach?

My searches also turned up [this answer](http://stackoverflow.com/a/28958125/77922)
on Stack Overflow. There DockerPool is also suggested, which you can use like this:

    docker pull dl.dockerpool.com:5000/ubuntu:14.04

But saving those extra keystrokes is always paramount, so sticking with a
registry mirror it is. Hope that helps you save a good chunk of time!

  [^dockerslow]: A quick peek at Little Snitch shows a hostname ending in `us-east-1.aws.dckr.io`. It's in the US, and it's AWS. No surprises there.
  [^mirror]: You would [really, really](http://researchcenter.paloaltonetworks.com/2015/09/novel-malware-xcodeghost-modifies-xcode-infects-apple-ios-apps-and-hits-app-store/) hope so. But we're not dealing with random downloads.
  [^boot2docker]: Some trivia: I decided to first follow the instructions from DaoCloud, and went to the Mac OS tab, which uses `boot2docker`. That didn't exist. Turns out that's because with the release of Docker Toolbox, it's [been replaced](https://blog.docker.com/2015/08/docker-toolbox/) with Docker Machine. The commands are [similar to boot2docker](https://docs.docker.com/engine/installation/mac/#migrate-from-boot2docker), except you need to name your Docker VM (aptly defaulted to `default`). So `docker-machine start default | docker-machine ssh default ... | docker-machine restart default` pretty much gets the same results. As it turns out though, the instructions had another tab called "Toolbox". Doh!
