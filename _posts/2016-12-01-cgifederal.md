---
layout: post
title:  "CGI Federal improves efficiency and quality of its ProperPay offering through DevOps"
author: "Kevin Remde"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-12 
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/cgifederal/logo.png" #should be ~350px tall
excerpt: As it moves to embrace Microsoft Azure, CGI Federal seized upon the opportunity to leverage new tools and approaches to optimize the software lifecycle for its products, both in the cloud and on-premises.
verticals: [Public Sector, Financial Services]
language: English

---

![CGI Federal Logo]({{ site.baseurl }}/images/cgifederal/logo.png)

In five short days, CGI Federal and Microsoft worked together to find ways to help streamline CGI's software lifecycle by applying a DevOps mindset to its traditional processes. A joint team collaborated to map the process and identify multiple opportunities to improve the process by introducing approaches like:

-  Automated cloud resource provisioning
-  Database versioning and publication
-  Continuous integration/continuous deployment

### The hack team

The team brought together technical leads from across CGI Federal, joined by Technical Evangelists from Microsoft. 

**CGI Federal’s profiles:**

-	Product manager
-	Test lead
-	Dev Lead
-	3 Developers
-	Database administrator

**Microsoft team:**

-	William Buchwalter
-	Jessica Deen
-	Dan Pilling
-	Kevin Remde
-	Brent Stineman

## Customer profile ##

Since 1976, US defense, civilian, and intelligence agencies have partnered with [CGI Federal](https://www.cgi.com/en) to support their mission-essential needs at every stage of the program, product, and business lifecycle. These partnerships fuel CGI's deep understanding of the challenges and opportunities facing its clients and inform the development of solutions to help them improve outcomes and maximize results.

A key product in CGI Federal’s portfolio is ProperPay. Since 1990, ProperPay and its predecessors have been helping identify and reduce fraud, waste, and abuse (FWA) by providing healthcare claims auditing and recovery.

CGI’s pioneering in this space has carried over into how it embraces new technologies such as cloud computing for new projects. It was during the exploration of technologies such as Microsoft Azure, and engaging with the various Microsoft technologists, that CGI began to question how these technologies could be applied to existing solutions and processes for its existing product lines.

ProperPay is a mature, n-tier solution. Due to the sensitive nature of the data on its operations (healthcare information), and importance of the results produced by the product, a complex, strictly maintained process has developed around each release to ensure quality and privacy.

CGI Federal wanted to investigate DevOps practices to improve the way it develops solutions. First, it wanted the team to fully understand the current development lifecycle, and then hack with Microsoft on some areas of improvement.

## Problem statement ##

As a strategic partner, CGI Federal reached out to Microsoft to see how we could apply our own “cloud first” DevOps processes to help them improve the development processes of ProperPay. This was considered a first step toward helping new versions of various CGI Federal products more easily adopt cloud innovations.

ProperPay is a mature product. Its lifecycle is well established and equally well known to the people who make it happen. What was needed was to take an objective step back, map the process, and look for opportunities to improve it. This would be the focus of our workshop.

A five-day hackfest was proposed, starting with value stream mapping (VSM) activity. This VSM activity would allow us to uncover the most troublesome parts of the processes from the time the first line of code was laid until it was delivered into production. It also helps in identifying which practices are the most appropriate, and to clearly see how the application of these practices will reduce processing times and lead times at every step, as well as in the overall lifecycle. We would then spend any remaining time working to implement changes that would be used to address those challenges.

![Team at Work]({{ site.baseurl }}/images/cgifederal/teamworking.jpg)


## Solution, steps, and delivery ##

The VSM took the first day and a half of the event. The amount of discovery that occurred during this time cannot be overstated. We uncovered manual processes that were unknown to anyone but the people responsible for them. We found points of failure that caused huge amounts of effort to need to be repeated. There were challenges with environment scheduling and “drift” from one environment to another. There were also issues around introducing database changes and preparing for schema changes.

![VSM]({{ site.baseurl }}/images/cgifederal/vsm.jpg)


Key problem areas we identified for enhancements were:

- Environmental “drift” and manual deployments.
- Database release preparations.
- Report release preparations.
- Build/deploy as a single operation.
- Lack of experience with automation tools/technologies.

Accordingly, we were able to recognize these DevOps best practices as being applicable at various stages of their development lifecycle:

- Infrastructure as code
- Automated testing

With these findings in place, we divided the team into three groups to pursue building out some technical muscles and building some technical proofs of concepts:

- Separating build and deploy
- Database/report release automation
- Automated provisioning of environments

This work, combined with upskilling/mentoring, would occupy the remain three days of the workshop.

#### Azure Resource Manager

CGI Federal brought some of the best ProperPay developers to the event. However, three of them had no prior exposure to Microsoft Azure. Brent Stineman took point to help them get skilled on the Azure Resource Manager model while simultaneously helping craft a way for Resource Manager templates and PowerShell scripts to be run from their existing automated build solution (Octopus).

CGI Federal has no interest in moving from its local Team Foundation Server to Visual Studio Team Services. Its local configuration of Octopus (plus Atlassian Bitbucket and other tools) as its deployment solution must remain in place, but will be further used to launch automated deployments in Azure.

Training up the developers covered everything from the redemption of Microsoft Developer Network (MSDN) Benefits for Azure to performing complex deployments via Azure PowerShell and Resource Manager. The initial deployments start using the GitHub quickstart templates and the “Deploy to Azure” button. The developers were also shown resources.azure.com where they could see the deployed resources and how the Resource Manager resource providers viewed them.

Since existing versions of the ProperPay solution are deployed on physical and virtualized infrastructure, the developer training focused on Azure’s infrastructure as a service (IaaS) solutions—the deployment of virtual machines, software-defined networking, and resource management, everything from deploying a single virtual machine to deploying a cluster of virtual machines across multiple subnets.

Their education moved to increasingly complex scenarios where they were eventually able to craft their own basic Resource Manager templates and deploy them via PowerShell. This set the stage for the final step where the automation could be brought in.

Brent adapted some existing samples and demonstrated how self-signed certificates, the use of Azure Active Directory, and service principles could be used to allow a script being run by Octopus to leverage Azure’s role-based access control (RBAC) to execute scripts just like a user would.

```
# Create Azure AD App object for new Service Principal
$adApp =
New-AzureRmADApplication `
    -DisplayName $adAppName `
    -HomePage $adAppHomePage `
    -IdentifierUris $adAppIdentifierUri `
    -KeyCredentials $keyCredential

$principleID = New-AzureRmADServicePrincipal  `
    -ApplicationId $adApp.ApplicationId

New-AzureRmRoleAssignment `
    -RoleDefinitionName Owner `
    -ServicePrincipalName $adApp.ApplicationId
```

The solution centered around a self-signed certificate with a common name attribute using the SHA256 hash algorithm that could be used in the creation of an Azure Active Directory application. This application can then be associated with the service principal and granted specific permissions within Azure. Once created and authorized, PowerShell could then use the certificate thumbprint to retrieve the certificate from the local machine’s certificate store and authenticate against Azure.

```
Login-AzureRmAccount `
    -ServicePrincipal `
    -TenantId $tenantId `
    -ApplicationId $appId `
    -CertificateThumbprint $certThumbprint
```

Additional details on the approach along with a link to the source code samples can be found at: [https://brentdacodemonkey.wordpress.com/2016/12/07/azure-administration-with-certificates/](https://brentdacodemonkey.wordpress.com/2016/12/07/azure-administration-with-certificates/)

#### Automation

Jessica Deen and a developer from CGI Federal worked on automating uploads of SQL RDL reports using PowerShell. The code examples for this are in our CGIFederal repo under "Devops Hackfest\Jessica\RDL Demo".

Below is an example of the SQL RDL PowerShell script used:

![Using rdl_test.ps1]({{ site.baseurl }}/images/cgifederal/rdl_test_example.png)


Additional demo examples can be found in the same file:

- cgi-upload-test.ps1
- cgi-upload-test.ps2

#### Infrastructure as code (IaC)

Jessica Deen also adapted existing samples of three PowerShell scripts for Azure where you could capture an image of a system, copy the newly captured image to a public or private blob storage account, and then use a deployment script to copy that image for reuse and subsequent deployment. The code examples for this are in our CGIFederal repo under "Devops Hackfest\Jessica". Prior to this project, CGI Federal did not do any kind of image capturing to be able to reproduce its current environments, if needed, for devs and future testing. 

Three scripts are needed to log on to Azure, capture an image, and then deploy the image to a new resource group.

- login.ps1

  Logs the user into both Azure CLI and Azure PowerShell.

- captureimage.ps1

  Uses positional parameters to capture the resource group name and virtual machine name in plain text when executing the PS script. The script will then capture the virtual machine created using the previous createbasevm.ps1 script and copy the image VHD to a public storage account. After the copy completes, an Image URI is printed to the output of the screen for the end user to use with the following script.
  
  ![Using captureimage.ps1]({{ site.baseurl }}/images/cgifederal/captureimage_example1.png)
  

- deployVM.ps1

  Uses positional parameters to capture a NEW resource group name and Image URI from the third script in plain text when executing the PS script. The script will then copy the VHD from the public storage account to the user's local storage account in their subscription. From there, the script will use the image to complete the deployment using the associated JSON template files, which can be found in the templates folder.
  
  ![Using deployVM.ps1]({{ site.baseurl }}/images/cgifederal/deployVM_example1.png)
  

  - deployVM.ps1 creates a Custom Storage Account Parameters file as seen below:

    ![Using CustStorageAcct.parameters.json]({{ site.baseurl }}/images/cgifederal/CustStorageAcct.parameters_example.png)
  
  
  - deployVM.ps1 creates a Custom Gold VM Parameters file as seen below:

    ![Using CustomGoldVM.Parameters.json]({{ site.baseurl }}/images/cgifederal/CustGoldVM.parameters_example.png)
  

*The above scripts are just samples and need to be used with template files, not provided.*

Another aspect of infrastructure as code required a virtual machine to be configured with:

- Specific roles and features installed.
- A second hard disk installed, partitioned, and formatted.
- The possibility of files downloaded.
- Scripting run to perform any additional automation.

Along with web services, CGI Federal required these machines to have MSMQ installed.

In order to demonstrate this, Kevin Remde created and provided sample Resource Manager templates, PowerShell DSC scripts, and a PowerShell script that launched the creation of the required resource group and resources. The PowerShell DSC scripting installed the requested roles and features (including MSMQ), as well as adding and formatting the additional storage disk. (As a bonus, at their request, the DSC also sets the time zone of the virtual machine to Eastern Standard Time.)

While Kevin was working on this, two team members from CGI Federal were able to find scripting that would configure MSMQ in an automated way. While we didn't have time to add their learnings and script to the framework that Kevin built, it was encouraging, and holds promise to be a huge improvement to the tedious manual configuration of MSMQ they were previously doing with every new machine configured.

The script sample automates the login to Azure, creates and assigns variables, finds a unique DNS name for the webserver, creates the resource group, and finally launches the Azure Resource Manager deployment

All source code is found at the repo link in the Source Code section below.

The end result is a Resource Group with this set of resources:

![Resource Group]({{ site.baseurl }}/images/cgifederal/krrg.JPG)


The machine has an F: drive partitioned and formatted, the time zone set to Eastern Standard Time, the Web and MSMQ roles installed, and the sample files downloaded and unzipped.

## Conclusion ##

- Key conclusions:
	- Automation is key. Consistent and reproducible execution eliminates manual error and reduces scrap rate.
	- Current total process time (PT) of 20 days and total lead time (LT) of 36 days can be greatly reduced by the implementation of recommended improvements such as smaller batch sizes, better code coverage in testing, and consistent and reproducible automation for cloud infrastructure creation or report uploads.  
	- The team did not make any estimates on improvements to PT and LT as a result of implementing suggested recommendations, but encourages CGI Federal to use the VSM as a guide and a living document to revisit and update in an on-going basis.

- General lessons:
    - Each one of the identified key problem areas in the VSM offered unique challenges and required different levels of training/expertise.
    - It is important to note the SQL/RDL Reports upload PowerShell script for the Database/Report release automation must be run each time for each upload at this time.
    - Many of the scripts and templates we worked with the customer to compile can be adapted for reuse in other, similar environments.
    - Looking at the big picture and the total lead time, we really were able to see where the wasted time was. If we automate, we know that what we build will succeed before we deploy. We can maintain a better quality and reduce our scrap rate.
    - We learned that our project/process is much bigger than imagined. We dove into the pain points—MSMQ, for example. Queues had to be added manually along with rules, triggers, and so on, so we built the automation to make it simple.

- Opportunities going forward:
	- The customer expressed a desire to take the scripts and template files we started in the three-day workshop and expand them to more thoroughly address the key problem areas identified through the VSM. 
	- If we can automate the DB deployments, that would free up a lot of database administrator (DBA) bandwidth to focus on production issues.
	- Build up environments ... test ... and destroy 'em.  

CGI Federal feedback:
> "It's great to see that we've made the connection of how these improvements will actually improve the business."  
    
## Source code ##

- Brent's example of using a certificate to authenticate against Azure Active Directory to perform management operations against Azure's Resource Manager API: [https://github.com/brentstineman/PersonalStuff/tree/master/PowerShell/ServicePrinciple](https://github.com/brentstineman/PersonalStuff/tree/master/PowerShell/ServicePrinciple)
- Jessica's infrastructure as code sample: [https://github.com/jldeen/CDemo](https://github.com/jldeen/CDemo)
- Kevin's infrastructure as code sample: [https://github.com/KevinRemde/CTest](https://github.com/KevinRemde/CTest)

## Additional resources ##

- Brent's blog post on [Azure Administration with Certificates](https://brentdacodemonkey.wordpress.com/2016/12/07/azure-administration-with-certificates/).
