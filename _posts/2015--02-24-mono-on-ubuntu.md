---
layout: post
title: Install Mono 3.12.0 on Ubuntu 14.10
tags: how-to mono ubuntu
---

To run BrightstarDB on Ubuntu you will need Mono > 3.2.4 installed. 
Ubuntu 14.10 does come with Mono 3.2.8, but even that is a way behind
the current state of Mono development and it is a good idea to instead
build and install the latest Mono version. Fortunately, this is not
that hard to do. The following procedure will get you up and running with 
the latest version of Mono (at the time of writing).

    # First make sure your base installation is up-to-date
    sudo apt-get update
    sudo apt-get upgrade
    
    # Install build-essential (may well already be installed, but worth checking)
    sudo apt-get install build-essential
    
    # You may want to run the remainder of these commands in a scratch directory
    
    # This grabs the current version of mono (3.12.0)  
    # Point your browser to http://download.mono-project.com/sources/mono
    # to check that it is still the most recent version.
    wget http://download.mono-project.com/sources/mono/mono-3.12.0.tar.bz2
    tar -xvf mono-3.12.0.tar.bz2
    
    cd mono-3.12.0
    ./configure --prefix=/usr/local
    make
    sudo make install
    
    # This should show that you now have the latest version of Mono installed successfully!
    mono --version
    

Once you are done and everything is built and installed you can delete both the downloaded .bz2 package and the source extracted from it.

One thing to note, if you want to automate this process, use `sudo apt-get -y` instead of 
`sudo apt-get` to automatically accept the changes that the apt-get process makes.