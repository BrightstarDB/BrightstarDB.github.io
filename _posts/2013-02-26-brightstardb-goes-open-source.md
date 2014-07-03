---
layout: post
title: BrightstarDB Goes Open Source
tags: opensource licensing
---
In the last year or so of developing BrightstarDB, one thing we have always been aware of is how much of our competition is open source. We initially chose not to do that.  Having a closed source approach has enabled us to move quite quickly towards having a stable first release with the core features that we felt were important – it gave us total control over what we developed and how we did it. We briefly looked at the possibility of making a viable closed-source commercial product, but it was not to be. So now the time has come to relinquish some of that control and so we are today taking the step of announcing that BrightstarDB is to be released under an Open Source license.

The license we have chosen is the permissive MIT license – we hope that by choosing this license we can encourage developers to use BrightstarDB in their projects regardless of whether they are developing open or closed-source software.

So what’s next ?

Over the next few days there will be some tweaks and changes to the site and some tidying up of the code to enable everything to be built with a basic tool-kit of Visual Studio 2012 and some free / open-source tools. The main task facing us is the migration of documentation from the commercial Help and Manual product to some other form (right now we are thinking probably DocBook XML as HTML output for the site will be our primary concern); and some updates to the installer build to remove dependencies on the commercial obfuscator/IL merge tool we were using.

We are hosting the source on [GitHub](https://github.com/BrightstarDB/BrightstarDB) and we will use [CodePlex](http://brightstardb.codeplex.com/) to host the “official” project binaries as and when they become available as well as using NuGet to distribute the core libraries as we have been doing with the closed-source offering up to now.

Want to get involved ?

Great! To be honest right now we are focussing just on getting the distribution updated and a new release out under the MIT license which can be built without using any of the commercial tools we have used in the past (with the exception of VS2012 for now). However, we would welcome potential contributors to come forwards, and of course you can always fork us any time you like on GitHub!

Onwards and upwards!