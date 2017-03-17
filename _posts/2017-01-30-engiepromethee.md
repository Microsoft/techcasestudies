---
layout: post
title:  "A hackfest helps ENGIE GEM begin to migrate its applications to Azure"
author: "Julien Corioland and Sébastien Pertus"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-15
categories: [Azure App Service, Azure Functions]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Microsoft France and ENGIE Global Energy Markets partnered to migrate the first of its applications to Azure. Leveraging Azure App Service and Azure Functions, the hackfest team also addressed topics such as security, scheduled execution, and monitoring. 
language: [English]
verticals: [Energy, Financial Services]
---

As part of its project to move applications to Microsoft Azure platform as a service (PaaS), ENGIE Global Energy Markets (GEM) has selected one of its internal projects—*Prométhée*—as a first candidate for the migration. This project is the tool it uses to handle human resources (HR) workloads such as requests for days off and reporting of work hours. 

The project is developed using ASP.NET Core 1.0 and AngularJS 2. All the data is stored in an Azure SQL database. 

The software architecture is actually quite simple, as you can see in the following figure:

![Prométhée Architecture Overview]({{ site.baseurl }}/images/2017-01-30-engiepromethee/architecture-overview.png)


For this hackfest, Microsoft partnered with three teams within ENGIE GEM: the team that develops Prométhée, the DevOps team, and the infrastructure team.

Here is the list of the hackfest participants:

- Amine Bizid – Promethée Project Manager, ENGIE GEM
- Mickael Zaggoury – Promethée Developer, ENGIE GEM
- Mehdi Chemingui – Promethée Developer, ENGIE GEM
- Jinlong Wang – Promethée Developer, ENGIE GEM
- Ousmane Barry – Promethée Developer, ENGIE GEM
- Sacha Ott – DevOps Team, ENGIE GEM
- Mohamed Bellahcene – DevOps Team, ENGIE GEM
- Gregory Hanin – Architecture Team Manager, ENGIE GEM
- Vincent Bouchereau – Service Delivery Manager, ENGIE GEM
- Pierre Coppee – Infrastructure Architect, ENGIE GEM
- Olivier Stucki – Engineer, ENGIE GEM
- João Borras – Senior Consultant Modern Apps, Microsoft France
- [Sébastien Pertus](https://twitter.com/sebastienpertus) – Technical Evangelist, Microsoft France 
- [Julien Corioland](https://twitter.com/jcorioland) – Technical Evangelist, Microsoft France
- [Pascal Sauliere](https://twitter.com/psauliere) – Technical Evangelist, Microsoft France 
 
## Customer profile ##

A leader in international energy markets, [ENGIE Global Markets](http://www.engie-globalmarkets.com/) is the group’s trading platform. It offers a full range of financial products and services to Group BUs and third parties throughout the energy value chain.

ENGIE Global Markets, formerly GDF SUEZ Trading, is a wholly owned ENGIE subsidiary employing nearly 400 people worldwide. Active in energy markets for nearly 20 years, ENGIE is a trusted partner in energy and trading. In 2015, it traded commodity volumes equivalent to 13,350 TWh across the entire energy mix: natural gas, power, crude oil and refined oil products, bulk commodities, and environmental products.
 
Drawing from its long-term industry experience, ENGIE has been active throughout the liberalization of energy markets and strives to serve the energy community day to day.
 
## Problem statement ##

Because this is one of the first projects to be moved to Microsoft Azure PaaS, ENGIE GEM has a lot of concerns about security, the way to deliver the application smoothly, how to execute batch operations, and how to monitor the application in production. They are used to managing these on-premises or with infrastructure as a service (IaaS) applications. 

Actually, this kind of move impacts every team within development, IT, and operations.  

One of ENGIE's first requirements for this application is security: the ability to identify who is using the application and give authorization depending on the role within the company. ENGIE decided to use Azure Active Directory (Azure AD) to do that, using OpenID Authentication for the web front and Azure AD service principal for securing APIs. This part was one of the challenges we addressed during the hackfest.

ENGIE also wanted some help with designing the way it is going to handle side workloads in the application—data import, for example. They were used to using Quartz.NET in their previous .NET project and wanted to move to a lighweight solution that does not need an infrastructure running 24/7, so we proposed the Azure Functions serverless service available on the Microsoft cloud platform.

Another point we handled together during this partnership was to create an automated build-and-release pipeline into Azure, using App Service Web Apps deployment slots and Visual Studio Team Services.

Finally, we had some discussions about how to monitor the application and the PaaS infrastructure, at least for the part where it makes sense.

## Solution, steps, and delivery ##

Because of the different topics to be addressed during the hackfest, we divided our work into three parts:

- Using Azure Active Directory for the application security
- Using Azure Functions to handle some workloads outside of the application
- Cloud architecture design and implementation of a continuous integration/continuous deployment (CI/CD) pipeline 

### Using Azure Active Directory for the application security ###

The application architecture is composed of three parts:

- **HR Service** back end: A web API back end, shared by multiple resources in the ENGIE ecosystem. This web API back end is secured by **Azure Active Directory** and every request must be authenticated within a bearer token.
- **Prométhée** back end: A web API back-end application developed with **ASP.NET Core 1.0** providing specific information. The Prométhée back end is secured under **Windows authentication**, providing information about the currently logged-on user. The Prométhée back end acts as a **daemon service** proxy and is able to make requests to HR Service with its own credentials.
- **Prométhée** front end: An **Angular** application, using the Prométhée back end to access Prométhée information and HR Service information.

![Prométhée Authentication Overview]({{ site.baseurl }}/images/2017-01-30-engiepromethee/azuread-architecture.png)


**Angular workflow**

The Angular services, such as [Http](https://angular.io/docs/ts/latest/api/http/index/Http-class.html), provide a simple way to get the credentials from the current user, using the **withCredentials** option contained in the [RequestOptions](https://angular.io/docs/ts/latest/api/http/index/RequestOptions-class.html#!#withCredentials-anchor) object.

Here is an example, using the Http service from Angular and passing the user credentials during an http request: 

```javascript
getSingle(controllerName: string, id: Number) {
    return this.http.get(WebAPI.url() + '/api/' + controllerName + '/' + id
    , { withCredentials: true })
        .map(request => request.json());
}
```

**Prométhée daemon workflow**

On the other side, the Prométhée back end is configured as a **daemon** and is authenticated to access the **HR Service** web API.
We used [Active Directory authentication library (ADAL)](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) to be able to authenticate the Prométhée back end to HR Service, using the **AuthenticationContext** class.

Every call to **HR Service** is done through a base class, where we can find two different methods.
  
The **GetToken** method:

```cs
private async Task<AuthenticationHeaderValue> GetToken()
{
    if (_authenticationResult == null || _authenticationResult.ExpiresOn <= DateTimeOffset.Now.AddSeconds(1))
    {
        await _semaphoreSlim.WaitAsync();

        if (_authenticationResult == null || _authenticationResult.ExpiresOn <= DateTimeOffset.Now.AddSeconds(1))
            _authenticationResult = await _authContext.AcquireTokenAsync(_audience, _clientCredential);
        _semaphoreSlim.Release();
    }
    return new AuthenticationHeaderValue(Bearer, _authenticationResult.AccessToken);
}
```

The **http call** method:

```cs
public async Task<string> Get(string api, object parm)
{
    _httpClient.DefaultRequestHeaders.Authorization = await GetToken();
    var response = await _httpClient.GetAsync(api + "/" + parm.ToString());
    if (response.IsSuccessStatusCode)
    {
        return await response.Content.ReadAsStringAsync();
    }
    return null;
}
```

**HR Service workflow**

At the end, the **HR Service** web API is secured using the **JWT bearer authentication** through **Azure Active Directory**.  
During the initialization process, the application builder is configured to use the JWT bearer authentication:

```cs
// Token Authentication, used for external app connections
app.UseJwtBearerAuthentication(new JwtBearerOptions
{
    AutomaticAuthenticate = true,
    AutomaticChallenge = true,
    Authority = Configuration["Authentication:AzureAd:AADInstance"] + Configuration["Authentication:AzureAd:TenantId"],
    Audience = Configuration["Authentication:AzureAd:Audience"],
});
```

To secure each call in the API, we are using policies:

```cs
[Route("api/[controller]")]
[Produces("application/json")]
[Authorize(Policy = "DefaultPolicy")]
public class ResourceController : Controller
{
    …
    
```

```
public class DefaultPolicy : AuthorizationHandler<DefaultPolicy>, IAuthorizationRequirement
{
    protected override async Task HandleRequirementAsync(AuthorizationHandlerContext context, DefaultPolicy requirement)
    {
        …


                var claims = ((System.Security.Claims.ClaimsIdentity)context.User.Identity).Claims;

                var appid = claims.FirstOrDefault(p => p.Type.Equals("appid"))?.Value;
                if (appid == null)
                {
                    _logger.LogError("Undefined appid");
                    context.Fail();
                    return;
    }

        if (_appIds.Contains(appid))
                {
                    context.Succeed(requirement);
                }
                else
                {
                    _logger.LogWarning("Unknown appId");
                    context.Fail();
    }

…

```

**Documentation**

This scenario is described in the official Azure Active Directory documentation website:

[Daemon or Server Application to web API scenario](https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-authentication-scenarios#daemon-or-server-application-to-web-api). This section describes a daemon or server application that needs to get resources from a web API. There are two subscenarios that apply to this section: A daemon that needs to call a web API, built on OAuth 2.0 client credentials grant type; and a server application (such as a web API) that needs to call a web API, built on OAuth 2.0 On-Behalf-Of draft specification.

You will find a full sample, involving a daemon website secured within a JWT bearer authentication, in the [Azure samples GitHub repository](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity).

![Dameon to Web API workflow]({{ site.baseurl }}/images/2017-01-30-engiepromethee/azuread-workflow.png)


### Using Azure Functions to handle batch/triggered operations ###

Promethée has a particular scenario in which it has to:

- Upload files in a global blob storage.
- Asynchronously perform several operations on those files.
- Send an email to the relevant persons once the process is done.

To achieve this scenario, we used:

- **Azure Blob storage** to store the files. 
- **Azure Queue storage** to queue a related message, indicating upload is successful for **all** files.
- **Azure Functions** to trigger the queue message, get the corresponding emails, and then call a SendGrid mail task.
- **SendGrid** to be able to send emails from the Azure function.

The upload phase is pretty straightforward. Once the files are correctly uploaded, we queue a message in the blob queue:  

![Uploading files to azure blob and then queue a message]({{ site.baseurl }}/images/2017-01-30-engiepromethee/azure-function-upload.png)


The function trigger action allow us to retrieve the information we need to send emails, thanks to the **SendGrid** service available in Azure:

![Sending emails from Azure function and SendGrid]({{ site.baseurl }}/images/2017-01-30-engiepromethee/azure-function-mail.png)


Note that we **prepare** the mail, but we don't **send** it, in the Azure function. Here is a straightforward sample of how to use SendGrid in an Azure function:

```cs
#r "Microsoft.WindowsAzure.Storage"
#r "Microsoft.Azure.WebJobs.Extensions.SendGrid"

using System;
using Microsoft.WindowsAzure.Storage.Table;
using SendGrid.Helpers.Mail;

public static void Run(out Mail message, string myQueueItem, TraceWriter log)
{
    var personalization = new Personalization();
    personalization.AddTo(new Email("AN_EMAIL_ADRESS@DOMAIN_COM"));

    var messageContent = new Content("text/html", $"Content from the queue :{myQueueItem} .");

    message = new Mail();

    message.AddContent(messageContent);
    message.AddPersonalization(personalization);

    log.Info($"C# Mail sent: {myQueueItem}");
}

``` 

**Azure function and Visual Studio integration**

We used the [Visual Studio tools for Azure Functions](https://blogs.msdn.microsoft.com/webdev/2016/12/01/visual-studio-tools-for-azure-functions/) to be able to track and integrate the function development process within Visual Studio Team Services.

![Visual Studio tools for Azure Functions]({{ site.baseurl }}/images/2017-01-30-engiepromethee/visualstudio-tools-function.png)


Using Visual Studio tools for Azure Functions lets the ENGIE team:

- Allow developers to use their Visual Studio IDE.
- Allow an easier way to [debug and run the functions locally](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local).

Also, we used the [continuous deployment feature](https://docs.microsoft.com/en-us/azure/azure-functions/functions-continuous-deployment) to link the VSTS source code repository to the Azure Functions. By doing this, each time a commit is done on the source code repository, the function is automatically deployed.

### Cloud architecture design and implementation of a CI/CD pipeline ###

In this part of the workshop, the team discussed what would be the best architecture in Microsoft Azure to be able to handle the following:

- A full CI/CD pipeline 
- An up-to-date development environment 
- Ability to execute load tests without impact on production 
- Ability to release a new version with zero downtime 

These are quite strong requirements but actually not so hard to implement when using a PaaS service like Azure App Service.

The key thing to understand is how the Azure App Service plan is working. Basically, the application plan represents the virtual machines that will execute the application and the features that will be available for your web apps. Depending on the service tier you have chosen (Basic, Standard, Premium), you will have access to some features or not. 

For example, if you want to use deployment slots, you need at least to use the Standard tier. The billing is also based on the tier and virtual machine size you choose and, of course, you can run many web apps in one application plan to minimize cost.

But you need to keep a few things in mind: If two web apps share the same application plan, they also share the same virtual machines. That means you should absolutely not share an app plan between production and a test environment in which you plan to execute load tests, for example.

After discussing the different needs of the teams, we agreed on the following architecture:

![Azure Architecture Overview]({{ site.baseurl }}/images/2017-01-30-engiepromethee/azure-architecture-overview.png) 


We do not detail each web app running in each application plan for reading purposes, but the idea is that there are two web apps, one for the API and one for the Prométhée ASP.NET Core MVC back end and Angular front end that are deployed as one.

We had some discussions about the need to split the front end and back end, to be able to have a different lifecycle, but finally the team decided that releasing a new version of the front end will usually require releasing a new version of the back end, and vice versa.

As you can see on the architecture schema above, there are three environments:

- One for development, that is always available in Azure.
- One for test, that can be deployed on demand when they need to execute load tests before releasing a new version.
- One for the production, always available and that contains the two web apps and one *staging* slot for each (this part will be detailed later).

Each environment has its own SQL database and its own Azure Storage account with the queues that are used to trigger new Azure Functions execution. All functions are running on demand into a dynamic application plan—that means an application plan that is provided on demand when a function is triggered. They do not need an always-running infrastructure to handle those workloads!

The most important part here is that this architecture answers the EMGIE GEM key requirements:

- Being able to have an environment that executes the last bits.
- Being able to spawn a new environment to execute load tests (or any other purpose in the future, actually).
- Being able to deploy with zero downtime.

This last point is possible because we decided to use Web Apps slot into the production environment. Basically, a Web Apps slot is nothing but a web app that is linked to another into App Services. In this case, the *staging* slot is only used for doing the latest smoke tests on the application, before swapping it into production in one simple action (handled by Visual Studio Release Management, after an approbation). 

*For more information about staging environments with slots in Azure App Service, see the article [Set up staging environments in Azure App Service](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-staged-publishing).*

After drafting this architecture, we can start to implement a CI/CD pipeline using Visual Studio Team Services.

The development team works with two Team Services Git repositories: one for the HRService API and one for the Prométhée project. They have a master branch on each project, and use feature branching to implement new features according to their backlog.

Each time a feature branch is merged onto the master, a new build and release should be done on the development environment.

When they have a new version ready for production, they create a new release branch from the master, set a label, and trigger a new build. Once the new version of the API or the new version of the Prométhée front end and back end have been built, they can trigger a release that deploys the binaries into Azure App Services.

![HRService Build]({{ site.baseurl }}/images/2017-01-30-engiepromethee/hrservice-build.png)


![HRService Release]({{ site.baseurl }}/images/2017-01-30-engiepromethee/hrservice-release.png)


![Prométhée Build]({{ site.baseurl }}/images/2017-01-30-engiepromethee/promethee-build.png)


![Prométhée Release]({{ site.baseurl }}/images/2017-01-30-engiepromethee/promethee-release.png)


Visual Studio Team Services has almost all of the tasks built in for building and releasing a .NET Core application into Azure App Service. But for some tasks such as stopping and starting an application slot or swapping slots into the release process, we also used the [build and release tasks for Microsoft Azure](https://marketplace.visualstudio.com/items?itemName=geeklearningio.gl-vsts-tasks-azure), available on the Visual Studio Marketplace.

To install it on your Team Services account, you just click **Install** and choose the account on which the extension should be installed:

![Prométhée Release]({{ site.baseurl }}/images/2017-01-30-engiepromethee/vsts-tasks-geeklearnings.png)


Then, all the tasks are available in the tasks picker of the build and release workflow.
 
## Conclusion ##

During the three-day hackfest with ENGIE GEM, we partnered with the team to help them move Prométhée to Microsoft Azure, using PaaS services such as Azure App Service, Azure SQL Database, Azure Functions, and more.

We had different topics to address, such as how to secure a web API using Azure Active Directory, how to continuously deliver an application on the PaaS platform, and how to use Azure Functions to execute some workloads outside the application without having to maintain a 24/7 architecture.

On the last day, we also had some discussions with operations team members, who wanted to understand how they can monitor applications running in PaaS, such as Azure App Service. 

We demonstrated how Azure Application Insights can help handle this kind of monitoring on both technical and usage levels, with collecting performance counters, response times, errors or custom events that are sent using the [Application Insights SDK for ASP.NET Core](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-asp-net-core).

This part was actually the more challenging one for the teams because monitoring PaaS applications is totally different than monitoring infrastructures as they were accustomed to doing.

## Additional resources ##

Last but not least, here are some pointers to additional resources that will help you to deeply understand the concepts and technologies we discussed in this article, as well as pointers to the external tools we used with ENGIE GEM:

- [Azure App Service plans in-depth overview](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)
- [Set up staging environments in Azure App Service](https://docs.microsoft.com/en-us/azure/app-service-web/web-sites-staged-publishing)
- [Microsoft Azure Build and Release Task for VSTS](https://marketplace.visualstudio.com/items?itemName=geeklearningio.gl-vsts-tasks-azure)
- [Try Functions](https://functions.azure.com/try)

