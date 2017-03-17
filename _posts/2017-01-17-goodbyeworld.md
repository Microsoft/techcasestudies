---
layout: post
title:  "Creating an automated build pipeline with Azure for GoodbyeWorld Games"
author: "Tobiah Zarlez"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-14 
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: With help from Microsoft, GoodbyeWorld Games implemented a DevOps solution that automated its build process, speeding up and improving its development pipeline. 
verticals: ["Retail, Consumer Products & Services"]
language: English
---

Microsoft helped GoodbyeWorld Games create a new, automated build and release pipeline. What used to take five hours of work with a seven-day lead time now happens automatically in minutes through DevOps practices.

After performing a value stream mapping (VSM) session, the project team identified key areas of focus and Microsoft helped implement a solution using the following DevOps practices:

- Infrastructure as code (IaC)
- Continuous integration (CI)
- Continuous deployment/build management 
- Automated testing

**The project team:** 

- Will Hellwrath ([@GoodbyeWorldWIL](https://twitter.com/GoodbyeWorldWIL)) – Captain of GoodbyeWorld Games
- Tobiah Zarlez ([@TobiahZarlez](https://twitter.com/TobiahZarlez)) – Owner/Technical Evangelist, Microsoft
- Jennelle Crothers ([@jkc137](https://twitter.com/jkc137)) – Technical Evangelist, Microsoft

## Customer profile ##

GoodbyeWorld Games is the Los Angeles-based studio behind "[Close Your](http://www.closeyour.com/)," a first-person interactive short story. In the game, you live an entire life from birth to death and make decisions big and small that shape the outcome of that life. The catch is that each scene lasts only as long as you can keep your eyes open. The game uses your webcam to track your eyes in real life, so every time you blink, you skip forward: could be five seconds or five years. Life literally flashes before your eyes as you try desperately to hold onto each fleeting moment.

A Kickstarter campaign to fund the game attracted 1,515 backers and garnered $35,992 in pledges. Expected launch date is in Q1 2017.

The game is developed with Unity and is currently targeting the Windows and Mac desktops for its initial release, but they plan to expand to other platforms after that.

## Problem statement ##

GoodbyeWorld has three engineers. When we met with them, they had no official build process. Builds were created manually, usually by one engineer (Will). Builds were then manually uploaded and shared via Dropbox.

Creating new builds took time, and the non-standardized process led to confusion from the rest of the team when determining which build was the latest. This manual process made it almost impossible for the team to load previous known good builds.

![VSM Part 1]({{ site.baseurl }}/images/goodbyeworldgames/old.png)


"We were doing it the dumb way," Will said during our VSM session.

A new build was a five-hour process, with a seven-day lead time.
 
## Solution, steps, and delivery ##

Our goals were as follows:

- Create a value stream map of the current GoodbyeWorld Games process.
- Identify problem areas that can be automated/streamlined.
- Set up a technical solution to solve those problems.
	
### Value stream map ###

![VSM Part 1]({{ site.baseurl }}/images/goodbyeworldgames/vsm1.png)


![VSM Part 2]({{ site.baseurl }}/images/goodbyeworldgames/vsm2.png)


The value stream mapping was one of the most valuable parts of the project for GoodbyeWorld Games, as they had never put effort into formalizing their build process before.

The first draft of the GoodbyeWorld Games' VSM had only three nodes. Then after an hour of Tobiah and Jennelle clarifying with them further, the map was expanded to reveal the full extent of their existing build process: 10 nodes. They were then able to quantify how much work and lead time went into their process every single week. Will was identified as major hero. He was spending multiple hours every week just on builds.  

With the process quantified, it was easy to come up with solutions on where automation made sense. Tobiah and Jennelle explained how setting up a Jenkins instance would streamline the majority of their process. We estimated with our solutions we could eliminate the five-hour build process and seven-day lead time.

### Project scope

We agreed on the following:

* Automating builds 
* Archiving of current and previous builds
* Acid test of "Did the build work?"
* Email and notifications of new build links

Possible expansions for future projects:

* Unit testing
* Enhanced deployment processes 

### Implementation and delivery ###

![VSM Part 1]({{ site.baseurl }}/images/goodbyeworldgames/azure.png)


The goal of this project was to provide an easy to deploy, easy to maintain "everything in one box" server, which reduced Azure costs and kept infrastructure management to a minimum. To meet this need, we used the base Azure virtual machine image of Windows Server 2012 R2 with Visual Studio Community already installed. A [JSON template](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) was used to deploy the required virtual network and ports supporting Jenkins, the required storage accounts, and virtual machine. The template included a DSC extension to configure some additional operating system features and to install [Chocolatey](https://chocolatey.org/), which was then used to install community-maintained packages for Jenkins, Unity, and Git.

We walked Will through setting up an Azure account and connecting via PowerShell to deploy the template. This process was also documented in a "[getting started](https://github.com/TobiahZ/JenkinsUnityBuildPipeline/blob/master/StartHere.ps1)" PowerShell file. Once the deployment was completed, we assisted in the direct configuration of the Jenkins server for connectivity to Git and to manage the Unity deployment. This included making sure Unity had the correct license information to be able to create builds via Azure CLI. We found that manually connecting via VPN into the virtual machine to configure Unity made more sense for GoodbyeWorld Games than creating an automated script. A script would have to be uniquely created per user, and the configuration only needed to be done once.

[JSON template](https://github.com/TobiahZ/JenkinsUnityBuildPipeline/blob/master/buildserverdeploy.json)

[DSC configuration file](https://github.com/TobiahZ/JenkinsUnityBuildPipeline/blob/master/BuildServerDSCconfig.ps1)

### Jenkins configuration details

After the server was ready, we then worked with Will from GoodbyeWorld Games to configure Jenkins to do the following:

1. When a new commit is created on the master branch of their private GitHub repo, trigger a new build.
2. Jenkins then downloads the changes via Git and then passes parameters to Unity to do the build and unit testing.
3. If successful, it will archive the build artifacts and upload them to Azure cool blob storage.
4. Jenkins will then notify the team via email and Slack of the new build, with a direct link to download.

### Technical issues

**Jenkins with private GitHub repos**

The documentation for working with private GitHub repos with Jenkins was not accurate. We learned that the best way was to let Jenkins handle it all automatically. When we tried to configure the setting manually, despite following the documentation, Jenkins would not authenticate with GitHub.

The automatic solution is easy to follow once we knew to make sure we do not manually interfere with it.

[Jenkins GitHub documentation (must follow Automatic instructions).](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+Plugin)

**Unknown dependencies**

GoodbyeWorld Games had been building their projects on one developer's machine, and were not aware of all the dependencies they needed to compile builds. 

We identified file naming and file format issues, which will cause problems as development on the project continues. Unfortunately, finding and fixing these minor issues took up the majority of time on the project and are still not fully resolved. In order for the builds to be consistently successful on Jenkins, the developers will have to devote more time to resolving the file issues.

These issues likely would have been discovered at some point later in the project's development. It's good to address the issues as early as possible because it will save time down the road.

## Conclusion and impact of DevOps process

As predicted in the VSM, the process that used to take five hours with a seven-day lead time now happens automatically every time an engineer commits to the master branch on their GitHub repo, completing within 10 minutes. 

New builds are created automatically without taking up any developer time, and links are sent out via email and Slack. Every build is then saved to Azure cool blob storage, in case they ever want to revisit an older version.

**Results**

Builds are now produced more frequently and with more clarity as to which is the latest. The team also now has easy access to known good build history, without requiring "heroic" acts each week for build processing and other related administrative tasks, including uploading files and sending manual email notifications. Because GoodbyeWorld Games lacks a formal Ops team to manage infrastructure at this time, providing a templated configuration for easy deployment to Azure allows them to control their ops costs and confidently deploy additional virtual machines if they need to expand on the existing setup with additional servers in the future.

Convincing any team to take time away from the development of their project is tough, but this GoodbyeWorld Games example shows how much benefit even a small team can get. Time invested in implementing DevOps practices is time saved.

**Benefits to others**

We think the process that we helped GoodbyeWorld create can be adopted by other developers who want to have their own Unity cloud build system. We hope that these documented learning and deployment scripts are useful to other companies that want to implement a similar system.

**Next steps for GoodbyeWorld Games:**

- Solve any remaining file name and format issues on the primary code base of Close Your.
- Expand automatic unit testing with more comprehensive tests.
- Create build triggers for more platforms.

## Source code

- [Public GitHub repo with infrastructure templates](https://github.com/TobiahZ/JenkinsUnityBuildPipeline)
- [Public GitHub repo with sample Unity project for testing](https://github.com/TobiahZ/DumbGame)
