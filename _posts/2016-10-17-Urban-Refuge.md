---
layout: post
title:  "Building a Xamarin app with Urban Refuge"
author: "James Sturtevant"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2016-12-12
categories: [Mobile Application Development with Xamarin, Mobile DevOps, Azure App Service]
color: "blue"
#image: "{{ site.baseurl }}/images/urbanrefuge/urbanrefuge.png" #should be ~350px tall
excerpt: Microsoft teamed up with Urban Refuge to give urban refugees in Amman, Jordan, access to local assistance opportunities via a Xamarin cross-platform mobile application.
language: English
verticals: [Public Safety, Transportation & Logistics, Smart Cities]
---

*This report was updated with additional resource URLs on February 27, 2017.*

Microsoft teamed up with [Urban Refuge](http://www.urbanrefuge.org/) to bring urban refugees in Amman, Jordan, access to local assistance opportunities via a Xamarin cross-platform mobile application. Urban refugees make up 78% of the 655,000 registered Syrian refugees in Jordan and 66% of refugees worldwide. Evidence from the field shows this population has access to mobile devices, yet largely share information via word of mouth. Urban Refuge's mission is to enable access to aid by leveraging technology to address information asymmetries in the urban refugee experience.

Core team:

- Kate Skow – Director of Technology & Design, Urban Refuge
- Victoria Kelberer-McKee – Co-founder and CEO, Urban Refuge
- James Sturtevant ([@aspenwilder](https://twitter.com/aspenwilder)) – Senior Technical Evangelist, Microsoft
- Gavin Bauman ([@gavination2](https://twitter.com/gavination2)) – Technical Evangelist, Microsoft


## Solution overview ##

To solve the problem of access to aid and circumvent social stigma facing urban refugees in Jordan, a solution was developed to create a mobile application that provides information on assistance opportunities, with the listings made available in English and Arabic. A management portal allows the Urban Refuge team the ability to update assistance opportunities as they become available.

The mobile application is a Xamarin Forms cross-platform mobile application for Android and iOS that communicates with an ASP.NET Core API and management portal hosted in the Web Apps feature of [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/). [Azure B2C](https://azure.microsoft.com/en-us/services/active-directory-b2c/) was chosen for authentication, which enables the Urban Refuge team to focus on providing assistance opportunities while alleviating concerns around security. [HockeyApp](https://azure.microsoft.com/en-us/services/hockeyapp/) is integrated into the mobile application so custom events can be collected, informing the Urban Refuge team which assistance opportunities are most used.  

Using the DevOps practice of Automated Deployment, the Urban Refuge team uses Visual Studio Team Services (VSTS) to deploy to their beta testers on demand via the HockeyApp integration. The testers can then submit bug reports directly back to the Urban Refuge team, shortening the feedback cycle.

> "The way that Urban Refuge has been built allows the app to be extremely flexible and applicable in contexts globally. While the specific platform for the launch will be tailored to Amman, focusing on Syrian users, the use of Xamarin and intuitive platforms will make it relatively easy to translate this app to new markets." - Kate Skow, Urban Refuge

## Customer profile ##

The [Urban Refuge](http://www.urbanrefuge.org/) team originally formed at Boston University from a research project conducted in 2015.  Urban Refuge leverages technology to address information asymmetries in the urban refugee experience by providing a mobile application that uses geo-tagged locations for aid resources. The mapping function connects urban refugees with a variety of local assistance opportunities not easily located using traditional addresses. The aim of Urban Refuge is to both improve the urban refugee experience and address administrative inefficiencies that arise when service access is largely through word of mouth. The work Urban Refuge is doing provides humanitarian organizations with a centralized database and accessible contact information to help address administrative coordination issues. 

The Urban Refuge application will be piloted in Amman, Jordan, with the aim of scaling up the technology to be easily modified and applied in a variety of urban refugee areas, including Boston and Turkey.

![urban refuge team]({{ site.baseurl }}/images/urbanrefuge/ur-team.jpg)

 
## Problem statement ##

Urban refugees make up 78% of the 655,000 registered Syrian refugees in Jordan. Although evidence from the field suggests the majority of urban Syrian refugees have access to mobile phones in Jordan, the way refugees currently discover assistance opportunities is largely through word of mouth due to a dearth of accessible online resources and social stigma. During their research, the team at Urban Refuge found that putting the aid on the map in the form of a mobile application enables urban refugees to gain access to vital aid, without the stigma that comes with being a refugee in Jordan. Currently, the Urban Refuge team has a database of more than 160 organizations tagged by geo-coordinates to help refugees find aid.

As the Urban Refuge team began to look at possible technical solutions, they wanted to address several concerns around implementation, feedback, and testing:

1. Urban Refuge is a small team that wanted to be able to develop a solution without the cost and complexity of maintaining multiple codebases. Since the application has the requirement to provide information in both English and Arabic, they wanted to make sure that the localization they did could be applied to both the iOS and Android apps. This was addressed using [Xamarin Forms localization](#4-localization-of-resources).  

2. Urban Refuge needed to make sure they could test the solution in a timely fashion with their beta testers on the ground in Amman, Jordan, to validate all the geo-tagged locations and get direct feedback from the testers. Using the DevOps practice of [Automated Deployment via VSTS and HockeyApp](#6-automated-deployment-via-vsts-and-hockeyapp), they were able to address this concern by enabling builds to be deployed on demand and bypassing official distribution channels.

3. Urban Refuge wanted to be able to understand which resources refugees are searching for and using. The [integration with HockeyApp](#5-usage-metrics-via-hockeyapp-integration) allowed them to be able to see exactly which filters and resources were most popular.

## Solutions, steps, and delivery ##

The solution has several components to it. The most important components are:

1. [Create Xamarin Forms application using iconography](#1-create-xamarin-forms-application-using-iconography)
2. [Create ASP.NET Application API and Management Portal](#2-create-aspnet-application-for-api-and-management)
3. [Integrate Azure B2C for Authentication](#3-integrate-azure-b2c-for-authentication)
4. [Localization of resources](#4-localization-of-resources)
5. [Usage metrics via HockeyApp Integration](#5-usage-metrics-via-hockeyapp-integration)
6. [Automated Deployment via VSTS and HockeyApp](#6-automated-deployment-via-vsts-and-hockeyapp)

A high-level view of the architecture is as follows:

![architecture diagram]({{ site.baseurl }}/images/urbanrefuge/ur-architecture.png)

### 1. Create Xamarin Forms application using iconography

The first step to the solution was to create a Xamarin Forms application for Android and iOS. The Urban Refuge team designed the UX of the application to be simple and straightforward with very little wording. By using minimal words, the design becomes more inclusive for all users despite their level of literacy. An added benefit of using iconography is raising the accessibility level for people who are color blind. You can learn more about inclusive design at [Microsoft Design](https://www.microsoft.com/en-us/design/inclusive). 

![app filter page - inclusive design]({{ site.baseurl }}/images/urbanrefuge/ur-inclusivedesign.png)

To lay out the buttons, create a [responsive grid](https://developer.xamarin.com/guides/xamarin-forms/user-interface/layouts/grid/) in Xamarin Forms XAML. By specifying the image size and using the ```*``` operator on the end row and columns, the grid becomes centered on all screens.  The ```*``` operator fills the remaining space on the screen after allocating space for the other rows/columns.

```csharp
<Grid >
        <Grid.RowDefinitions>
          <RowDefinition Height="100"></RowDefinition>
          <RowDefinition Height="90"></RowDefinition>
          <RowDefinition Height="90"></RowDefinition>
          <RowDefinition Height="*"></RowDefinition>
        </Grid.RowDefinitions>
        <Grid.ColumnDefinitions>
          <ColumnDefinition Width="*"></ColumnDefinition>
          <ColumnDefinition Width="90"></ColumnDefinition>
          <ColumnDefinition Width="90"></ColumnDefinition>
          <ColumnDefinition Width="90"></ColumnDefinition>
          <ColumnDefinition Width="*"></ColumnDefinition>
        </Grid.ColumnDefinitions>

        //Items in grid go here
</Grid>
```

Xamarin Forms allows you to customize the user interface beyond the built-in controls using [Custom Renderers](https://developer.xamarin.com/guides/xamarin-forms/custom-renderer/). There is also a custom component store that lets you use community-created controls quickly. In this case, the 
[Circle Image Control Plugin for Xamarin.Forms](https://github.com/jamesmontemagno/ImageCirclePlugin) was used, which leverages Custom Renderers to create the circular buttons:

```csharp
// one item in the grid 
<ContentView Padding="15"
            Grid.Row="1"
            Grid.Column="1">
    <ic:CircleImage Source="education.png"
                x:Name="Education"
                WidthRequest="90"
                HeightRequest="90">
    <ic:CircleImage.GestureRecognizers>
        <TapGestureRecognizer Command="{Binding FilterByCommand}" CommandParameter="Education"/>
    </ic:CircleImage.GestureRecognizers>
    </ic:CircleImage>
</ContentView>
```

The team could then add images (including high-res images) to each platform-specific project, and Xamarin Forms manages finding the appropriate image to display for each platform.

![adding images to each platform]({{ site.baseurl }}/images/urbanrefuge/ur-images-platform.png)


### 2. Create ASP.NET Application for API and Management 

To make it easier to add and update assistance opportunities, an ASP.NET Core application was created using MVC:

![web dashboard portal]({{ site.baseurl }}/images/urbanrefuge/web-dashboard-list.png)


The ASP.NET Core API is used to serve up the location of the resources in 'json' format, which is then consumed by the Xamarin application:

![web api json view]({{ site.baseurl }}/images/urbanrefuge/web-api-json.png)


The database schema was designed to allow for the expansion of the application as more regions and languages are brought online for the application. When a request is made, the secondary language is specified (in this case Arabic) and retrieves the correct string and returns the values back to the Xamarin application.  

![web database schema]({{ site.baseurl }}/images/urbanrefuge/web-database-schema.png)


### 3. Integrate Azure B2C for Authentication 

The solution is being secured using [Azure B2C](https://azure.microsoft.com/en-us/services/active-directory-b2c/). The team at Urban Refuge is planning to enable customization features to the application that will enable the users to save preferences and custom maps per user. We used the social logon features of Azure B2C to allow users to sign up with existing accounts. 

The Management portal is also being secured by Azure B2C to ensure only the Urban Refuge staff has access. The Urban Refuge team uses the Management portal to update assistance opportunities as new ones become available and as information about existing opportunities changes.

Azure B2C is being used to authenticate calls from the Xamarin Application to the ASP.NET Core API and for the ASP.NET Core Management Portal. The diagram below outlines who has access to which parts of the application.

![application access]({{ site.baseurl }}/images/urbanrefuge/application-access.png)


There are two parts to the setup for the project:

1. [Xamarin Configuration](#xamarin-configuration)
2. [ASP.NET Core Configuration](#aspnet-core-configuration)

> What I love about this project is that you made sure that the security was there, which was extremely important from a political and ethical standpoint to protect these communities. - Professor Noora Lori, Boston University

#### Xamarin Configuration

After setting up an [Azure B2C tenant in Azure](https://azure.microsoft.com/en-us/documentation/articles/active-directory-b2c-get-started/), the authentication is integrated into the Xamarin application using the [Azure B2C library](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet). When the logon page loads, there is an initial check to see if the user has already logged on:

```csharp
//in on view is appearing
 try
{
    string token = await this._authenticaitonService.AquireTokenSilently();
    if (IsLoggedIn(token))
    {
        // have token so already logged in so move them to next page.
        // otherwise just stay here and let user login
        _userMetricsService.TrackEvent("AutoLoggedInOccured");
        await CoreMethods.PushPageModel<ResourceFilterViewModel>();
    }
}
catch (Exception exception)
{
    // Do nothing which will display login screen
    Debug.WriteLine($"Something went wrong during silent authentication. User was not authenticated: {exception}");
}

// is logged in method
private static bool IsLoggedIn(string token)
{
    return !string.IsNullOrWhiteSpace(token);
}
```

If the user is not logged on to the application, clicking the logon button initiates the authentication flow with the following code:

```csharp
//in sign in button click handler
try
{
    IsBusy = true;
    // this will kick off platform specific login screen flow
    var token = await _authenticaitonService.AquireTokenViaUserLogin();
    if (IsLoggedIn(token))
    {
        _userMetricsService.TrackEvent("ManualLoginOccured");
        await CoreMethods.PushPageModel<ResourceFilterViewModel>();
    }
    else
    {
        MessagingService.Current.SendMessage(MessageKeys.DisplayAlert, new MessagingServiceAlert()
        {
            Title = DisplayMessages.Authentication.Title,
            Message = DisplayMessages.Authentication.Message,
            Cancel = DisplayMessages.AcceptMessage
        });
    }
}
catch (System.Net.WebException certException)
    when ((int)certException.Status == 9)
{
    Debug.WriteLine($"Error with cert: {certException}");
    DisplayCertError();
}
catch (Exception ex)
{
    Debug.WriteLine($"An error has occurred. Exception message: {ex.Message}");
}
```

The library manages the OAuth logon flow through different screens for obtaining an access token to use when calling the API. However, on iOS and Android it requires additional platform-specific code to finished the configuration. A custom renderer for each of the platforms allows for hooking into the rendering pipeline to pass the ```PlatFormParameters``` to the Azure B2C library being used in the Xamarin Forms project.

On iOS, the custom renderer looks like:

```csharp
using Xamarin.Forms;
using Xamarin.Forms.Platform.iOS;
using Microsoft.Identity.Client;
using UrbanRefuge.XForms.iOS;
using UrbanRefuge.XForms.Pages;

[assembly: ExportRenderer(typeof(LoginPage), typeof(LoginPageRenderer))]
namespace UrbanRefuge.XForms.iOS
{
    class LoginPageRenderer : PageRenderer
    {
        LoginPage page;
        protected override void OnElementChanged(VisualElementChangedEventArgs e)
        {
            base.OnElementChanged(e);
            page = e.NewElement as LoginPage;
            page.PlatformParameters = new PlatformParameters(this);
        }
    }
}
```

On Android, the custom renderer looks like:

```csharp
using Android.App;
using Microsoft.Identity.Client;
using UrbanRefuge.XForms.Droid;
using UrbanRefuge.XForms.Pages;
using Xamarin.Forms;
using Xamarin.Forms.Platform.Android;

[assembly: ExportRenderer(typeof(LoginPage), typeof(LoginPageRenderer))]
namespace UrbanRefuge.XForms.Droid
{
    class LoginPageRenderer : PageRenderer
    {
        private LoginPage _page;

        protected override void OnElementChanged(ElementChangedEventArgs<Page> e)
        {
            base.OnElementChanged(e);
            _page = e.NewElement as LoginPage;
            var activity = this.Context as Activity;
            _page.PlatformParameters = new PlatformParameters(activity);
        }
    }
}
```

#### ASP.NET Core Configuration

The ASP.NET Core application is using two types of authentication. The first is using token authentication to authenticate the requests coming from the Xamarin Application. This completes the configuration for the Xamarin application in the previous section. To enable token authentication, you need to configure it in the ```Configure``` method in the ```Startup.cs```:

```csharp
// App config settings
ClientId = Configuration["AzureAD:ClientId"];
AadInstance = Configuration["AzureAD:AadInstance"];
Tenant = Configuration["AzureAD:Tenant"];
RedirectUri = Configuration["AzureAD:RedirectUri"];
SignInPolicyId = Configuration["AzureAD:PolicyName"];

// Authentication using JWT tokens for api
app.UseJwtBearerAuthentication(new JwtBearerOptions
{
    AuthenticationScheme = "Bearer",
    AutomaticAuthenticate = false,
    AutomaticChallenge = true,
    Audience = ClientId,
    ConfigurationManager = new ConfigurationManager<OpenIdConnectConfiguration>(
        metadataAddress: string.Format(AadInstance, Tenant, SignInPolicyId),
        configRetriever: new OpenIdConnectConfigurationRetriever(),
        docRetriever: new HttpDocumentRetriever() { RequireHttps = true })
});
```

To enforce the authorization on the API, add the ```Authorize``` attribute to the API controller.

```csharp
namespace UrbanRefuge.Web.Controllers
{
    [Authorize(ActiveAuthenticationSchemes = "Bearer")]
    [Route("api/[controller]")]
    public class ResourcesController : Controller
    {
        // controller definition 
    }
}
```

You will notice that the value of ```Bearer``` was passed for ```ActiveAuthenticationSchemes``` in the authorization attribute. In ASP.NET Core it is possible to pass the ```ActiveAuthenticationSchemes``` type, which allows for the configuration of the token authentication configuration for the API and the cookie authentication for the Management portal. This allows for a second authentication mechanism that secures the Management portal and, in the case above, enables the API for bearer only. If there is no bearer token, then the request will be unauthenticated.

To configure the Management portal to use Cookie Authentication, open the ```Startup.cs``` file and add the following to the ```Configure``` method:

```csharp
// Authentication for MVC using cookies
app.UseCookieAuthentication(new CookieAuthenticationOptions()
{
    AuthenticationScheme = CookieAuthenticationDefaults.AuthenticationScheme,
    AutomaticAuthenticate = true,
    LoginPath = "/Account/SignIn"
});
app.UseOpenIdConnectAuthentication(CreateOptionsFromPolicy(SignInPolicyId));
```

And add the following to the ```ConfigureServices``` method:

```csharp
services.AddAuthentication(sharedOptions => sharedOptions.SignInScheme = CookieAuthenticationDefaults.AuthenticationScheme);
```

An account controller must be added so the user can click the sign-in button to initiate the logon flow:

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Http.Authentication;
using Microsoft.AspNetCore.Mvc;

namespace UrbanRefuge.Web.Controllers
{
    public class AccountController : Controller
    {
        // GET: /Account/Login
        [HttpGet]
        public async Task SignIn()
        {
            if (HttpContext.User == null || !HttpContext.User.Identity.IsAuthenticated)
            {
                var authenticationProperties = new AuthenticationProperties { RedirectUri = "/" };
                await HttpContext.Authentication.ChallengeAsync(Startup.SignInPolicyId.ToLower(), authenticationProperties);
            }
        }

        // GET: /Account/LogOff
        [HttpGet]
        public async Task LogOff()
        {
            if (HttpContext.User != null && HttpContext.User.Identity.IsAuthenticated)
            {
                string scheme = (HttpContext.User.FindFirst("http://schemas.microsoft.com/claims/authnclassreference"))?.Value;
                await HttpContext.Authentication.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
                await HttpContext.Authentication.SignOutAsync(scheme.ToLower(), new AuthenticationProperties { RedirectUri = "/" });
            }
        }

        [HttpGet]
        public IActionResult AccessDenied()
        {
            return View();
        }
    }
}
```

Finally, on the Portal Controller add the ```Authorize``` attribute with the ```ActiveAuthenticationSchemes``` property set to the value ```Cookies```:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using UrbanRefuge.Web.Models;
using UrbanRefuge.Web.ViewModels;
using Microsoft.AspNetCore.Authorization;

namespace UrbanRefuge.Web.Controllers
{
    [Authorize(ActiveAuthenticationSchemes = "Cookies", Policy = "Admin")]
    public class RefugeResourcesController : Controller
    {
        //mvc controller implementation
    }
}
```

The API is now configured to use both bearer token and cookie authentication. When a request is made to the API, the ASP.NET Core Middleware will check for the existence of the bearer token. When a request is made to the MVC portal, the ASP.NET Core Middleware will check for the existence of a cookie.  

### 4. Localization of resources 

The application will be deployed in Amman, Jordan, where Arabic is the most commonly spoken language. Although the application was designed with minimal text, there are a few resources that need to be localized such as the app name when on the phone home screen and any messages that might be displayed to the user.

The localization code required both Xamarin Forms PCL code and platform-specific code. First, we added the resource files to the Xamarin Forms PCL folder. Three files were created: ```AppResources.resx```, ```AppResources.en-US.resx```, and ```AppResources.ar.resx```. The first file is required for the localization but the language-specific translations are stored in the respectively named resource file.

![language specific resource files]({{ site.baseurl }}/images/urbanrefuge/ur-forms-resource-files.png)


For the localization to work, we require platform-specific code that needs to be called from Xamarin Forms PCL project to get the cultural information. We used Xamarin's [Dependency Service](https://developer.xamarin.com/guides/xamarin-forms/dependency-service/) to enable the calls from the PCL to the specific platforms.  

First we created an interface called ```ILocalize```:

```csharp
public interface ILocalize
{
    CultureInfo GetCurrentCultureInfo();
}
```

The next step is to write the platform-specific code for both iOS and Android. All of the code and related details on localization can be found in [Xamarin's Localization documentation](https://developer.xamarin.com/guides/xamarin-forms/advanced/localization/), including customizing the app name for each platform. 

To get a localized string in the Xamarin Forms PCL project, the team wrote a helper function that is passed the name of the resource and returns the localized string. It uses the ```ILocalize``` function defined above:

```csharp
public static class Translator
{
    private const string ResourceId = "UrbanRefuge.XForms.Resx.AppResources";

    public static string GetTranslation(string key)
    {
        var localizationService = DependencyService.Get<ILocalize>();
        var ci = localizationService.GetCurrentCultureInfo();

        ResourceManager resmgr = new ResourceManager(ResourceId, typeof(Translator).GetTypeInfo().Assembly);

        var translation = resmgr.GetString(key, ci);

        if (translation == null)
        {
#if DEBUG
            throw new ArgumentException($"Key '{key}' was not found in resources '{ResourceId}' for culture '{ci.Name}'.", nameof(key));
#else
            translation = "No translation availiable";
#endif
        }
        return translation;
    }
}

// usage
string message = Translator.GetTranslation("AuthenticationErrorMessage");
```

### 5. Usage metrics via HockeyApp integration 

One of the important components for the Urban Refuge team was getting feedback about which assistance opportunities refugees are using.  To accomplish this, the team used HockeyApp to collect user metrics every time a filter was chosen in the app. The usage data is anonymous and the Urban Refuge team can use this data to make sure there are enough opportunities available for a given type and can adjust as usage data flows in. This creates a valuable feedback loop from the Urban Refuge app users to the team, allowing the team to utilize the limited business resources they have to maintain the most relevant opportunities for their users. 

Following the blog post by Gavin that covers [how to use HockeyApp metrics in Xamarin Forms](http://theothergavin.net/hockey-app-for-xamarin-forms-no-problem/), the team integrated HockeyApp metrics into the application where each of the filters is used:

```csharp
private async Task ExecuteFilterCommand(string resourceType)
{
    var type = resourceType.ToResourceType();
    if (type == ResourceType.NotSet)
    {
        // handle errors
        Debug.WriteLine($"Resource was not set properly. Should not get here. String passed: {resourceType}");

        DisplayResourceError();

        //stay on this page.
        return;
    }

    _userMetricsService.TrackEvent($"Filter-{resourceType}");
    // Execute the call to get resource
}
```

HockeyApp metrics view:

![language specific resource files]({{ site.baseurl }}/images/urbanrefuge/ur-hockeyapp-metrics.png)


### 6. Automated Deployment via VSTS and HockeyApp 

As a new and small team, the Urban Refuge team did not have any DevOps practices in place before the project began. During conversations, it was identified they were a distributed team with beta testers actively testing in Amman and needed to be able to quickly make updates to the application and deliver them to the testers in a timely and automated fashion. This led to the team choosing the DevOps practice of Automated Deployment to deploy to beta testers using HockeyApp.

In VSTS the project was set up to build the Xamarin application following the tutorials using for [Android](https://blog.xamarin.com/continuous-integration-for-android-with-visual-studio-team-services/) and [iOS](https://blog.xamarin.com/continuous-integration-for-ios-apps-with-visual-studio-team-services/) from the Xamarin blog. Once the ability to build the Xamarin App on demand was configured, the team added the [Deploy to HockeyApp VSTS task](http://www.blogaboutxamarin.com/xamarin-dev-ops-with-vsts-deploying-to-hockeyapp-and-devices/), which allowed them to automate the deployment to their testers. 

![build definition in vsts]({{ site.baseurl }}/images/urbanrefuge/ur-vsts-builddef.png)


By choosing to implement Automated Deployment, the development team located in Boston can make a bug fix and quickly deploy the fix for the testers in Amman, Jordan, in a reliable and automated fashion. As the team grows they will be in good position to look at adopting other practices such as Continuous Deployment.

![HockeyApp deploy and notify setup]({{ site.baseurl }}/images/urbanrefuge/ur-hockeyapp-beta-deploy-notify.png)


## Conclusion ##

The Xamarin cross-platform implementation will allow the Urban Refuge team to begin beta testing with a team in Amman in January 2017, with the official launch date planned for summer of 2017. Ensuring the accuracy of the information being provided via the app is paramount for success and it requires time on the ground in Amman. The team will be using the DevOps practice of Automated Deployment to deploy to the beta testers in Amman in January. The beta testers will be testing the app's functionality in the field along with the accuracy of the location information. The launch is expected to enable refugees to gain access to much needed aid without the stigma they might normally endure.

Beyond the initial launch in Amman, the Xamarin application has future impact potential on communities across the globe and in research.

### Expansion ###

As the Urban Refuge story has gained momentum, there has been a surge in requests to bring this solution to other geographical areas and verticals. To address the surge in interest, the Urban Refuge team plans to open-source the entire application as a solution for other communities. They have many partners lined up to use the app:

- A partnership with [SREO](http://sreoconsulting.com/about-us/), which will help develop assistance opportunities for refugees in the Republic of Turkey.
- A partnership with a faculty member from Scripps College in Claremont, California, to help bring the application to that community. 
- Urban Refuge is hiring new team members to create location points in Boston to provide access for local urban refugees in the fall of 2017.

### Research ###

The integration with [HockeyApp User Metrics](https://www.hockeyapp.net/features/user-metrics/) will not only help the team understand which components of the application are being used the most, but also has an impact on research that originally sparked the creation of Urban Refuge. 

> One of the most exciting things about working on this has been realizing that the way you design the tool has a huge impact on what I can do in the classroom with my students and what I can do as a social scientist researching forced migration and human trafficking. - Professor Noora Lori, Boston University

## General lessons ##

The lessons outlined above, such as HockeyApp Xamarin Forms integration, multiple authentications in ASP.NET, and how to implement localization on Xamarin Forms, are generally applicable to many projects. Another lesson not outlined above but that is applicable to open source projects is how to hide secrets in the source code yet still be a part of an automated deployment process.

Because the Urban Refuge team will be open-sourcing the entire project, one of the concerns that came up was how to host the code without any secrets in the source yet still be able to integrate with Visual Studio Team Services and HockeyApp for automated deployment. As part of the solution, the team leveraged VSTS Extension Marketplace to add an extra step into the build process that finds placeholder tokens in the code and replaces them with the secrets. This allows the code to be open source and still use VSTS for build and deploy. The entire step-by-step process is outlined in the [Build and deploy VSTS projects without sharing secrets](http://www.jamessturtevant.com/posts/Build-and-Deploy-Opensource-project-with-Visual-Studio-Team-Services-without-sharing-secrets) blog post.

## Additional resources ##

[Urban Refuge website](http://www.urbanrefuge.org/)

[Registered Syrian refugees (UNHCR)](http://data.unhcr.org/syrianrefugees/country.php?id=107)

[Refugee cell phone coverage](http://news.psu.edu/story/350156/2015/03/26/research/ist-researchers-explore-technology-use-syrian-refugee-camp) 

[Behind the Scenes of Other Technical Case Studies](https://channel9.msdn.com/Blogs/DevRadio/)

