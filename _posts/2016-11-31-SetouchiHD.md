---
layout: post
title: "Learning from a microservices/mobile hackfest with Setouchi Holdings"
author: "Tsuyoshi Ushio"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:  2017-02-28
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Setouchi Holdings needed to improve the scalability of their pipeline. CI&T, Creationline, and Microsoft used the Web Apps and API Apps features of Azure and Visual Studio Team Services to quickly solve pain points.
language: English
verticals: [Manufacturing, Transportation & Logistics]
---

Setouchi Holdings provides several services related to small-airplane manufacturing. They are developing a new reservation system for aerial sightseeing in the Setouchi area. They work with [CI&T](http://www.ciandt.com/home) and [Creationline](http://www.creationline.com/en/): CI&T is a software-development company, skilled in Agile development; Creationline is a Microsoft partner, with strength in DevOps-related technologies.

Traditionally, a large company in Japan outsources software development and still uses the waterfall model. However, Setouchi chose CI&T and Creationline as their partners because their technical excellence could accelerate business for Setouchi. They are using the latest lean/kanban approach, combined with Microsoft Azure technologies. 

### Solution overview ###

Sky Trek is a membership service for travel, including amphibious aircraft. They want to provide a one-stop reservation system for custom tours, using airlines, helicopters, amphibious aircraft, rental cars, and hotels. The system doesn't need a lot of concurrent access, but it does need to quickly adapt to change and enable expansion of the business model while maintaining high-quality service. They already have a good lead time for web deployment. However, they also want to have blue-green deployment for exploratory testing before production and an automated continuous integration/continuous deployment (CI/CD) pipeline.

![Architecture diagram]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/architecture.jpg)

Key technologies used:

  * Web Apps and API Apps features of Azure with C# (Infrastructure as Code)
  * Visual Studio Team Services (CI/CD and release management)
  * Swift (iOS), TestFlight, and *fastlane* (mobile DevOps)
  * Goal integration with Visual Studio Team Services (telemetry)

![Hackfest members]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/allmembers.jpg)

Core team:

  * Geovanne Borges Bertonha ([@geobertonha](https://twitter.com/geobertonha)) – Software Architect (DevOps), CI&T
  * Andre Ogura Dantas – Software Architect (iOS Specialist)
  * Leandro de Lima Machado – Software Engineer
  * Andre Santos Kano ([@drekano](https://twitter.com/drekano)) – Software Engineer
  * Tadahiro Yasuda ([@yasudatadahiro](https://twitter.com/yasudatadahiro)) – CEO, Creationline
  * Daisuke Ono – Creationline
  * Tsuyoshi Ushio ([@sandayuu](https://twitter.com/sandayuu)) – Senior Technical Evangelist (DevOps), Microsoft
  * Naoki (NEO) Sato ([@satonaoki](https://twitter.com/satonaoki)) – Senior Technical Evangelist (Azure Technologist), Microsoft

## Customer profile ##

[Setouchi Holdings](http://setouchi-hd.com/en/), based in Hiroshima, Japan, markets Kodiak aircraft, leases aircraft for its seaplane operations, and is preparing to launch a land and sea aircraft business in the Maldives, Thailand, and the Philippines. Setouchi also offers construction management and engineering services.
 
## Problem statement ##

Setouchi personnel are familiar with Agile development and have solid technical skills. However, they were new to Azure and Team Services. Mobile pipeline automation with Team Services and blue-green deployment on Azure were their biggest issues.

They also had a lot of manual processes, partly because they used a lean approach. They had enough knowledge to automate by using open-source tools. But they intentionally didn't do so because, unless they needed to, they didn't automate. (I think this is the right approach.)

Because they like the kanban system in Team Services, they wanted to integrate Goal, the CI&T internal dashboard system, with Team Services rather than using Jira.

## Solution, steps, and delivery ##

We solved these problems by implementing the following:

* [Value stream sapping](#value-stream-mapping)
* [Blue-green deployment without noisy neighbor problem](#blue-green-deployment-without-noisy-neighbor-problem)
* [Automated "Retain indefinitely" flag](#automated-retain-indefinitely-flag)
* [iOS deployment pipeline automation](#ios-deployment-pipeline-automation)
* [Kanban customization with Test & Feedback extension](#kanban-customization-with-test--feedback-extension)

### Value stream mapping ###

Value stream mapping (VSM) is valid for a project that already has a decent amount of processes and a deployment pipeline. However, Setouchi hadn't used it because they follow the "lean" mindset: They don't do anything unless the project needs it.

However, their system will become much more complex. After a few sprints, their system will have several microservices using web and API apps, mobile, payment gateway, and so on. Instead of designing the entire architecture at the beginning, for the first sprint they focused on the web app. Then they focused on the API, then mobile, and so on. When I held the VSM session, this process was just beginning. That is why they didn't have the deployment pipeline. 

We couldn't see the whole picture at that time. So we discussed the existing development process for a typical web front end and then focused on the web front-end VSM.

![Value stream mapping discussion]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/VSMdiscussion.jpg)  

![Sketch of value stream map]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/ValueStreamMapping.png)  

The lead time for the web site is not bad: two days. My recommendation was to use release management. They had a lot of manual processes for deployment for every environment. They could implement automated testing by themselves. However, they didn't implement release pipelines per environments like Dev/Staging/Production.

We can automate quite easily, reaching 10 deployments per day soon. So I also recommended using feature flags for getting feedback from production.

But Agile development is about "embracing change," right? When we did the VSM, they hadn't started mobile and microservices development; that began at the time of the hackfest. Release management was the same: They still had some manual processes, but they needed to automate the mobile CI/CD pipeline.

Also, they needed to implement blue-green deployment for the new microservices, needed for an API app. Plus they were going to have load testing for the slot in the near future, causing the "noisy neighbor" problem: Load testing for a slot also affects the production slot.

So we settled on the following as the targets for our hackfest:

* Release management to automate deployment for multiple environments 
* [Blue-green deployment]((http://martinfowler.com/bliki/BlueGreenDeployment.html)) for exploratory testing 
* CI/CD for the mobile pipeline to automate previously manual processes

For more information about these practices, see [DevOps Practices](http://www.itproguy.com/devops-practices/) in David Tesar's blog, ITProGuy.com.

Let's get hacking!

### Blue-green deployment without noisy neighbor problem ###

Blue-green deployment itself is easy for us. In Azure, Web Apps and API Apps already have deployment slots and the ability to swap slots. It is a fantastic feature—read [Implement continuous deployment of your app to an Azure web site](https://www.visualstudio.com/en-us/docs/build/apps/cd/deploy-webdeploy-webapps) to understand the functionality.

Team Services uses tasks to support deployment and swapping, which makes pipeline configuration easy. For example, Team Services offers the "Azure App Service Manage" task. Simply set up this task to swap slots without writing a PowerShell script.

![Swap deployment slots]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/deploymentslot.jpg)  

However, this raises the noisy neighbor problem. Naoki Sato provided the solution: Add an App Service plan, and then change the App Service plan of a slot. When you use Web Apps or API Apps, the resources are shared among the slots. But if you change only one slot, the slots have different App Service plans and the apps allocate separate resources among the slots. Using this technique, you can avoid the noisy neighbor problem.

![App Service plans]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/AppServicePlan.jpg)

As a practice of Infrastructure as Code, we use an Azure Resource Manager template after finishing configuration of resources via the Azure portal. (See [Authoring Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates) for more information.) Before deployemnt, we can download the template and the deployment script.

![Infrastructure as Code]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/IaC.png)

<img alt="Downloaded scripts" src="{{ site.baseurl }}/images/2016-11-31-SetouchiHD/IaC2.png" width="384">

If you are using PowerShell, you can deploy it like this.

```powershell
PS> ./deploy -subscriptionId XXXX-XXXX-XXXX-XXXX... -resourceGroupName YourGroup -resourceGroupLocation JapanEast -deploymentName deploy01
```

### Automated "Retain indefinitely" flag  ###

The web service pipeline was already fully automated—except for one thing: After a successful release, they needed to select the "Retain indefinitely" option to keep the successfully deployed artifact for later checking.

![Enabling the "Retain indefinitely" option]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/RetainIndifinitely.jpg)

The Team Services REST APIs would seem ideal to update the status of a release: Simply use System.AccessToken variables to access the API. We tried it. However, we couldn't figure out how to implement it in release management. Instead, we used personal access tokens for this purpose. (As it turns out, we can indeed use System.AccessToken to access OAuth tokens.)

Here's how to request and use a personal access token. First, add a batch file step in your release environment.

```batch
curl --request PATCH -H "Accept: application/json" -H "Content-type: application/json"  -d '{ "keepForever": true }' -u %USER_NAME%:%PERSONAL_ACCESS_TOKEN% %SYSTEM_TEAMFOUNDATIONSERVERURI%DefaultCollection/%SYSTEM_TEAMPROJECT%/_apis/release/releases/%RELEASE_RELEASEID%?api-version=3.1-preview.4
```

Then set the variables of the personal access token in your release definition.

![Setting variables for personal access token]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/PersonalAccessToken.jpg)

Finally, use the "Run script" task to run the script. (If you use Linux or MacOS on your build machine, you can use the same technique with a shell script task.)

![Defining a "Run script" task]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/RunScript.jpg)

### iOS deployment pipeline automation ###

The iOS deployment pipeline presented several challenges, beginning with the need to build three times for their environments. Every artifact must be included in a .ipa file, but we need to change the connection string for the database. We don't have any way to avoid to build three times. Generally speaking, you need to build one time; then you need to pass the artifacts into the release. For this reason, they had only one build pipeline.

![Initial build pipeline]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/InitialBuildPipeline.png)

We improved this step by step. First, we build only for the Dev environment on the build definition as CI. When the Git repository changes, Team Services fires the build definition; we don't want to build every time for the QA/Staging/Production environment. Then we separate the deployment process into release management. We also add the source code to the drop folder, which is a shared directory that contains artifacts, pass release management, and deploy.

However, the problem is the time needed to upload the artifact from the build machine to Team Services, especially for the .ipa file. That is why we remove some unused files and zip the artifact before upload. It works well.

Also, sometimes, we had a problem restoring POD files. When we used the `pod update` command, it occasionally timed out. So we used the `pod install` command instead, which brought restoration time down to 14 seconds. 

![Improved build pipeline]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/ImprovedBuildPipeline.png)

Then we created some release definitions to adopt this strategy. We need to re-build and test on the release pipeline. Doing so is against the rules of continuous delivery, but it is the better solution for the .ipa build.

![iOS release pipeline]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/ReleaseiOS.jpg)

The next challenge was automated deployment for TestFlight and automated submission to the Apple App Store. (They use [HockeyApp](https://hockeyapp.net/) for Android deployment for testing. Currently, they use TestFlight.) We used *[fastlane](https://fastlane.tools/),* which helps us automate iOS and Android deployment and release.

We used the Team Services build agent for Mac. (See [Deploy an agent on OSX](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-osx).) For now, they use their own Mac; they plan to move to MacinCloud. (See [Getting Started with the MacinCloud VSTS Build Agent Plan](https://support.macincloud.com/support/solutions/articles/8000016614-getting-started-with-the-macincloud-vsts-build-agent-plan).) A more-recent option is using [Visual Studio Mobile Center](https://www.visualstudio.com/vs/mobile-center/) to build an iOS application. 

After we created the Fastfile, we can automatically deploy into TestFlight and use HockeyApp to submit to the Apple App Store. It is very convenient.

```ruby
fastlane_version "1.109.0"

generated_fastfile_id "{someid}"

default_platform :ios

before_all do
    ENV["SLACK_URL"] = "{Webhook URL from Slack}}" # Webhook URL created in Slack
    ENV["FL_COCOAPODS_PODFILE"] = "SKY TREK/"
    ENV["PILOT_USERNAME"] = "{Your e-mail address}"
  end

lane :beta do
  # build your iOS app
  cocoapods
  
  gym(
    workspace: 'Sky Trek/Sky Trek.xcworkspace',
    scheme: "Sky Trek Dev"
  )

  testflight(changelog: "Testing fastlane build support")

  slack(
        message: "Successfully distributed a new beta build for testing.",
        success: true
      )

end
```

This code builds iOS apps, pushes artifact into TestFlight, and then pushes a message to Slack. 

Also, we need to change the Fastfile depending on the environment. We created one more artifact directory and added three Fastfiles. After accepting these on the release, we just copy it to Fastfile, according to the environment. Now we were ready to deploy automatically!

![fastlane file structure]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/Fastlanefiles.jpg)

The Fastfile doesn't include submission to the Apple App Store. However, adding that step is easy. The [Apple App Store task](https://marketplace.visualstudio.com/items?itemName=ms-vsclient.app-store) uses *fastlane,* so we decided to keep using *fastlane* to submit the application. After release, we can see the Slack notification from the trigger of TestFlight. We're done!

### Kanban customization with Test & Feedback extension ###

We needed to send some kanban telemetry to Goal, the CI&T dashboard system. Goal is impressive: It collects a lot of telemetry via Jira and determines business complexity based on the story. Generally speaking, we have no way to measure productivity among Agile projects across a company. However, Goal calculates productivity and visualizes it, along with other useful metrics.

![CI&T Goal]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/Goal.png)
  
First, we needed to customize the kanban board column. Then we needed to develop the adapter using the Team Services REST API. We were waiting for the API specification from the Goal team, so we started hacking the two technical elements, both of which were easy. You can not Customizing a product backlog of the scrum template isn't possible, but you can copy the scrum template and customize every work item like a product backlog.

![Work item customization, step 1]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/WorkItemCustomize01.png)

<img alt="Work item customization, step 2" src="{{ site.baseurl }}/images/2016-11-31-SetouchiHD/WorkItemCustomize02.png" width="462">

![Work item customization, step 3]({{ site.baseurl }}/images/2016-11-31-SetouchiHD/WorkItemCustomize03.png)

(After this hackfest, Team Services gained a feature for customizing work item templates without copying the scrum template. See [Use templates to add and update work items](https://www.visualstudio.com/en-us/docs/work/productivity/work-item-template).)

After we get the Goal API specification, we are going to start the next hackfest!

Setouchi was pleased to learn about the [Test & Feedback](https://marketplace.visualstudio.com/items?itemName=ms.vss-exploratorytesting-web) extension in the Visual Studio Marketplace. This is one of my favorite features in Team Services.

## Conclusion ##

In only two days, we automated a lot: web/app swap, CI/CD pipeline for Web Apps and API Apps, and iOS CI/CD pipeline using *fastlane.* They can measure the business outcome using the Goal dashboard. After the hackfest, they can enable 10 deployments per day for their web and API apps.

Here are some of the comments from this hackfest:

>"It's amazing the level that Microsoft has achieved regarding DevOps, and this hackfest was fantastic. It's impressive thinking that two hacking days can help us saving lots of days of productivity." —Geovanne Borges Bertonha, Software Architect

>"The hackfest was very productive and helpful; we've learned a lot of things and improved our continuous integration flow. Amazing. Hope we can have it again someday. Great job!" —Andre Ogura, iOS Specialist

>"I did like very much the hackfest. [It] was an opportunity to improve knowledge and merge ideas. New solutions always are born in these events." —Leandro de Lima Machado, Software Engineer

>"I had the opportunity to know more about Azure and Visual Studio Team Services. It was a great experience." —Andre Santos Kano, Software Engineer

## Other resources ##

* Martin Fowler on blue-green deployment: [BlueGreenDeployment](http://martinfowler.com/bliki/BlueGreenDeployment.html)
* Switching deployment slots in Web Apps: [Implement continuous deployment of your app to an Azure web site](https://www.visualstudio.com/en-us/docs/build/apps/cd/deploy-webdeploy-webapps)
* [Authoring Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)
* [HockeyApp](https://hockeyapp.net/)
* *[fastlane](https://fastlane.tools/)*
* [Deploy an agent on OSX](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-osx)
* [Visual Studio Mobile Center](https://www.visualstudio.com/vs/mobile-center/)
* [Apple App Store](https://marketplace.visualstudio.com/items?itemName=ms-vsclient.app-store) extension for Team Services
* [Use templates to add and update work items](https://www.visualstudio.com/en-us/docs/work/productivity/work-item-template)
