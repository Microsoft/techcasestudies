---
layout: post
title:  "DevOps practices help startup Alterbooth focus on the essentials for an upcoming launch"
author: "Junichi Anno"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/junichia.png"
date:   2017-02-13
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/alterbooth/logo.png" #should be ~350px tall
excerpt: Microsoft and Alterbooth conducted a joint hackfest to introduce DevOps practices into Alterbooth's new sauce-ordering and customizing service.
language: [English]
verticals: [Technology, "Retail, Consumer Products & Services"]
---

Microsoft and Alterbooth conducted a joint hackfest to introduce DevOps practices into Alterbooth's new service, "MySauce Factory." The DevOps practices we implemented are:

-	Continuous integration/continuous delivery
-	Automated testing

**Core team:**

- Kunihiro Morita – Alterbooth
- Yuta Matsumura – Alterbooth
- Daisuke Kozuka – Technical Evangelist, Microsoft
 
![Alterbooth]({{ site.baseurl }}/images/alterbooth/alterbooth01.png)


## Customer profile ##

[Alterbooth, Inc.](https://www.alterbooth.com/), is a small, Japan-based system integrator (SI) founded in 2015. It has 10 employees at this time and has been awarded silver competency as a Microsoft partner. Its business is mainly IT consulting and customized development with UI design. It describes its business as:

1. System planning, development, and operation using cloud computing.
2. Internet-related system planning, development, and operation.
3. SaaS, ASP service, and IT product planning, development, and operation.
4. Any business related to the above areas.

Alterbooth has just started work on a new solution, *[MySauce Factory](https://mysaucefactory.com/)*, currently in beta. It allows customers to order original and customized sauces from the MySauce Factory website. Consumers select a base sauce and customize it with increasing/decreasing flavors—for example, they can adjust the level of sweet and sour, oil, vinegar, or onion.

Although still in beta operation now, customers can buy the original sauce. Alterbooth plans to release the solution in March 2017.

This service is Alterbooth's own—not ordered from another company. Kojima-san, Alterbooth's CEO, said, "We are focusing on designing business logic," not only IT design. 

Although this is a small, new company, every employee has a high skill set. This is recognized within communities such as JAWS (Japan AWS users group) and JAZUG (Japan Azure User Group). In Japan, 75 percent of engineers work for SI companies, and 25 percent work for end users. 

To expand the DevOps market in Japan, cooperation and change for SI companies are necessary. To motivate them to implement and use DevOps practices, we needed an actual DevOps case that is both advanced and influential. Alterbooth is a good company for that, especially once its new service is released in March. 

## Architecture ##

![whole of architecture]({{ site.baseurl }}/images/alterbooth/alterbooth02.png)


MySauce Factory's architecture is simple—a front webpage and one API. The front webpage is built with HTML/PHP. The API is built on .NET Core using Visual Studio for coding. Once they commit the API's source code and sync to GitHub, Jenkins issues a get command every 15 minutes, compiles a build, tests it automatically, and deploys it into a Docker container using Ansible. They already have successfully automated the release of the API.

![code of build pipeline on Jenkins]({{ site.baseurl }}/images/alterbooth/Alterbooth04.png)


<img src="{{ site.baseurl }}/images/alterbooth/alterbooth05.png" width="600">


The front webpage is simple. HTML/PHP files are deployed to an Azure web app directory using a git command. See [Local Git Deployment to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-deploy-local-git).

They use Azure Container Service (Swarm) for a Docker orchestrator. The reason for this selection is because:

- Azure Container Service can use Docker Swarm and work together with Ansible.
- Azure Container Service is easy to learn—they can prepare a Container Service environment from Visual Studio. (The lower cost of learning is important to them.)
- They once compared it with AWS ECS and felt they were similar, so they chose the one that was easier to use.

To learn basic procedures for deploying a Docker image onto Container Service, see the following resources:

- [Windows Containers on Windows 10](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)
- [Visual Studio Tools for Docker](https://docs.microsoft.com/en-us/dotnet/articles/core/docker/visual-studio-tools-for-docker)
- [Deploy an ASP.NET container to a remote Docker host](https://docs.microsoft.com/en-us/azure/vs-azure-tools-docker-hosting-web-apps-in-docker)
- [Deploy an Azure Container Service cluster](https://docs.microsoft.com/en-us/azure/container-service/container-service-deployment)

## Problem statement ##

We first conducted a value stream mapping (VSM) exercise to clarify Alterbooth's issues.

![Value Stream Mapping]({{ site.baseurl }}/images/alterbooth/alterbooth03.png)


Alterbooth already uses Agile development and succeeded in shortening the release cycle to one week. The full lead time is 4 days and 3.5 hours. The process time is 4 days and 3.1 hours, a small difference. But the "web load testing" requires 3 days and two people. This occupied a large part of the process time. 

Although they once tried to use Selenium for load testing, they couldn't make it work well. Because time for the beta release was approaching, they decided to do load testing manually after all. 

The "MySauce Factory" will be available in March 2017, so Alterbooth wanted to focus on coding and accelerate development of new features. It is a small company and cannot increase the members of this project. So, the members were under pressure to improve their web load testing with automation.

They didn't use Visual Studio Team Services (VSTS) for release management because VSTS didn't support .NET Core and Docker when they were designing the architecture. But by the time of the VSM, VSTS supported both. So we proposed the following:

- Migrate release management to VSTS.
- Don't discard Jenkins and Ansible, but test without them for now.
- Automate web load testing with VSTS.

They decided to use Container Service (DC/OS) for the orchestrator. They wanted to use DC/OS after seeing it at our tier 1 event, "Tech Summit 2016," in Tokyo. The reason is simple. DC/OS has a Marathon web UI for managing and monitoring. Using a Marathon web UI, we can monitor usage of CPU, memory, and storage at a glance. 

[Manage an Azure Container Service DC/OS cluster through the Marathon web UI](https://docs.microsoft.com/en-us/azure/container-service/container-service-mesos-marathon-ui)

## Getting started with VSTS ##

First, I coached the project team on how to get started with VSTS by opening a VSTS account. To open a VSTS account for free, go to [https://www.visualstudio.com/](https://www.visualstudio.com/) and click **Get started for free** under Visual Studio Team Services. If you already have an Azure Active Directory tenant, you can connect your VSTS to Azure AD when opening the VSTS account. (Alterbooth had connected to Azure AD.) If you don't have an Azure AD tenant, you can manage users in your VSTS locally.

After opening a VSTS account, you can create a new project and then assign users to it.

Be aware of restrictions if using the free version of VSTS. A project can have up to 5 users who do not have a Visual Studio subscription. Check the following for reference: [Visual Studio Team Services Pricing](https://www.visualstudio.com/team-services/pricing/).

In my experience, most participants get confused when they first try to add a project member. To help, I explain about the hierarchy of VSTS using an example like the following:

*Assigning users to a VSTS project*

![Assigning users to VSTS project]({{ site.baseurl }}/images/alterbooth/Alterbooth14.png)


Also, the following is useful: [Visual Studio Team Services and Personal Microsoft Accounts](https://blogs.msdn.microsoft.com/winsdk/2016/09/29/visual-studio-team-services-and-microsoft-accounts-vsts-and-msas/)

## Migrating release management to VSTS ##

As described previously, Alterbooth uses Jenkins and Ansible. Its Jenkins pipeline is simple, as shown below, so team members thought the transition would be easy. 

[Sample code of build pipeline on Jenkins](https://gist.github.com/junichia/72658d7abea82c406188acb861f83e9b).

They knew how to build it from an MSDN site. They succeeded in building a pipeline, shown below, within 1 day. Because they manage their source code using Git and GitHub, they could easily migrate to VSTS. The release pipeline starts when they sync their committed code from Visual Studio 2015.

*The pipeline after the migration* 

![build pipeline on VSTS]({{ site.baseurl }}/images/alterbooth/Alterbooth07.png)


- Step 1. Restore .NET Core.

  Restore the NuGet package references in the project.
  
  ![Step1:Restore .Net core]({{ site.baseurl }}/images/alterbooth/Alterbooth_1_core_restore.png)
  
  
- Step 2. Build .NET Core. 

  ![Step2:Build .Net core]({{ site.baseurl }}/images/alterbooth/Alterbooth_2_core_build.png)
  
  
- Step 3. Publish .NET Core.

  All resources are collected in one folder.
  
  ![Step3:Publish .Net core]({{ site.baseurl }}/images/alterbooth/Alterbooth_3_core_publish.png)
  
  
- Step 4. Copy the files. 

  Copy the Docker files to the location in step 3.
  
  ![Step4:Copy files]({{ site.baseurl }}/images/alterbooth/Alterbooth_4_copy.png)
  
  
- Step 5. Build a Docker image. 

  After collecting the required files in step 4, this step creates a Docker image. The created image is stored in the location written in the "Docker Files" field.
  
  ![Step5:Build a Docker Image]({{ site.baseurl }}/images/alterbooth/Alterbooth_5_docker_build.png)
  
  
- Step 6. Publish a Docker image.

  The Docker image created in the previous step is registered at Azure.
  
  ![Step6:Publish a Docker Image]({{ site.baseurl }}/images/alterbooth/Alterbooth_6_docker_push.png)
  

After the build pipeline, the release pipeline starts. In this pipeline, a Docker image is deployed to Azure Container Service.
Although there are no test steps in the following image, you can add a test step by clicking the **+ add task** button. If you want to delay deployment until receiving an individual's approval, you can add that approver in the Environment configuration.

<img src="{{ site.baseurl }}/images/alterbooth/Alterbooth10.png" width="700">


<img src="{{ site.baseurl }}/images/alterbooth/Alterbooth06.png" width="700">


<img src="{{ site.baseurl }}/images/alterbooth/Alterbooth08.png" width="700">


Here is the docker-compose.yml file: [https://gist.github.com/junichia/795da5a1e9a6edc1a2c5e8314b4c3938](https://gist.github.com/junichia/795da5a1e9a6edc1a2c5e8314b4c3938). As you can see on the "image" tag, 260 or 261 corresponds to the Docker build number. The Docker image is registered at Azure using this number. In the "environment" are important variables "VIRTUAL_HOST" and "VIRTUAL_PORT".

In the following image, you can see that VIRTUAL_HOST and VIRTUAL_PORT are used for accessing from the front webapp.

![accessing api]({{ site.baseurl }}/images/alterbooth/access_image.png)


"Networks" in each build (blue or green) refer to which networks the Docker image uses. Network is defined at the "Networks" tag at the bottom of the file. This Docker image uses Network_front for receiving requests from the front webpage, and Network_back for accessing the SQL database.

Here are some of the Alterbooth team's impressions from this hackfest:

- *We can create a pipeline just by adding steps from the selector or the catalog. We can cut scripting and debugging costs with it.* 

  <img src="{{ site.baseurl }}/images/alterbooth/Alterbooth_7_core_command.png" width="500">
  
- *VSTS has a high affinity for Azure. All we need to do is choose steps associated with Azure from the task catalog.*

  <img src="{{ site.baseurl }}/images/alterbooth/taskcatalog.PNG" width="500">
  
- *Needless to say, VSTS also has a high affinity for Visual Studio! We can connect to VSTS from VS directly, easily commit. and sync. We don't have to open another window when developing. This means the "environment for development becomes so simple!"*
- *The affinity between tools gives us speed! Speed is most important for us.*

We also got some requests from the Alterbooth team:

- *We sometimes had difficulties clarifying what we should put as a parameter.*

  *We could find articles we wanted with the evangelist’s help.*
  
- *We want a function for versioning a pipeline. The Jenkins pipeline is a code. So it is easy to manage their versioning using Git.*

  *VSTS can "clone" definitions or save as a "custom template." Custom templates can be reused when creating a new pipeline. So we can use it for simple versioning.*
  
  <img src="{{ site.baseurl }}/images/alterbooth/contextmenu.png" width="300">

## Automated testing ##

Next, we tried the automated testing. We created a "Web performance and load test project" using Visual Studio 2015 Enterprise. They had learned the testing steps on an MSDN blog by Charles Sterling. The series describes steps ranging from "creating a test project" to "doing load test on the VSTS."

[Load Test Series Part I: Creating Web Performance Tests for a Load Test](https://blogs.msdn.microsoft.com/charles_sterling/2015/06/01/load-test-series-part-i-creating-web-performance-tests-for-a-load-test/)

The following is the test code created after recording.

![load test project]({{ site.baseurl }}/images/alterbooth/Alterbooth11.jpg)


Test result on Visual Studio 2015. In this hackfest, they didn't upload this test project.

![load test result on VS2015]({{ site.baseurl }}/images/alterbooth/Alterbooth12.jpg)


Alternatively, they executed load testing on Application Insight. To use Application Insight, you must create an Application Insight workspace on the Azure portal. You can upload ".webtest" file onto Application Insight and execute it. The following article describes this step by step: [Monitor availability and responsiveness of any web site](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-monitor-web-app-availability).

Viewing the test result on Application Insight. 

![load test on Application Insight]({{ site.baseurl }}/images/alterbooth/application_insight.jpg)


This is a sample web load test project they created: [https://github.com/junichia/samples/tree/master/WebAndLoadTestProject1](https://github.com/junichia/samples/tree/master/WebAndLoadTestProject1)

Project members were surprised at how easy it was to do load testing with the web recording function. Although some adjustments such as Request Header were necessary, they loved this function and understood how to use it. 

At the end, they could see the prospect of shortening their load test time from 3 days to just 1 hour! 

## References and documents ##

- [Visual Studio Team Services and Personal Microsoft Accounts](https://blogs.msdn.microsoft.com/winsdk/2016/09/29/visual-studio-team-services-and-microsoft-accounts-vsts-and-msas/)
- [Deploy an agent on Linux](https://www.visualstudio.com/en-us/docs/build/admin/agents/v2-linux)　
- [Building .NET Core Linux Docker Images with Visual Studio Team Services](https://blogs.msdn.microsoft.com/stevelasker/2016/06/13/building-net-core-linux-docker-images-with-visual-studio-team-services/) 
- [Performance test your app before release](https://www.visualstudio.com/en-us/docs/test/performance-testing/run-performance-tests-app-before-release) 
- [Load Test Series Part I: Creating Web Performance Tests for a Load Test](https://blogs.msdn.microsoft.com/charles_sterling/2015/06/01/load-test-series-part-i-creating-web-performance-tests-for-a-load-test/)
- [Monitor availability and responsiveness of any web site](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-monitor-web-app-availability)
- [Local Git Deployment to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service-web/app-service-deploy-local-git)

## Conclusion ##

Before the hackfest, Alterbooth team members thought that introducing automated load testing would be the next step after GA in March 2017. But through VSM and the hackfest, they got new skillsets to do the load test project during coding in Visual Studio 2015. Although they use some OSS components, VSTS supports integrating these tools and managing the release process. Our hackfest helped accelerate their processes, which was important because they must launch MySauce Factory on schedule in March 2017.

They have a plan to create native client apps using Xamarin in the future. And they are going to adopt microservices. Since the current API includes all functions, they want to divide it into multiple APIs and make them easier to release.

The Alterbooth team's thoughts following the hackfest:

> "We learned that tools from Microsoft work together well even in the cloud. Why does Microsoft recommend OSS on Azure these days? You should appeal your own tools more."
