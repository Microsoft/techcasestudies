---
layout: post
title:  "Migrating the We Carpool platform PHP back end to Azure"
author: "Mostafa Elzoghbi"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-20
categories: [Azure App Service]
color: "blue"
#image: "{{ site.baseurl }}/images/WeCarPool/IntuPower-Profile.PNG" #should be ~350px tall
excerpt:   When We Carpool wanted to migrate its platform PHP back end to Azure, Microsoft helped with the solution. The platform now has continuous delivery from Visual Studio Team Services to Azure.
language: English
verticals: ["Retail, Consumer Products & Services"]
---


Microsoft teamed up with We Carpool to migrate its platform back end to Microsoft Azure. The platform has been set up to have continuous delivery deployment from Visual Studio Team Services into Azure.

In this effort, we implemented the following DevOps practices:

- Continuous integration (CI), the practice of merging all working copies of developers' code with a shared mainline, producing a new build upon code check-in. 
- Hypothesis-driven development (testing in production), the practice of deploying new parts of the code to only a fraction of the total user base in a production environment.

**The core team:**

- Sergey Znutin – CEO, We Carpool
- [Mostafa Elzoghbi](http://www.twitter.com/mostafaelzoghbi) – Senior Technical Evangelist, Microsoft
 
## Customer profile ##

[We Carpool](http://www.wecarpool.com) is a startup car-sharing platform based in California that helps people carpool to public events. People can use We Carpool to share rides and gain access to the faster carpool lanes and also to get paid for driving a carpool of people to events. 

*We Carpool website*

![We Carpool website]({{ site.baseurl }}/images/WeCarPool/WeCarPoolWebsite.PNG "We Carpool website")

 
## Problem statement ##

We Carpool was looking to migrate its PhoneGap app back end to Azure. The back end was built using PHP and the customer needed help to migrate this back end to Azure from 1&1 hosting. Also, the customer wanted to maximize the benefits of a [Microsoft BizSpark](https://bizspark.microsoft.com/) subscription by moving existing components of the product to Azure.

The We Carpool team was looking to take advantage of scalability features in the cloud by moving their back end to scale as needed. The PHP back end connects to a MySQL database. The team was looking to automate builds to deploy product releases to their customers faster using the continuous delivery feature in Azure. 
 
## Solution, steps, and delivery ##

I met with the We Carpool team to discuss the project requirements and integration points with their PHP back end. 

Here is a depiction of the architecture employed: 

![Solution Architecture]({{ site.baseurl }}/images/WeCarPool/Architecture.PNG "Solution Architecture")


### 1. Set up Visual Studio Team Services Git repository

We started by moving the PHP back-end code base to Visual Studio Team Services (VSTS). We set up a new project with Git source control to host the product source code into VSTS.

*Git version control in VSTS*

![GIT version control in VSTS]({{ site.baseurl }}/images/WeCarPool/VSTS-GIT.PNG "GIT version control in VSTS")

Then we checked in the product source base to have VSTS as a new host as a source for continuous delivery and deployment to Azure.

*PHP code base in VSTS*

![We Carpool code base]({{ site.baseurl }}/images/WeCarPool/WCPVSTS.PNG "We Carpool code base")


### 2. Set up Azure App Service

We created a staging deployment slot in the newly created [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/). Then we configured the deployment to pull the source code from VSTS to Azure App Service.

*Continuous delivery in Azure*

![Continuous Delivery in Azure]({{ site.baseurl }}/images/WeCarPool/Azure-CI.PNG "Continuous Delivery in Azure")


Azure App Service offers [MySQL in-app (preview)](https://azure.microsoft.com/en-us/updates/announcing-mysql-in-app-preview-for-web-apps/), which runs a local MySQL instance with your app and shares resources from the App Service plan.

Note that apps using MySQL in-app are not intended for production environments and they will not scale beyond a single instance. Because we are configuring continuous delivery for the staging slot from VSTS, that fits the purpose of this deployment.

*MySQL in-app service*

![MySQL In App Service]({{ site.baseurl }}/images/WeCarPool/MySQLInApp.PNG "MySQL In App Service")


To configure the app to connect to the MySQL instance in App Service, you need to establish a connection first from the app. Therefore, the MySQL process starts in Azure App Service.

For that purpose, we created this page with the following code snippet to start the MySQL process:

~~~~~

$connectstr_dbhost = '';
$connectstr_dbname = '';
$connectstr_dbusername = '';
$connectstr_dbpassword = '';

foreach ($_SERVER as $key => $value) {
    if (strpos($key, "MYSQLCONNSTR_localdb") !== 0) {
        continue;
    }
    
    $connectstr_dbhost = preg_replace("/^.*Data Source=(.+?);.*$/", "\\1", $value);
    $connectstr_dbname = preg_replace("/^.*Database=(.+?);.*$/", "\\1", $value);
    $connectstr_dbusername = preg_replace("/^.*User Id=(.+?);.*$/", "\\1", $value);
    $connectstr_dbpassword = preg_replace("/^.*Password=(.+?)$/", "\\1", $value);
}

$link = mysqli_connect($connectstr_dbhost, $connectstr_dbusername, $connectstr_dbpassword,$connectstr_dbname);

if (!$link) {
    echo "Error: Unable to connect to MySQL." . PHP_EOL;
    echo "Debugging errno: " . mysqli_connect_errno() . PHP_EOL;
    echo "Debugging error: " . mysqli_connect_error() . PHP_EOL;
    exit;
}

echo "Success: A proper connection to MySQL was made! The my_db database is great." . PHP_EOL;
echo "Host information: " . mysqli_get_host_info($link) . PHP_EOL;

mysqli_close($link);


~~~~~


You can validate that the process of MySQL is running by checking the Process Explorer tool in the Azure portal.

![Process Explorer]({{ site.baseurl }}/images/WeCarPool/ProcessExplorerAppService.PNG "Process Explorer")


The App Service provides access to MySQL through the phpMyAdmin web tool.

![Managing MySQL in App Service]({{ site.baseurl }}/images/WeCarPool/MySQLManagementAppService.PNG "Managing MySQL in App Service")


The final step was to change the back-end database configuration file that contains database settings to load up this configuration using environment variables as a best practice.

~~~~~
$connectstr_dbhost = '';
$connectstr_dbname = '';
$connectstr_dbusername = '';
$connectstr_dbpassword = '';

foreach ($_SERVER as $key => $value) {
    if (strpos($key, "MYSQLCONNSTR_localdb") !== 0) {
        continue;
    }
    
    $connectstr_dbhost = preg_replace("/^.*Data Source=(.+?);.*$/", "\\1", $value);
    $connectstr_dbname = preg_replace("/^.*Database=(.+?);.*$/", "\\1", $value);
    $connectstr_dbusername = preg_replace("/^.*User Id=(.+?);.*$/", "\\1", $value);
    $connectstr_dbpassword = preg_replace("/^.*Password=(.+?)$/", "\\1", $value);
}


/** MySQL database username */
define( 'YOURLS_DB_USER', $connectstr_dbusername );

/** MySQL database password */
define( 'YOURLS_DB_PASS', $connectstr_dbpassword);

/** The name of the database for YOURLS */
define( 'YOURLS_DB_NAME', $connectstr_dbname);

/** MySQL hostname : this contains the port number in this format host:port . Port is not 3306 when using this feature*/
define( 'YOURLS_DB_HOST', $connectstr_dbhost ); 
~~~~~


## General lessons ##

We Carpool was extremely excited about the fast turnout of this project by migrating the product code base to Microsoft online services including Visual Studio Team Services and Microsoft Azure.

The development team saw the real business value of continuous delivery of the code into different deployment slots in Microsoft Azure. The team is now able to push code updates into different environments without the need to copy and paste project files. The team was not using source control that allows them to track releases and changes in the product code base.

One of the lessons learned when setting up continuous delivery is to make sure to have the proper permissions on Visual Studio Team Services in addition to Azure.

## Additional resources ##

- [We Carpool website](http://www.wecarpool.com)
- [Microsoft Azure](https://azure.microsoft.com/en-us/?b=17.06)
- [Microsoft BizSpark](https://bizspark.microsoft.com/)
- [Azure App Service](https://azure.microsoft.com/en-us/services/app-service/)
- [MySQL in-app (preview)](https://azure.microsoft.com/en-us/updates/announcing-mysql-in-app-preview-for-web-apps/)


