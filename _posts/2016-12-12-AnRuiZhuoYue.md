---
layout: post
title:  "Using Azure App Service to develop a conference-management system for AnRuiZhuoYue"
author: "Shijun Liu"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-02-23
categories: [Azure App Service]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Digital marketing provider AnRuiZhuoYue worked with Microsoft to develop an intelligent conference-management system with facial registration and check-in functions, based on a WeChat app and using Azure App Service features.   
language: [English]
verticals: [Digital Marketing, Communications/Media]
---


Digital marketing provider AnRuiZhuoYue worked with Microsoft to develop an intelligent conference-management system with facial registration and check-in functions. This system's front end is based on the widely used WeChat app. Its back end is hosted on the Web Apps feature of Azure App Service and powered by Microsoft Cognitive Services. The project team also helped migrate AnRuiZhuoYue's digital marketing WeChat solution to Azure.

WeChat is the center of digital marketing strategy in China. More than half of its 500 million users open the app at least 10 times a day. Its solutions can be hosted on Web Apps. The ability to scale out and scale up makes Azure App Service ideal for WeChat live-campaign scenarios.

**The project team:**

* Qisheng Zhu –	GM, AnRuiZhuoYue 
* Weiqi Ma –	Dev Manager, AnRuiZhuoYue 
* Feihong Gan –	Developer, AnRuiZhuoYue 
* Malgosia Mazany – DX Audience Evangelism Manager, Microsoft
* Yan Zhang – DX Audience Evangelism Manager, Microsoft China
* Zepeng She – DX Technical Evangelist, Microsoft China 
* Rita Zhang – DX Technical Evangelist, Microsoft
* Bhargav Nookala – DX Technical Evangelist, Microsoft
* Haishi Bai – DX Technical Evangelist, Microsoft
* Zepeng She – DX Technical Evangelist, Microsoft
* Qixiao Wang –	DX Technical Evangelist, Microsoft China 
* Shijun Liu –	DX Technical Evangelist, Microsoft China 

## Customer profile ##

[AnRuiZhuoYue](http://www.anruichina.com/) is the leading end-to-end marketing services provider in China. Its services include content marketing, digital marketing, and online to offline (O2O) marketing. It won the *Microsoft Global MSP Excellence Awards for Supplier of the Year 2016*, and is also a content and marketing supplier for Apple, Oracle, Lenovo, and China Telecom.
 
## Problem statement ##

### *Business pain points* ###

In today's Internet environment, AnRuiZhuoYue's customers need more digital tools to improve their offline campaign experiences, such as for client registration, check-in, and feedback surveys at marketing roadshows or workshops. Traditional event management has several disadvantages: 

* 	No self-services portal. 
* 	No fully mobile support and social integration. 
* 	Need to manually check in. 
* 	No real-time data collection and analysis. 

So AnRuiZhuoYue plans to develop a self-service registration, check-in, and feedback system based on WeChat for offline events.

### *Technical pain points* ###

* When an audience checks in at an offline event, large amounts of concurrent requests go to the virtual machine. AnRuiZhuoYue has to spend time scaling out instances and load balancing. It needs a method to easily manage the multi-instances.
*	Self-service check-in has risks such as someone checking in on behalf of another. Face identification is ideal for strengthening the process of self-service check-in.

AnRuiZhuoYue plans to develop this system with Cognitive Services Face API, and host its back end on a web app for load balancing.

## Solution, steps, and delivery ##

### *Solution delivery* ###

AnRuiZhuoYue and Microsoft worked together to deliver an end-to-end WeChat conference management solution, including a WeChat app for offline events, a management portal that also works as a back-end server, a Universal Windows Platform (UWP) and auto-deployment Azure Resource Manager template. 

* The WeChat app includes:
    * *Basic features:* Users can follow this app in WeChat to sign up for the event, upload personal information, and get notifications from the organizers.
    * *Face registration and check-in features:* Users can upload photos of faces to the WeChat facial database and test to see if they can be identified.

* The management portal includes:
    *	*Management features:* The operator can monitor all users through this portal. 
    *	*Back-end service:* It provides a check-in API for the mobile app.

* The UWP mobile app includes:
    *	*Onsite check-in feature:* Attendees can check in using a mobile device onsite provided by the organizer. 

* The auto-deploy template includes:
    *	*Azure Resource Manager template:* This template is for Azure App Service, Storage, SQL Database creation, and code package deployment.
    *	*Deployment project:* This is used to sign in and trigger the deployment process for users.

*Figure 1. Face uploaded in WeChat*

![Face Upload Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/H5Capture.jpg)


*Figure 2. WeChat response*

![Wechat Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/WechatCapture.jpg)


*Figure 3. Face detection UWP*

![UWP Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/UWPCapture.jpg)


*Figure 4. Management portal*

![Management Portal Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/ManagementPortal.png)


### *Technical architecture* ###

This solution leverages Web Apps, Azure SQL Database, Azure Storage, and Cognitive Services.

* The back end of the WeChat app is actually a web server, hosted on a web app.
* The management portal and the API service back end are hosted on a web app.
* Images are stored in Azure Blob storage.
* User data is stored in a SQL database.
* Face detection and identification are implemented with Cognitive Services.

*Figure 5. Technical architecture*

![Technical Architecture]({{ site.baseurl }}/images/AnRuiZhuoYue/Architecture.png)


### *Activities* ###

We hosted two hackfests with AnRuiZhuoYue during which we helped them migrate the solution to Azure.

*Figures 6a, 6b, 6c. Hackfest photos*

![Hackfest Photo]({{ site.baseurl }}/images/AnRuiZhuoYue/Hackfest.jpg)


![Hackfest Photo]({{ site.baseurl }}/images/AnRuiZhuoYue/HackfestGroup.jpg)


![Hackfest Photo]({{ site.baseurl }}/images/AnRuiZhuoYue/HackfestWhiteBoard.jpg)


### *Technical Implementation* ###

After several months of work, we delivered a code package based on .NET and a WeChat API, including a WeChat app back end, API server, and management web page.

*Figure 7. WeChat back-end code*

![Wechat Backend Code Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/CodeCapture.png)


All of these web servers are hosted on a web app with some advanced features being used, especially the WeChat back-end server.

* We leveraged the autoscale feature to handle the high number of concurrent requests when the WeChat campaign activities are being hosted. This decreases their IT operations efforts compared with using their local servers.  
* We found that WeChat sometimes blocks the Azure domain name. We could bind the custom domain name with our web app to solve this.
* They used the remote debugging feature of Web Apps to check their SQL database connection issue.

*Figure 8. Using the Web Apps feature of Azure App Service*

![Azure Web App Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/AzureWebApp.jpg)


They also leveraged Azure Storage and SQL Database for storing data.

*Figure 9. Azure Storage*

![Azure Storage Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/AzureStorage.jpg)


*Figure 10. Azure SQL Database*

![Azure SQL Database Capture]({{ site.baseurl }}/images/AnRuiZhuoYue/AzureSQLdb.jpg)


The auto-deploy Azure Resource Manager template helped AnRuiZhuoYue quickly set up the Azure environment. Part of the JSON file is shown here:

```ARM
  {
      "name": "[parameters('WebAppName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [
        "[concat('Microsoft.Web/serverfarms/', parameters('WebAppSvcPlanName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', parameters('WebAppSvcPlanName'))]": "Resource",
        "displayName": "WebApp"
      },
      "properties": {
        "name": "[parameters('WebAppName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms/', parameters('WebAppSvcPlanName'))]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "config",
          "dependsOn": [
            "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
          ],
          "tags": {
            "displayName": "WebAppConfig"
          },
          "properties": {
            "phpVersion": "5.6",
            "netFrameworkVersion": "v4.6",
            "use32BitWorkerProcess": false,
            "webSocketsEnabled": true,
            "alwaysOn": false,
            "remoteDebuggingEnabled": false
          }
        },
        {
          "name": "appsettings",
          "type": "config",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
          ],
          "tags": {
            "displayName": "WebAppAppSettings"
          },
          "properties": {
            "Oxford:FaceAPIPrimaryKey": "[parameters('CognitiveKey')]",
            "AppUrlBase": "[concat('http://',parameters('WebAppName'),'.chinacloudsites.cn')]",
            "blob:Account": "[parameters('StorageAccountName')]",
            "blob:Key": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('StorageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).keys[0].value]",
            "wx:AppId": "[parameters('wxAppId')]",
            "wx:AppSecret": "[parameters('wxAppSecret')]",
            "ef:ConnectionString": "[concat('Server=tcp:', reference(concat('Microsoft.Sql/servers/', parameters('SQLServerName'))).fullyQualifiedDomainName, ',1433;Database=', parameters('SQLDBName'), ';User Id=', parameters('sqlAdministratorLogin'), '@', parameters('SQLServerName'), ';Password=', parameters('sqlAdministratorLoginPassword'), ';Trusted_Connection=False;Encrypt=True;Connection Timeout=30;')]"
          }
        },
        {
          "name": "MSDeploy",
          "type": "extensions",
          "location": "[resourceGroup().location]",
          "apiVersion": "2015-08-01",
          "dependsOn": [
            "[concat('Microsoft.Web/sites/', parameters('WebAppName'))]"
          ],
          "tags": {
            "displayName": "WebAppMSDeploy"
          },
          "properties": {
            "packageUri": "[variables('packageURI')]"
          }
        }
      ]
    }
```

The deployment project is implemented as a console application. An IT Pro can set up the environment and deploy the code by signing in with an Azure account, powered by Azure Active Directory. Part of the .NET code is shown here:

```.NET
public static void Main(string[] args)
{

            //Deploy Parameters
            var groupName = "";
            var rgPara = new ResourceGroup("China North");
            var subscriptionId = "";
            string ClientId = "";
            var deploymentName = "";

            //Resource Parameters
            ...

            //Get Token
            var token = GetAccessTokenAsync(ClientId);
            var credential = new TokenCredentials(token.Result.AccessToken);

            //Create Parameter Json
            string parametersJson = UpdateParameterJson(webAppName, ...);
        
            //Create ARM
            var dpResult = CreateTemplateDeploymentAsync(credential, rgPara, groupName, deploymentName, subscriptionId, parametersJson);
            Console.WriteLine(dpResult.Result.Properties.ProvisioningState);

            Console.WriteLine("Press Enter to exit");
            Console.ReadLine();
        }
        private static async Task<AuthenticationResult> GetAccessTokenAsync(string clientId)
        {
            Uri rUri = new Uri("https://login.chinacloudapi.cn");
            var context = new AuthenticationContext("https://login.chinacloudapi.cn/common");
            var token = context.AcquireToken("https://management.chinacloudapi.cn/", clientId, rUri);
            if (token == null)
            {
                throw new InvalidOperationException("Could not get the token.");
            }
            return token;
        }
        public static async Task<DeploymentExtended> CreateTemplateDeploymentAsync(
          ServiceClientCredentials credential,
          ResourceGroup rgPara,
          string groupName,
          string deploymentName,
          string subscriptionId,
          string parametersJson)
        {
            Console.WriteLine("Creating the template deployment...");
            var resourceManagementClient = new ResourceManagementClient(new Uri("https://management.chinacloudapi.cn/"), credential)
            { SubscriptionId = subscriptionId };
            DeploymentExtended aa = null;
            try
            {
                var result = resourceManagementClient.ResourceGroups.CreateOrUpdateAsync(groupName, rgPara).Result;
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }
            var deployment = new Deployment();
            deployment.Properties = new DeploymentProperties
            {
                Mode = DeploymentMode.Incremental,
                Template = File.ReadAllText("..\\..\\WebappTemplate.json"),
                Parameters = parametersJson
            };

            try
            {
                aa = await resourceManagementClient.Deployments.CreateOrUpdateAsync(
                groupName,
                deploymentName,
                deployment);
            }
            catch (Exception e)
            {
                Console.WriteLine(e.Message);
            }
            return aa;
}
```
 
## Conclusion ##

As a result of this technical engagement, AnRuiZhuoYue is positioning this solution as the company's software as a service (SaaS) product. It has published the solution to one of its WeChat public accounts: AnRuiEvent. This product is hosted entirely on Microsoft Azure. 

This solution also will be promoted on the [official Azure China website](https://www.azure.cn) as a pilot of a digital marketing solution based on WeChat and Azure, delivered by a Microsoft partner. This promotion will help attract WeChat partners in order to grow the Azure business in China.

From a technical aspect, we demonstrated that Azure App Service can handle the load-balancing demands of a WeChat live campaign, got a lot of experience in integrating the WeChat back end to Azure, and delivered an Azure Resource Manager template for WeChat-related Azure services. This will help us and our partners to quickly implement an Azure-based WeChat solution. 

## Great team ##

Special thanks to the AnRuiZhuoYue team, the Microsoft China DX Technical Evangelist team, and the Microsoft Audience Evangelism team.
