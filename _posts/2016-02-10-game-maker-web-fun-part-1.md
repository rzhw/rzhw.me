---
layout: post
title: "Game Maker web fun, part 1: The Server and the Docker"
---

I thought it might be time I took a stab at a long series of blog posts. In it, I'd like to share some notes trying to run a web server made with Game Maker, of all things. And we're not done yet: in Docker! Now, whether this might be useful is *completely* up for debate (who am I kidding, it's already veered straight into a negative scale of usefulness). But it sure might make for some fun experiences, and that's what counts.

In this part, I'll be doing an overview on the web server to run, and getting a suitable Docker container set up. To get through everything in this series, you hopefully aren't running in fear (if you weren't already) when I say "X server", "TCP", "Wine", or even "bash". It'll also be helpful to have a Game Maker background, and to have done a little experimenting with Docker. Well then, without further ado, let's dig in.

## Gm Webserver, the star of the show

To start off, we'll need something to run. Within the sphere of Game Maker, using it to write a server isn't uncommon at all. After all, that gets you a server for your multiplayer games. We can go with one of them, but when you think of what Docker's commonly used to run, it's not game servers, right? That's why we're going for a web server.

Saving us some effort, there happens to be an existing source-available[^oss] server. It's [Gm Webserver](http://gmc.yoyogames.com/index.php?showtopic=301447), complete with .gm6 extension, by the same person behind 39dll. Awesome! I've tested it with Game Maker 8.1 on Windows 10, and it works pretty great. Though, sadly Game Maker itself's a bit buggy[^gm8bugs].

### Structure

Let's first look at the structure of the file, to get an overview of what's going on. We've got:

- Objects
    - `objServer`, which acts like a main loop for the server, creating instances of
    - `objClient`, which acts like a thread for each TCP connection
- Scripts
    - *DllScripts*, for 39dll scripts
    - *ContentProcessing*, for a few content type handlers
    - Miscellaneous scripts under the root
- Constants
    - Numeric ids to represent content types
    - Server IP and port configuration
    - Upload speed limit
- Rooms
    - A 250x100 room with a single objServer instance

### Virtual hosts

You might notice if you've tried to run the server beforehand that it doesn't run out of the box. Well, turns out it's featured enough to have virtual hosts. Under the `domains/` directory, you'll find filenames corresponding to hosts. Their contents refer to the root path of files for that host.

For testing purposes, we want to edit `domains/unknown`, the unknown case. You'd likely want to point to the `websites/gmweb` directory. After that, hit the run button, and we're off:

{% include retinaimg.html src="/blog/images/gmwebserver.png" alt="Gm Webserver running on Windows 10" %}

### Content types

Let's talk about the content types a bit before we move on. They're various handlers that get called, depending on the file requested. Coming along with the .gm6 are HTML, octet stream, GML, and index handlers. These get set up in objServer's create event, which places them under `global.content[]`.

In objClient's Step event, the request URL is parsed for the filename. After that, it figures out the virtual host, and passes the filename over to `sendSuccess`. That script calls `getContentType` for the type, and with that info, can call the right "start" handler. Sending's chunked, so future steps call the "process" handler until there's nothing left to send. After that, the "end" handler's called, and you've got a nice page in the browser.

## Getting the star on stage

Alright. We're dealing with an .exe straight out of Game Maker, and Docker's for containerisation of Linux applications. Time for some Wine? (Remember: None of that "GameMaker: Studio" business here. We're not going to rely on some Linux export module. Besides, we're dependent on 39dll too.)

To get started with our base, we'll need a Docker image with Wine. One pre-existing one is [`suchja/wine`](https://hub.docker.com/r/suchja/wine). What makes it pretty good for our purposes is that we can route graphical output from Wine to an X server. Perfect for seeing our program launch, and seeing info on-screen (since we can't do console output).

### Setting up the prerequisites

To get our X output, we'll need `suchja/x11server` as well. We'll go ahead and grab them, though be prepared for a somewhat hefty download if you don't have their base images[^baseimage]. In a terminal with Docker, run:

    $ docker pull suchja/wine
    $ docker pull suchja/x11server

Next up is spinning up our X server. The command off the `suchja/wine` readme will do:

    $ docker run -d --name display -e VNC_PASSWORD=newPW -p 5900:5900 suchja/x11server

(`-d` puts the new container into the background and outputs the container id, and `-e` sets an environment variable for the container)

We should now be able to fire up a VNC client and point it to our (Docker) machine at port 5900. If you're on OS X or Windows using Toolbox and aren't familiar with Docker, that means getting the IP to your Docker VM. If you're on a Mac, you can follow it up by launching the inbuilt VNC client.

    $ docker-machine ip default
    192.168.99.100
    $ open vnc://:newPW@192.168.99.100:5900

At this point, we've got a blank screen with a cursor, which we'll be launching our Game Maker executable into. It's time to spin up Wine.

### The Wine container

To get started, you'll need to grab the path to Gm Webserver. If you're on OS X or Windows, Docker Machine [auto-mounts the Users directory](https://docs.docker.com/v1.8/userguide/dockervolumes/), so you might want to move it there. From there we can launch the Wine container, initialise the default prefix, and link in the directory. You'll might see some X errors, but they can safely be ignored. Let's go:

    $ docker run --rm -it --link display:xserver --volumes-from display -p 80:80 -v /Users/richard/Projects/gmWebServer:/home/xclient/gmWebServer suchja/wine /bin/bash
    xauth:  file /home/xclient/.Xauthority does not exist
    $ wine wineboot --init && ln -s ~/gmWebServer ~/.wine/drive_c/gmWebServer
    wine: created the configuration directory '/home/xclient/.wine'
    X Error of failed request:  BadValue (integer parameter out of range for operation)
      Major opcode of failed request:  130 (MIT-SHM)
      Minor opcode of failed request:  3 (X_ShmPutImage)
      Value in failed request:  0x140
      Serial number of failed request:  213
      Current serial number in output stream:  213

(`--rm` automatically removes the container after it exits, `-i` keeps STDIN open even when not attached, `-t` allocates a pseudo-TTY, `--volumes-from` mounts volumes from the given container(s), `-p` publishes container ports to the host)

And for the main event:

    $ wine c:\\gmWebServer\\gmWebServer.exe

Flipping over to our VNC client, we'll get the server, right?

{% include retinaimg.html src="/blog/images/gmwebserver-nod3d.png" alt="Gm Webserver failing to launch due to failing to initialise Direct3D" %}

Sadly not. We'll figure out a way around in the next part.

  [^oss]: I'm careful to say source-available and not open source, since there’s no explicit license.
  [^gm8bugs]: For the most part everything works just fine, but in object windows the list of events and actions can inexplicably turn invisible after a while. You can solve it by reopening the modal window, but it's definitely not trivial to do all the time. Okay sure, I could always use Studio, but where's the fun in that!
  [^baseimage]: At time of writing, `ubuntu:14.04` for Wine, and `debian:jessie` for the X11 server. That gets us about 740 MB and 200 MB respectively.
  [^whynotearlier]: If you try and link the directory in `docker run` to `~/.wine/drive_c/` beforehand, you'll run into `wine: /home/xclient/.wine is not owned by you` when running wineboot.
