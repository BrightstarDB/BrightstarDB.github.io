---
layout: post
title: BrightstarDB on the Raspberry Pi
tags: how-to, pi
---

Hands up all those of you with a Raspberry Pi sitting unused at home, this is one for you!

I decided to take a stab at getting BrightstarDB to run on one of the new Pi2s - these little
marvels have 1 GB of RAM as well as a quad core ARM CPU that should be more than enough to 
run a small BrightstarDB server and maybe build a little web application on top with something
like Nancy or perhaps Flask. In any case the first step is to install Mono. 

## Install Mono

I'm going to assume that you are running Raspbian (Arch linux users I may have another post or an
update to this post for you in the near future!). If that is the case, then installation is reasonably
easy as the Mono project already provide Debian packages. You really just need to follow 
[their installation instructions](http://www.mono-project.com/docs/getting-started/install/linux/#debian-ubuntu-and-derivatives)
which basically boil down to:

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
    echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get install mono-complete
    
We install ``mono-complete`` so that we have the mono compiler (mcs) and the
other tools that will be necessary to build BrightstarDB. You should now
be able to run mono to check the version:

    kal@techquilaapi ~/BrightstarDB/build $ mono --version
    Mono JIT compiler version 4.0.4 (Stable 4.0.4.1/5ab4c0d Tue Aug 25 23:45:14 UTC 2015)
    Copyright (C) 2002-2014 Novell, Inc, Xamarin Inc and Contributors. www.mono-project.com
            TLS:           __thread
            SIGSEGV:       normal
            Notifications: epoll
            Architecture:  armel,vfp+hard
            Disabled:      none
            Misc:          softdebug
            LLVM:          supported, not enabled.
            GC:            sgen

Hoorah! You are all set to build BrightstarDB!

## Add NuGet SSL Certificates

The BrightstarDB build will retrieve its dependencies via NuGet using HTTPS connection. 
Unfortunately the certificates used by the NuGet servers aren't recognized by default
in a Linux/Mono environment so we need to first manually import these certificates:

    sudo mozroots --import --machine --sync
    sudo certmgr -ssl -m https://go.microsoft.com
    sudo certmgr -ssl -m https://nugetgallery.blob.core.windows.net
    sudo certmgr -ssl -m https://nuget.org
    
The last three commands will prompt you to confirm import of certificates. You will see
messages about invalid signatures. I believe that this is because the root certificates necessary to validate
the signatures are not installed. In any case you can just go ahead and enter ``y`` to accept
the certificates and have them imported.

## Get BrightstarDB sources

The easiest way to do this is to use git:

    git clone git://github.com/BrightstarDB/BrightstarDB.git
    cd BrightstarDB
    
By default you will have the develop branch checked out. This is *usually* stable, but
if you instead want to build the most recent release then you should checkout the master
branch (though before you do that, please check the notes at the end of this blog post...):

    git checkout master
    
## Build the code!

This is all automated through the build.proj file in the root directory of the BrightstarDB
git sources so all you need to do is invoke XBuild on this project file. By default the 
Debug build is run, you can optionally build the release version though:

    xbuild build.proj
    
OR:

    xbuild build.proj /p:Configuration=Release
    
The build script will create a ``build`` directory under the main BrightstarDB source directory,
and inside that you should find ``sdk`` and ``server`` directories. 
The server is in the ``server`` directory (who'd have thought!):

    cd src/core/BrightstarDB.Server.Runner/bin/Debug
    mono BrighstarService.exe
    
The output should look something like this:

    pi@techquilapi ~/BrightstarDB/src/core/BrightstarDB.Server.Runner/bin/Debug $ mono BrightstarService.exe
    BrightstarDB REST Server 1.11.0.
    Copyright (c) 2015 Khalil Ahmed, Graham Moore, and other contributors
    -------------------------------------------------------------------------------
    
Yay!? Well... no, not quite. If you try to connect to the server now you will find that it cannot list the stores
available. This is because the default configuration file uses a Windows path for its store location and
you Pi2 is unlikely to have a ``c:\brightstar`` directory! So stop the server (just hit Enter) and fire up ``nano``
or your favourite text editor to edit the ``BrightstarService.exe.config`` file that is in the same directory
as the executable. So we finally need to...

## Fix the configuration file

Find the lines that look like:

    <add name="logFile"
         type="System.Diagnostics.TextWriterTraceListener"
         initializeData="c:\brightstar\log.txt" traceOutputOptions="DateTime" >
    </add>
    
and change the path to the log file. Second, find the line that looks like this:

    <brightstarService connectionString="type=embedded;StoresDirectory=c:\brightstar">

and change the path specified for the StoresDirectory parameter. While you are in this file you may want to add some extra configuration
to override the default settings of BrightstarDB. For example, the default cache size is 2GB, which
is more RAM that is available on the Pi2 so to avoid swapping we should specify some more suitable values.
These settings are all specified in the ``appSettings`` part of the config file (which you will find at the
end of the file). This is the configuration I am currently using::

    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <configSections>
        <section name="brightstarService" type="BrightstarDB.Server.Modules.BrightstarServiceConfigurationSectionHandler, BrightstarDB.Server.Modules" />
      </configSections>

      <startup>
          <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.0" />
      </startup>

      <system.diagnostics>
        <trace autoflush="true" indentsize="4"/>
        <sources>
          <source name="BrightstarDB" switchValue="Verbose">
            <listeners>
              <add name="logFile"
                   type="System.Diagnostics.TextWriterTraceListener"
                   initializeData="/home/pi/brightstar/log.txt" traceOutputOptions="DateTime" >
              </add>
            </listeners>
          </source>
        </sources>
      </system.diagnostics>

      <brightstarService connectionString="type=embedded;StoresDirectory=/home/pi/brightstar">
        <storePermissions>
          <fallback authenticated="All" anonymous="All" />
        </storePermissions>
        <systemPermissions>
          <fallback authenticated="All" anonymous="All" />
        </systemPermissions>
      </brightstarService>

      <runtime>
        <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
          <dependentAssembly>
            <assemblyIdentity name="Newtonsoft.Json" publicKeyToken="30ad4fe6b2a6aeed" culture="neutral" />
            <bindingRedirect oldVersion="0.0.0.0-7.0.0.0" newVersion="7.0.0.0" />
          </dependentAssembly>
        </assemblyBinding>
      </runtime>

      <appSettings>
        <add key="BrightstarDB.QueryExecutionTimeout" value="250000"/>
        <add key="BrightstarDB.UpdateExecutionTimeout" value="250000"/>
        <add key="BrightstarDB.TxnFlushTripleCount" value="2000" />
        <add key="BrightstarDB.PageCacheSize" value="512" />
        <add key="BrightstarDB.ResourceCacheLimit" value="100000" />
      </appSettings>
    </configuration>

Now start the BrightstarService again and this time you should be able to connect to the service by pointing
a browser at http://{ip.of.your.pi}:8090/brightstar

## Some Final Notes

* I recently (in the last few days) fixed a Mono-specific issue with command-line option parsing. So right now I recommend that you do build from the develop branch. Once 1.12.0 is out, this should not be a problem any more.
* The service takes a little while to start up (maybe 10 seconds or so) on my Pi2, so once you see the header message, count to 10 (SLOWLY!) before trying to connect from a browser. 
  The code on the develop branch now includes a few additional startup messages so if you do build from develop you can just wait for the message "BrightstarDB Service is running."
