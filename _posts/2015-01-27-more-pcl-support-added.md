---
layout: post
title: More PCL Support Added
tags: update
---

I've just committed a change to the develop branch of BrightstarDB that adds support
for the Xamarin.iOS PCL framework. This is an important step forward for our cross-platform
mobile support as it enables you to use the latest incarnation of Xamarin.Forms to build
Android, Windows Phone and iOS applications from a common code-base with BrightstarDB.

I hope to put together an introductory tutorial on this and add a sample to the next
BrightstarDB release. In the meantime, if you are eager to try it out for yourself, then you can compile
everything from the source in GitHub. This is best achieved on a Windows machine running
Visual Studio (I recommend VS2013 with latest updates) and the Xamarin VS plugin. You will
need to build the android.sln, ios.sln and portable.sln solutions all of which can be found
in the src/portable directory. Alternatively you can build the NuGet packages with the following
commands:

    cd installer
    msbuild installers.proj /t:NuGet
    
Note that this alternative build process will also build documentation and will therefore require
that you have Sandcastle Help File Builder installed.
    
If you have questions please feel free to ask them on our [Codeplex forum](http://brightstardb.codeplex.com/discussions)