---
title: Docker with Arch Linux and Raspberry Pi 2
tags: [docker, raspberry-pi-2, arch-linux]
date: "2015-03-02T03:30:33.000Z"
description: "A basic guide on how to use Docker with Arch Linux on a Raspberry Pi 2."
---

The new and faster Raspberry Pi 2 finally allows more sophisticated usage of services for home media and networking.
Combine that with the superfast [Docker](https://www.docker.com/) for better service management and [Consul](https://consul.io/) for health checks and you get an awesome system that is really fun to play with (and possibly a bit overkill ;) )!

I have used the first version of the Raspberry Pi for many tasks.
It was a print-server with cups, a media-center with minidlna and shared files with samba.
It is small, configurable and power-saving.
It also has tons of documentation available online.
The major downside of the Raspberry Pi v1 is its limited ressources:
If I use two services at the same time, it gets really slow with a 700Mhz ARMv6 processor.
Granted, with its 40€ price tag this is to be expected.
Now along comes the shiny new Raspberry Pi v2.
With a quad-core 900Mhz and 1GB of RAM it should be much faster than its predecessor.
The new processor also features the ARMv7 processor generation, meaning more software will be compatible with the new Raspberry Pi.

Basic setup
---------------------

###Arch Linux

Installing Arch Linux on the Raspberry Pi is pretty straightforward.
Just follow the [guide on archlinuxarm.org](http://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2).
If you have an older installation of Arch for the old Raspberry Pi, I recommend you create a fresh installation.
This way you dont need to use the armv6-packages (Possible dependency-hell) or upgrade them to armv7.

###Docker

To install Docker, login to your Raspberry Pi with SSH and type ```pacman -Syu docker``` to update the system and install docker.<br>
If you are logged in with a different user than root, you need to add yourself to the docker-group with ```gpasswd -a user docker``` and switch to the group with ```newgrp docker```.
Note that the last command will only affect the current session.
Now you only need to start docker ```systemctl start docker``` and optionally autostart it on boot with ```systemctl enable docker```.
Check if docker is up and running with ```docker info```.
Thats it!

Base image for Docker
---------------------

Since the Raspberry Pi 2 uses a different architecture than the old Raspberry Pi, the kernel differs, too.
This means that we cannot use already existing, prepackaged docker-images for Arch Linux like the repository [cellofellow/rpi-arch](https://registry.hub.docker.com/u/cellofellow/rpi-arch/).
Since there is no base-image for Arch Linux on the Raspberry Pi 2 (I still struggle to create a working and small automatically building image), we have to create it ourselves.

To do that, create a new directory on the Raspberry Pi to store the Dockerfile in.
Fetch the current Arch Linux archive with

    curl -L http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz -o archlinux.tar.gz

Then create a new file named ```Dockerfile``` with the following content:

    FROM scratch

    ADD archlinux.tar.gz /

    CMD ["/bin/bash"]

Running ```docker build -t base-arch-rpi2 .``` will extract the archive into the root of the new image named ```base-arch-rpi2```.

You can now start a container using the image with ```docker run -ti base-arch-rpi2``` or create children with ```FROM base-arch-rpi2``` in your Dockerfile.
