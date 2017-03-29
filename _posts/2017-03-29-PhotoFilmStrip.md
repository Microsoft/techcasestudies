---
layout: post
title: "Bringing PhotoFilmStrip to the Windows Store using Desktop Bridge and Desktop App Converter"
author: "Mike Taulty"
author-link: "https://mtaulty.com"
#author-image: "{{ site.baseurl }}/images/authors/MikeTaulty.png"
date: 2017-03-29
categories: [Desktop Bridge]
color: "blue"
excerpt: PhotoFilmStrip, a highly popular free tool with many downloads from Source Forge made use of the Desktop Bridge to bring their existing Win32 application to the Windows Store. The app conversion was largely handled by the Desktop App Converter, read on to learn more about their story.
language: English
verticals: [Entertainment]
---

At the time of writing, Windows 10 runs on more than 400 million PCs providing a huge base of users for application developers to target with the Windows Store being increasingly central to those users' experience of the operating system.

Since the release of the Windows 10 Anniversary Update in 2016, the Windows Store has had a new capability of being able to deliver its many strands of functionality to Windows Desktop applications ('Win32' apps) targeting PC devices.

For the desktop application user, the Windows Store makes it easier to find, install, update, rate, review, buy and safely uninstall those applications in a trusted manner.

For the developer, the Store adds many features including advanced analytics, commerce options, flighting of previews and trials, A/B testing, crash statistics and much more.

A growing number of desktop applications are taking advantage of the Windows Store and in this case study, we relate how Microsoft worked with Jens Göpfert, the developer of the popular [PhotoFilmStrip](http://www.photofilmstrip.org/) desktop application to bring his work to the Windows Store through the power of the [Desktop App Converter](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-run-desktop-app-converter) tool in a simple, repeatable manner which did not involve him making a single code change to his existing binaries.
 
Core team:

- [Jens Göpfert](https://sourceforge.net/u/hammerwerfer/profile/) – Developer, PhotoFilmStrip
- Mike Taulty ([@mtaulty](https://twitter.com/mtaulty)) - Technical Evangelist, Microsoft UK
- Matteo Pagani ([@qmatteoq](https://twitter.com/qmatteoq)) – Windows AppConsult Engineer, Microsoft Italy

## Customer profile ##

[PhotoFilmStrip](http://www.photofilmstrip.org/) creates video clips from images in a simple way by asking the user to work through the following three steps;

* Select the images
* Determine the motion path
* Create a video

The slideshow effect produced is often referred to as the "Ken Burns" effect and can include image titles (added to a subtitle file) and background music.

The application outputs to VCD, SVCD, DVD and one of the special features is that a slideshow in FULL-HD (1920x1080) is available.

PhotoFilmStrip is available on Windows, MacOS and Linux and is catalogued on Source Forge.

The app has received over 130,000 downloads in the past calendar year with over 65% coming from Windows and over 15% being from Italy (source: Source Forge statistics).

## Problem statement ##

On Windows, PhotoFilmStrip has a Windows Desktop ('Win32') application which is installed via a standard 'setup.exe' available via Source Forge and via the developer's website. 

This traditional installation process requires the user to have administrative rights and, as such, might not the best experience for a Windows 10 user.

In addition, there are many capabilities that a modern app store provides which aren't available to PhotoFilmStrip when delivered this way - e.g.commercialisation opportunities, analytics, trials and many more.

As the developer of PhotoFilmStrip, Jens was aware of the Windows Store but thought that taking his app to the Store might come with challenges;

	"the Store was on my todo list but I was somehow afraid to rewrite some code"

However, with Windows 10 Anniversary Edition and later there are some desktop applications where the existing binaries can be re-packaged and published to the Windows Store without necessarily having to make **any** code modifications.

With that said, where a developer can benefit from some of the Universal Windows Platform features (e.g. A/B testing, Toast notifications) then it is possible to add calls to the [supported UWP APIs](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-supported-api) into the part of their code base which targets Windows 10 and gain those advantages in a realistic, incremental manner. There is no need for the 'rewrite'.

In the case of PhotoFilmStrip, Jens wanted to minimise code changes and so the Desktop App Converter was the best choice in terms of producing a quick conversion from an existing setup process.

Jens worked with both Mike Taulty from the Microsoft UK DX team and Matteo Pagani from the Microsoft Italy AppConsult team to accelerate his route to the Windows Store.

## Solution, steps, and delivery ##

Initially, Mike showed Jens the power of the Desktop App Converter by making a [screen capture video](https://vimeo.com/206256651/1e4432d619) of the conversion process involved in taking;

* Jen's existing PhotoFilmStrip setup
* The [Desktop App Converter]([https://developer.microsoft.com/en-us/windows/bridges/desktop])

and running the setup through the converter in "quiet" mode so as to perform the conversion and produce a .APPX package via which the application can then be installed with all the [additional functionality around registry and file system virtualisation](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-behind-the-scenes) that the Desktop Application Bridge brings to these applications on Windows 10.

The specific command line used on the converter was as below;

	DesktopAppConverter.exe -Installer .\PhotoFilmStrip\setup_photofilmstrip-3.0.0.exe -InstallerArguments "/VERYSILENT" -Publisher "CN=PhotoFilmStrip" -Version 1.0.0.0 -Sign -MakeAppx -PackageName PhotoFilmStrip -Destination .\PhotoFilmStripOut\

The silent mode is crucial here whereas the rest of the parameters are fairly generic.

Jens did not initially have a [suitable development machine](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-run-desktop-app-converter#system-requirements) on which to run the Desktop App Converter but the demonstration of the process motivated him to go and build such a machine in order to experiment with this re-packaging of PhotoFilmStrip;

	"This video motivates me to do it right away and how to start."

Naturally, Jens wanted to tailor the process and so worked remotely with Mike and Matteo to improve the experience provided by the .APPX package beginning with the set of files produced by the converter.

Jens specifically altered the **appmanifest.xml** file in order to change the names, descriptions and icons used by the application and also to add a suitable splash screen.

Some of the areas that Jens and Mike covered along the way from desktop to Store included;

* The different options for producing .APPX files both [manually](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-manual-conversion) and automatically
* How .APPX packaging and the Windows Store handle processor architectures
* Making and installing development certificates and the process for manually building .APPX packages with the **makeappx** and [**signtool**](https://docs.microsoft.com/en-gb/windows/uwp/porting/desktop-to-uwp-signing) tools
* Creating a Windows Store [developer account](https://developer.microsoft.com/en-us/store/register) and submitting packages
* Investigating how the capabilities of the Windows Store [analytics](https://docs.microsoft.com/en-us/windows/uwp/publish/analytics) engine can improve Jens' information about the users of his app

<img alt="PhotoFilmStrip Desktop App in the Windows Store" src="{{ site.baseurl }}/images/PhotoFilmStrip/AppInStore.png" width="600">

Thanks to the Desktop App Converter, making a .APPX package that can be submitted to the Windows Store has been a straightforward operation. From a technical standpoint, the app did not raise any areas the the Desktop App Converter could not handle and did not require using some of the more sophisticated switches to accommodate installers that (e.g.) produce a range of exit codes or need to have the contents of their virtualised registry hives edited post-conversion.

## Conclusion ##

Thanks to Desktop App Converter, PhotoFilmStrip achieved several goals:

1. Provide a new, discoverable way for Windows 10 users to find, install, rate, review, uninstall the app in the natural way for the modern operating system
2. Learn more about the users of the application via analytics
3. Provide an additional way for PhotoFilmStrip to monetize the application
4. Open the way to add new features and provide an enhanced experience on Windows 10

On (2), Jens noted;

	"I am excited about the installation statistics that I can see in my dashboard. Until now, I only used the download counts which are not very reliable"
		
and around (4) he added;

	"For my next app update I plan to use more of the appx features like file type association"

and on the process of bringing his application to Store with help from Microsoft, Jens has been pleased to work with the Microsoft team;

	"Support by you and your colleagues was awesome. Very quick responses and cooperative - Thanks again ;-)"

## Additional resources ##

- Desktop Bridge: [https://developer.microsoft.com/en-us/windows/bridges/desktop](https://developer.microsoft.com/en-us/windows/bridges/desktop)
- Matteo Pagani's App Consult Desktop Bridge blog posts: [https://blogs.msdn.microsoft.com/appconsult/tag/desktop-bridge/](https://blogs.msdn.microsoft.com/appconsult/tag/desktop-bridge/)
- Mike Taulty's Desktop Bridge blog posts: [https://mtaulty.com/category/desktopappconverter/](https://mtaulty.com/category/desktopappconverter/)


