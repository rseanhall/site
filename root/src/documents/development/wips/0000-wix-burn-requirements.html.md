---
wip: ????
type: ????
author: Sean Hall (r.sean.hall at gmail.com)
title: Burn Requirements
draft: true
---

## User stories

* As a Setup developer I can build an installation package such that I can deploy my application along with its prerequisites in a unified user experience.


## Requirements

Nowadays, many applications can't be deployed in a single MSI. For example, some applications have prerequisites like the .NET Framework, and some handle localization by putting each locale's resources into its own MSI. WiX needs to provide a way for the setup developer to declare all of the application's packages and prerequisite packages, and a way to compile it into an executable that installs those packages. This will require four separate components: the compiler, the stub, the engine, and the bootstrapper application.


## Compiler

This new install package will be called a Bundle. Like MSIs, Bundles will be written in .wxs files, compiled with Candle, and linked with Light.

Unlike the [MSI](http://technet.microsoft.com/library/cc978328.aspx) and [MSP](http://msdn.microsoft.com/library/aa370578.aspx) packages, there is no standard way to detect whether [MSU](http://support.microsoft.com/kb/934307) or EXE packages are installed.  The setup developer will need to be able to declare how to detect MSU and EXE packages, and integrate them with the engine's reference counting feature. 

There are some options that will only be available at compile time:

* Whether the packages will be embedded in the bundle, be delivered alongside the bundle, or deployed to a network location so the engine can download it.  This can be set either at the Bundle level or individually for each package.
* Whether to cache the packages in parallel or serially.
* Which packages are included in the Bundle.
* The package installation order.
* For each MSI, whether it shows UI during install.
* For each package, how the engine verifies it.
* For each package, whether it will be cached.
* For each package, whether Burn can uninstall it.


## Stub

The stub will be responsible for starting the engine. The stub should be as small as possible, since a common scenario involves downloading the bundle from the internet.  It also needs to have as few dependencies as possible, because it will be the first thing to run.


## Engine

The engine will be called Burn, it will be able to:

* Coordinate with the bootstrapper application (BA) when planning exactly what needs to be installed/modified/repaired/uninstalled/cached.
* Silently install/modify/repair/uninstall/cache MSI, MSP, MSU, and EXE packages.
* Report progress to the BA, for both the overall operation and the current operation, so that the BA can easily provide a single progress bar (and a second progress bar for the current operation if it wants).
* Report progress for an EXE package if the EXE implements one of the protocols that Burn supports.
* Register and unregister with Add/Remove Programs (ARP).
* Rollback the operation when an error occurs.
* Automatically detect installation state of MSI and MSP packages, as well as the installation state of the features.
* Provide search capabilities so that the setup developer can declare how to detect EXE and MSU packages. 
* Download packages if they weren't provided with the bundle, but only if they are needed.
* Intelligently prompt for elevation.
* Download all packages in preparation for an offline install.
* Reference count packages so that Bundles can share packages without worrying about uninstalling those shared packages while other bundles still need them.
* Automatically determine the minimal number of packages and patches a user has to install to be current (patch slipstreaming).
* Download a newer version of the bundle and then hand over the installation to the newer bundle.
* Support the ISO/IEC 19770-2 standard.
* Expose a COM interface, IBootstrapperEngine, so that the BA can be written in any language that works with COM.


## Bootstrapper Application

The Bootstrapper Application (BA) will implement a COM interface, IBootstrapperApplication. Once the Burn engine initializes, it will startup the BA and wait for commands from the BA through the IBootstrapperEngine interface.  While Burn is processing those commands, it will coordinate with the BA through the callbacks in the IBootstrapperApplication interface.

The BA is responsible for the user interface.

WiX will provide a native code BA, as well as the infrastructure for a managed code BA.


## Burn Security Model

Burn must run elevated when installing per-machine Bundles. Burn is designed to fetch packages from various locations, and then install them. This means that there are lots of opportunities for malicious attackers to get Burn to execute their code. Therefore, Burn will have to be able to verify that the package it's about to run is the exact same package that the setup developer referenced when the Bundle was compiled.  The setup developer will be able to specify at compile time whether to verify by digital signature or hash. Because Burn needs access to the package's hash at compile time, it will not be possible to add or modify packages in a Bundle's chain after it is compiled.

Since the engine will be embedded in the Bundle, WiX needs to provide a way to sign the engine.

TODO: Insert WiX's rationale for not supporting the BA running elevated


## Bundle Identity

Each Bundle has its own identity. Burn uses this identity:

* When registering with ARP, which is necessary in order to be able to clean the Bundle's cached packages on uninstall.
* To support update and patch scenarios.
* To support reference counting packages.


## Add-on Bundles

TODO


## Sticky Patching

TODO


## Considerations

This document was written well after Burn was implemented, and by someone who was not in the loop while it was being implemented.


## See Also

These are all of the blog posts I could find about the design decisions of Burn.

1. http://www.joyofsetup.com/2008/06/21/wix-burn-volume-1-layers/
1. http://www.joyofsetup.com/2008/08/25/everybody-could-use-a-good-burn/
1. http://robmensching.com/blog/posts/2009/7/14/lets-talk-about-burn
1. http://robmensching.com/blog/posts/2009/7/22/burn-packages-payloads-and-containers
1. http://robmensching.com/blog/posts/2010/1/15/burn-moves-to-a-new-foundation
1. http://robmensching.com/blog/posts/2010/9/6/burn-baby.-burn
1. http://robmensching.com/blog/posts/2010/9/7/burn-in-the-corporate-environment
1. http://robmensching.com/blog/posts/2011/10/24/wix-v3.6-beta-released
1. http://blogs.msdn.com/b/heaths/archive/2012/02/21/introducing-package-reference-counting.aspx
1. http://blogs.msdn.com/b/heaths/archive/2012/02/21/introducing-sticky-patching-and-add-ons.aspx
1. http://blogs.msdn.com/b/heaths/archive/2012/03/07/packaging-caching-in-burn.aspx
1. http://robmensching.com/blog/posts/2012/4/16/wix-toolset-support-for-software-id-tagging---isoiec-19770-2
1. http://robmensching.com/blog/posts/2012/6/25/b-is-for-bundle-and-thats-good-enough-for-me
1. http://blogs.msdn.com/b/heaths/archive/2012/07/26/easier-authoring-for-slipstreaming-patches.aspx
1. http://robmensching.com/blog/posts/2012/8/21/enter-wix-v3.7-toolset
1. http://www.joyofsetup.com/2013/07/05/burn-zero-one-or-n/
