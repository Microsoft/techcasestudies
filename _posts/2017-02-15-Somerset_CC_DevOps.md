---
layout: post
title: "Migrating an on-premises solution to Azure PaaS services with automated deployment"
author: "Marcus Robinson, Ross Smith, Joni Collinge"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date: 2017-05-12
categories: [DevOps]
color: "blue"
image: "images/somersetcc/feat_roamss.png"
excerpt: "Somerset County Council wanted to migrate their on-premises mapping solution to Azure. We did this by using Azure Database for PostgreSQL, Visual Studio Team Services, Azure App Service, and Docker."
language: [English]
verticals: [Government]
geolocation: [Europe]
#permalink: /<page-title>.html
---

Somerset County Council wanted to migrate their on-premises mapping solution to Azure. We did this by taking advantage of Visual Studio Team Services, Azure App Service, Azure Database for PostgreSQL, and Docker.

### Key technologies used

- [Microsoft Visual Studio Team Services](https://www.visualstudio.com/team-services/)
- [Microsoft Azure App Service](https://azure.microsoft.com/en-us/services/app-service/)
- [Azure Database for PostgreSQL](https://azure.microsoft.com/en-us/services/postgresql/)
- [Docker](https://www.docker.com/)

### Core team

- Ed Loat – Principal Project Manager, Somerset CC
- Ben Sutcliffe – Software Developer, Somerset CC
- Greg Williams – Software Developer, Somerset CC
- Paul Warren – Senior Software Developer, Somerset CC 
- Andy Grigg – Service Manager, ICT Systems, Somerset CC 
- [Marcus Robinson](https://twitter.com/techdiction) – Technical Evangelist, Microsoft UK
- [Ross Smith](https://twitter.com/ross_p_smith) – Technical Evangelist, Microsoft UK
- [Joni Collinge](https://twitter.com/dotjson) – Technical Evangelist, Microsoft UK
- Ben Hummerston – Cloud Solution Architect, Microsoft UK
- Gina Dragulin – Director, Audience Evangelism, Microsoft UK

## Customer profile

Somerset County Council (Somerset CC) is a local government organization in the UK. Their IT function was outsourced for nine years until December 2016; as such, they are now looking to revitalize their existing IT estate.

While most of the IT management was outsourced, the development team at Somerset CC built a web application called Rights-of-way and Open space Area Management (ROAM) for managing rights of way and open spaces within the county, as well as providing some CRM-type functionality for logging issues with “furniture” on public byways (such as stiles and bridges). Issues can be reported by the public, and wardens can take ownership of issues and take action (visit site, phone, email person who reported it) and mark issues as resolved. 

The system relies heavily on mapping data that is a combination of image-based mapping tiles and vectors that are displayed on top of the tiles. The current setup is virtual machine-based and uses open source mapping software called [GeoServer](http://geoserver.org/), along with PostGIS for the back-end database. The Somerset CC development team is very keen to use platform as a service (PaaS) wherever possible to improve the performance of ROAM, and is very welcome to new ideas and ways of working. 

ROAM is known publicly as [Explore Somerset](http://roam.somerset.gov.uk/roam).

<br/>

<img alt="Explore Somerset" src="{{ site.baseurl }}/images/somersetcc/roamss.png" width="700"/> 

<br/>

## Problem statement

Because most of IT was outsourced, Somerset CC had very little control over their services up until now. As such, the current ROAM solution is sitting on 32-bit Linux servers with limited capacity. Also Somerset CC’s IT skillset is predominantly around Windows, which leads to challenges when administrating Linux servers.

Somerset CC also had interest from other councils throughout the UK to make ROAM available for them to use in their counties as well. As ROAM is currently virtual machine-based, it isn’t designed for either multi-tenant access or scalability and easy deployment if multiple instances are required.

## Solution, steps, and delivery

- [Architecture overview](#architecture-overview)
- [Value stream mapping](#value-stream-mapping)
- [Migrating the infrastructure to Azure](#migrating-the-infrastructure-to-azure)
- [Migrating PostgreSQL to Azure Database for PostgreSQL](#migrating-postgresql-to-azure-database-for-postgresql)
- [Migrating GeoServer to IaaS](#migrating-geoserver-to-iaas)
- [Migrating GeoServer to Web App on Linux using an Azure Blob storage plug-in](#migrating-geoserver-to-web-app-on-linux-using-an-azure-blob-storage-plug-in)

### Architecture overview

The ROAM architecture consists of three main components, all running on-premises on VMware virtual machines:

- **Public facing web site**: ASP.NET running on Internet Information Services (IIS) on Windows Server.

- **Geographic Information System (GIS)**: [GeoServer](http://geoserver.org/) is an open-source GIS that allows users to view and edit geospatial data; designed with interoperability in mind, it adheres to the [OGC](http://www.opengeospatial.org) standards.

- **Data layer**: [PostgreSQL](https://www.postgresql.org/) with a number of databases to hold business and mapping vector data using PostGIS extensions, running on 32-bit SUSE Linux.

<img alt="Architecture overview" src="{{ site.baseurl }}/images/somersetcc/arch.png" />

<br/>

### Value stream mapping

Prior to the hackfest, we visited Somerset CC to carry out a value stream mapping (VSM) of their existing processes and discuss the focus for the hackfest.

Value stream mapping enables value to be tracked throughout the software development process. The initial stage of the value stream mapping exercise involved discussing the existing development process. This resulted in a list of steps, grouped into stages.

<br/>

<img alt="Value stream mapping steps" src="{{ site.baseurl }}/images/somersetcc/steps.jpg" width="700"/>

<br/>

We then mapped out these stages detailing the people involved, the time it would take for the stage to complete, and any lead time needed into the stage.

<br/>

<img alt="Value stream mapping stages" src="{{ site.baseurl }}/images/somersetcc/vsm_in_progress.jpg" width="700"/>

<br/>

After this process was completed, we identified areas of inefficiency, including waste, manual processes, and heroics.

<br/>

<img alt="Value stream mapping inefficiencies" src="{{ site.baseurl }}/images/somersetcc/vsmchart.jpg" width="700"  />

<br/>

The major areas identified for potential improvement included:

-   **The use of Microsoft Visual SourceSafe (VSS) as their source control** and its lack of branching. Using VSS means that developers experience delays waiting to check in code and that incomplete code often gets shipped.

-   **Multiple steps that rely on the principal program manager, Ed**, who is the only person that manages backlog, signs off on releases, and negotiates with the business about release scheduling. If Ed is away, no releases can happen.

-   **A three-week lead time between signing off on a release and the actual release**. This delay includes waiting on additional sign-offs from elsewhere in the business and release scheduling because the system has to be taken offline for an hour or more to carry out a release. In addition, backups need to be scheduled with the infrastructure maintainers, and Ed and a developer must synchronize to execute the actual release. 

-   **Significant differences between test and production environments**, which can lead to unexpected errors during a release.

-   **A release process that involves following a list of manual steps**, which means it is a lengthy process that depends on a single person and steps can be forgotten.

In addition to the items identified during the VSM, we also know that Somerset CC was keen to migrate to PaaS services on Azure to improve agility and remove reliance on their on-premises infrastructure.

With this in mind, we agreed to look at the following areas during the hackfest:

- Migrating the infrastructure to Azure and implementing [Infrastructure as Code](https://www.visualstudio.com/learn/what-is-infrastructure-as-code/) to provide the ability to create new environments on demand.
- Migrating the source control to Visual Studio Team Services.
- Creating automated build and release pipelines to enable continuous integration (CI) and continuous delivery (CD) by using Visual Studio Team Services.  

### Migrating the infrastructure to Azure

When planning the migration to Azure, we considered three potential solutions:

1. Lift and shift the three virtual machines.

    - Migrate the three virtual machines as they are to Azure infrastructure as a service (IaaS).

    <img alt="Option 1" src="{{ site.baseurl }}/images/somersetcc/option1.png"  />

    <br/>

2. Introduce platform as a service (PaaS).

    - Migrate the front-end website to the Web Apps feature of Azure App Service.

    - Migrate PostgreSQL to Azure Database for PostgreSQL (a managed database service for app development and deployment).

    - Migrate GeoServer to IaaS.

    <img alt="Option 2" src="{{ site.baseurl }}/images/somersetcc/option2.png" />

    <br/>

3. Go 100% PaaS.

    - With this option, we would also migrate GeoServer to Web App on Linux.

    <img alt="Option 3" src="{{ site.baseurl }}/images/somersetcc/option3.png" />

    <br/>

Option 1 does not offer much of a challenge because multiple servers still have to be managed, and there are no improvements with regards to scalability. Option 3 goes 100% PaaS; however, we could foresee a number of issues with regards to storage trying to deploy GeoServer to Web App on Linux, a major requirement being the development of a Blob storage plug-in as a prerequisite. Given the limited time allocated for the hackfest, we decided to go with Option 2, leaving GeoServer on IaaS, but would look at Option 3 should time allow and the Blob storage plug-in be ready. For more information about Option 3, see the section [Migrating GeoServer to Web App on Linux using an Azure Blob storage plug-in](#migrating-geoserver-to-web-app-on-linux-using-an-azure-blob-storage-plug-in) later in this article.


#### Migrate the source control to Visual Studio Team Services

One of the standout areas during the value stream mapping was the application lifecycle management. Visual SourceSafe (VSS) was still being used and developers would often check in unfinished code so that their colleagues could work on the same file. There was no branching strategy, and it soon became apparent that not all the dependencies were in source control. 

The customer already had Office 365 so we knew that the identity question had been answered, so we tracked down the Azure Admin and asked them to create an instance of Visual Studio Team Services, which was then linked to their Azure Active Directory. This provided the development team with five free licenses and single sign-on. We then asked the Admin to invite the Microsoft Technical Evangelists as partners via the Azure Active Directory B2B mechanism, which allowed us to use Visual Studio Team Services under our MSDN licenses at no cost to the customer. For more information about how to do this, see [Learn all about the Azure AD B2B Collaboration Preview](https://blogs.technet.microsoft.com/enterprisemobility/2015/09/15/learn-all-about-the-azure-ad-b2b-collaboration-preview/).

We created a team project for the department and then created a team with an area path called `ROAM`, which allowed the developers to move other projects into the team project at a later date. This will simplify deploying to Azure in the future rather than having multiple team projects for each project. For more information about this model, see [Many Git Repositories, but one Team Project to rule them all](https://blogs.msdn.microsoft.com/willy-peter_schaub/2014/11/19/many-git-repositories-but-one-team-project-to-rule-them-all/). 

The final step was to get the source control and add it into Team Foundation Version Control via Visual Studio. We did debate for 30 minutes whether to use Git; however, we settled on Team Foundation Version Control because the Somerset CC staff were familiar with Visual SourceSafe, so a centralized source control was a natural step for them rather than teaching them Git. A good reference article for making the decision is [Choosing the right version control for your project](https://www.visualstudio.com/en-gb/docs/tfvc/comparison-git-tfvc).

The .NET projects had to be rebound to a new source control provider that was straightforward enough from within Visual Studio and resulted in changing the `sln` and `csproj` as follows.

<br/>

<img alt="Binding source control" src="{{ site.baseurl }}/images/somersetcc/BindingSourceControl.png" width="700" />

<br/>

We also had to add `.tfignore` files to ignore the Packages folder because these projects were created before NuGet was around, so some of the smarts that are added to `csproj` were missing. The first test was to try and build the source code on a new machine, which flushed out dependencies very quickly; there were references to libraries on network shares and to libraries that were manually copied. While this was being fixed, we created a new build server. 

#### Use a private build agent

The decision to use a private build agent was made due to the experience of previous hackfests where we relied on hosted agents and lost a lot of time waiting for builds. We did try to use a quick start template to save us time, installing the build agent from [Visual Studio and Visual Studio Team Services Build Agent VM](https://azure.microsoft.com/en-gb/resources/templates/visual-studio-vstsbuildagent-vm/). However, this failed on every different SKU of Visual Studio, so we used an image from the gallery instead. This image already had the Visual Studio Community edition and the Azure SDK installed, which saved us a lot of time. To create one directly, see [Visual Studio Community 2015 Update 3 with Azure SDK 2.9 on Windows Server 2012 R2](https://portal.azure.com/#create/Microsoft.VisualStudioCommunity2015Update3withAzureSDK29onWindowsServer2012R2-ARM). 

We asked one of the Somerset CC developers to create a [personal access token](https://www.visualstudio.com/en-us/docs/setup-admin/team-services/use-personal-access-tokens-to-authenticate) from their account so that we could set up the build server and leave them with the token management. Installing the build agent was straightforward, but if you have never done this, see the guidance at [Deploy an agent on Windows](https://www.visualstudio.com/en-us/docs/build/actions/agents/v2-windows). Creating our first build pipeline showed us that we required [Devart dotConnect for PostgreSQL](https://www.devart.com/dotconnect/postgresql) installed on the build server, which meant that we would not be able to go back to the hosted build agents.

#### Use Package Management to host class libraries

While building the solution, it quickly became apparent that the development workflow was to output libraries onto shared drives so that other solutions could make use of them. We decided to use [Package Management](https://marketplace.visualstudio.com/items?itemName=ms.feed) as a more manageable solution to host these class libraries so that we could have an end-to-end dependency solution. Although you get five free users, we still had to purchase one license for it to install, which was tiresome as we had to go back to the Azure Admin and get permission to do this. A simple build definition to create your own packages is shown in the following diagram; it consists of `Restore -> Compile -> Package -> Publish`. These tasks were written by colleague [Lawrence Gripper](https://twitter.com/lawrencegripper).

<br/>

<img alt="NuGet Packager" src="{{ site.baseurl }}/images/somersetcc/NugetPackager.png" width="700" />

<br/> 

A few things to call out on the previous screenshot:

1. If you change the Package Folder of the NuGet Packager task to be `$(Build.ArtifactStagingDirectory)`, it will automatically put the .nupkg file in a folder that will get wiped on every build. Otherwise, this can get problematic because they build up in the Sources Directory, meaning sometimes an old file can get published.

2. We were referencing a pre-release version of an Entity Framework library, which means that we should mark our package as alpha. This was achieved by updating the version field as shown in the previous screenshot. 

    <br/>

    <img alt="Pre-release dependency" src="{{ site.baseurl }}/images/somersetcc/PreReleaseDependency.PNG" width="700" />

    <br/> 

The publisher task was simply configured with the URL supplied when we created Package Management in Visual Studio Team Services. We repeated this for all of the dependencies, and could then remove all references to network shares. The final private feed looked like this.

<br/>

<img alt="Package Management" src="{{ site.baseurl }}/images/somersetcc/PackageManager.PNG" width="700" />

<br/> 

#### Create the main build

After we had packages, we could create the main build that would use the public and new private Package Management as its source as shown in the following image. A better solution would have been achieved by using a nuget.config file, but the customer did not have one already set up. Using the config file would mean that all developers would automatically read from the private feed from within Visual Studio.

<br/>

<img alt="ROAM build" src="{{ site.baseurl }}/images/somersetcc/RoamBuild.PNG" width="700" />

<br/>

While this build was non-trivial, a few small details to note are as follows:

1. I always use the `PackageLocation` parameter within the Visual Studio/MSBuild task to change the output to `ArtifactStagingDirectory` because that saves an extra copy on the build server.

    <br/>

    <img alt="PackageLocation parameter" src="{{ site.baseurl }}/images/somersetcc/PackageLocation.PNG" width="700" />

    <br/>

2. Three artifacts are to be published: the website itself, the [Azure Resource Manager (ARM) templates](https://azure.microsoft.com/en-us/resources/templates/) that define the infrastructure, and a file that is used to configure URL rewriting on the App Service. This was to ensure that when we make JavaScript calls to GeoServer, they appear to come from the same domain and minimize any CORS (cross-origin resource sharing) problems.

3. We separated out two ancillary services into their own build and release pipelines because they were not deployed that often and rarely changed.

4. The URLs of these ancillary services needed to be dealt with before, and I usually use parameters.xml files to achieve a tokenized file and then replace the token at deploy time. However, we wanted to utilize deployment slots; therefore, we decided to place the config file in the App Service itself by using ARM templates.

#### Use Azure App Service for ROAM web app builds

The ROAM web app was dependent on two other services for it to be able to run, and these services did not change as often as the main website. I decided that each should have its own separate build so that we only deployed the areas that changed. All were source artifacts to the ROAM release definition, and WebDeploy would only deploy artifacts that had changed, so it was safe to do this. I set up the dev and test environments to use the same App Service and use slots. This way we could test the slot behavior before we went to production. For the sake of the hackfest, we then mirrored this with UAT and production being a shared App Service too. We would deploy into the staging slot and when we were happy, we could swap the slots. This meant that we had to use slot configurations so that the staging ROAM web app was configured to depend on the staging services. We created variables in the ARM template to represent the staging and production URIs that we would depend on.

<script src="https://gist.github.com/ross-p-smith/c927cd249b9751937f9c363933ef22d0.js"></script>

<br/>

When we swapped, the app would then use different services. Designing the ARM templates was achieved through Visual Studio because I wanted to show the Somerset CC staff how to create their own templates from scratch. Unfortunately, you cannot add app settings configuration into the template via the Visual Studio JSON Outline tool; therefore, these were added manually in the Azure portal and then reverse engineered by using [Azure Resource Explorer](http://resources.azure.com). You can see that four parameters were passed into the ARM template so that at deploy time, we could change which supporting services the main ROAM web app communicated with through dev, test, UAT, and production. It was decided to use this mechanism rather than using `MSDeploy` parameters so that we could better use staging slots.

In the following screenshot, section 1 lists all of the app settings required for the production slot. Section 2 tells the App Service that two of those app settings should remain in the production slot when it is swapped. Section 3 defines the variables for the staging slot, which means that we will not override configuration during a swap.

<br/>

<img alt="ARM slots" src="{{ site.baseurl }}/images/somersetcc/ARMSlots.PNG" width="700"/>

<br/>

#### Add resource group to release pipeline

With repeatable Infrastructure as Code defined in an ARM template, we could now spin up a resource group in Azure with all of the necessary app services and have configuration pre-loaded into these app services on how they would communicate with each other. Now we just had to add this into a release pipeline. During the deployment of the ARM template, we need to override settings in the app settings with parameters that were stored for each environment. Here you can see that `GeoBackdropServer`, `GeoVectorServer`, `SecurityEntitiesUri`, and `SysAdminEntitiesUri` are stored for the test environment.

<br/>

<img alt="Test environment variables" src="{{ site.baseurl }}/images/somersetcc/TestEnvironmentVariables.PNG" width="700" />

<br/>

The release pipeline for the web tier is typical; we had a step in the dev environment that would delete the entire resource group, but due to time constraints in the hackfest, we disabled this. We then created or updated the resources and deployed three app services by using the out-of-the-box Azure App Service Deploy. Always ensure that you select **Take App Offline** because this will prevent the process from being locked when you deploy. We were deploying to the staging slot on every check-in to the source repository. The test environment could be thought of as a nightly build; in this stage we did a slot swap from staging to production. As you can see, we had great success with the new preview task.

<br/>

<img alt="ROAM release" src="{{ site.baseurl }}/images/somersetcc/RoamRelease.PNG" width="700"/>

<br/>

To provide governance and ensure that Ed could control releases into UAT, we set him up as an Approver before the release moved into UAT. This was called out specifically in the value stream mapping of an area that they required. 

<br/>

<img alt="Approvers" src="{{ site.baseurl }}/images/somersetcc/Approvers.PNG" width="700"/>

<br/>

### Migrating PostgreSQL to Azure Database for PostgreSQL

During the hackfest, we were fortunate to have access to the private preview of the Azure Database for PostgreSQL. 

#### Automate server deployment

The first step was to automate the deployment of a server by using an ARM template. The ARM template was created by using [Azure Resource Explorer](http://resources.azure.com) to identify the required JSON. Because it was a hackfest, the data has no personal information, so for ease of use we opened up the service to all IP addresses.

<script src="https://gist.github.com/marrobi/bb8a1c08b4154992f1838dc61e76eab4.js"></script>

<br/>

#### Connect to Azure Database for PostgreSQL

The next step after the service was deployed was to connect to Azure Database for PostgreSQL. We wanted to check connectivity and be able to verify the result of any operations through a graphical interface. Somerset CC uses tools such as [pgAdmin](https://www.pgadmin.org/) to do this. However, due to corporate firewall restrictions, we could not connect out to the service.

To work around this, we deployed a Docker container running pgAdmin 4 on an Azure virtual machine (actually for hackfest purposes, on GeoServer), which enabled us to connect to Azure Database for PostgreSQL.

`docker run --name my-pgadmin4  -p 5050:5050   -d fenglc/pgadmin4`

<br/>

#### Create and import the database

The next step was to create and import the database. We had data exports in two separate formats, SQL files and dumps created by `pg_dump`.

Because we wanted to be able to repeat this procedure, we decided to script the data import of the SQL files. Because we had an existing Visual Studio Team Services build agent configured running Windows, we decided to do this by using PowerShell. Another reason for doing this was so that future database updates could be automated as part of the release pipeline. In hindsight, it may have been easier to stick with `psql` on the command line.

Initially we attempted this by using the [PostgreSQL ODBC driver](https://odbc.postgresql.org). When running `CREATE DATABASE` commands, we encountered a timeout. After trying various workarounds, we contacted the product team who suggested an [alternative ODBC driver](http://www.npgsql.org) ([download](https://github.com/npgsql/Npgsql/releases)). This driver allowed us to specify connection timeout settings within the connection string. We created a simple script that resolved the majority of issues with some inspiration from [this PowerShell module](https://github.com/palpha/Simple-PostgreSQL-module-for-PowerShell). 

<script src="https://gist.github.com/marrobi/a28256e43edc2f137bc2ff95a68afe13.js"></script>

<br/>

This can be used with a command:

`.\execute_sql.ps1 -ServerHostName "servername.database.windows.net" -User "postgres@servername" -DBName "roam" -Password "XXX" -SQLCmd "XXX"`

<br/>

Or it can be used with a SQL file:

`.\execute_sql.ps1 -ServerHostName "servername.database.windows.net" -User "postgres@servername" -DBName "roam" -Password "XXX" -SQLFilePath "\pathto\mysqlfile.sql"`

<br/>

The extremely large imports were still troublesome. Because these would only be one-off events, we resorted to `psql` for these. However, the script would still be useful within the release pipeline.

For the files created by using `pb_dump`, we used `pg_restore`. One import took a number of hours and had to be left overnight to complete.

#### Enable the PostGIS extensions

The final step was to enable the [PostGIS extensions](http://www.postgis.net). Because a number of the databases are used to store location and mapping data, the PostGIS extensions are required. Azure Database for PostgreSQL supports these extensions, so it was simply a case of enabling the extensions on the databases after creation and prior to data import by using the following command.

`CREATE EXTENSION postgis`

<br/>

### Migrating GeoServer to IaaS

GeoServer can be run on Windows or Linux operating systems; however, when researching the product it soon became apparent that the majority of documentation and articles online targeted users running Linux. Further investigation of ways to get up and running with GeoServer quickly led to the discovery of several GeoServer images on Docker Hub. After testing a number of GeoServer images, the [`winsent/geoserver` image](https://hub.docker.com/r/winsent/geoserver/) appeared to be well-maintained and functioned well.

Because Somerset CC does not have an automated method for installing GeoServer, it made sense to use an existing image on Docker Hub for the hackfest.

#### Create ARM template with Docker extension

To enable the automated provisioning of the GeoServer image onto Azure, we created an ARM template utilizing the [Azure Docker extension](https://github.com/Azure/azure-docker-extension). For additional documentation and examples, see [Create a Docker environment in Azure using the Docker VM extension](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-linux-dockerextension).

The configured extension looked as follows.

<script src="https://gist.github.com/marrobi/23b11e7ac7eefa00b1a5a6e838eccdcf.js"></script>

<br/>

#### Configure GeoServer

One of the areas identified during the VSM was the requirement to have consistent dev, test, and production environments. GeoServer's configuration resides in a data directory within the container. To enable continuous delivery, this configuration will need transferring between dev, test, and production GeoServer instances. This is not easily done when the configuration is stored within a container.

To work around this, we decided to store the GeoServer configuration on an Azure Files share and mount this into the container as a Docker volume.

We created three Azure Files shares:

- dev-data-dir
- test-data-dir
- prod-data-dir

A script was executed by using a `CustomScript` extension to mount the shares.

<script src="https://gist.github.com/marrobi/f4beff75476543d038e318d01403b923.js"></script>

<br/>

<script src="https://gist.github.com/marrobi/c9cf92ebb718f53f4f64888d7270be20.js"></script>

<br/>

The Docker extension was updated to mount the data directory into the default data directory location within the GeoServer container.

<script src="https://gist.github.com/marrobi/e5152e05829150f0b0b3c2532b62c6ba.js"></script>

<br/>

#### Share GeoServer raster data

Somerset CC's GeoServer implementation access vector data was stored within PostgreSQL, and raster data was stored on locally accessible storage. 

The raster data is made up of many gigabytes of GeoTIFF files. This data does not change between dev, test, and production environments. If data is added to the share, it is not available until it is added to the GeoServer configuration for a specific environment. For this reason it makes sense to share the data between all GeoServer instances. As with the configuration data, we mounted an Azure Files share by using a script and presented that to GeoServer as a volume.

#### Configure GeoWebCache

GeoServer utilizes a caching component called [GeoWebCache (GWC)](http://geowebcache.org/), which can be run as an integrated component of GeoServer or independently. Although an Azure Blob storage extension for GeoWebCache was under development as part of the hackfest, we needed to use GeoWebCache prior to the development being completed. GeoWebCache requires a large amount of local storage, so we provisioned a new data disk to the virtual machine and mounted this as a volume into the container.

<script src="https://gist.github.com/marrobi/a3c25fa8b881945cbafdecde65517436.js"></script>

<br/>

The disk needs initializing, partitioning, and formatting prior to being presented into the container. We needed an automated way to do this, and some searching returned the blog post [Format Data Disks with Azure VM Custom Script Extension for Linux](http://blog.fullscale180.com/format-data-disks-with-azure-vm-custom-script-extension-for-linux/).

We saved the following script.

<script src="https://gist.github.com/trentmswanson/9c22bb71182e982bd36f.js"></script>

<br/>

We then added it to our `CustomScript` extension.

<script src="https://gist.github.com/marrobi/4ebeb7814dd8f13789d98aeddfae7eec.js"></script>

<br/>

The location of the GeoWebCache directory is specified within an XML file at `/opt/geoserver/webapps/geoserver/WEB-INF/web.xml`. We could have mounted *another* Azure Files share containing this xml file, but the decision was made to create a custom Docker image for the following reasons:

- Somerset CC wanted full control over the contents of the image and any changes that were deployed.
- Potential security concerns existed around pulling an image directly from a public repository.
- After the Azure Blob storage extension is complete, this will need to be added to the image.

The following describes the steps we took to create a custom Docker image:

1. We created a custom Dockerfile:

    <script src="https://gist.github.com/marrobi/697710c74e3ffc943b07693ccd625ec2.js"></script>

    <br/>

2. We created an Azure Container Registry to store the image and we built the image with a tag relating to the repository:

    `docker build --tag roamregistry.azurecr.io/geoserver:dev`

    <br/>

3. We ensured that a container that used the default configuration of GeoServer ran successfully:

    `docker run -d -p 8080:8080 --name geoserver roamregistry.azurecr.io/geoserver:dev`

    <br/>

4. We logged on to the registry using the following:

    `docker login roamregistry.azurecr.io`

    <br/>

5. We pushed using the following:

    `docker push roamregistry.azurecr.io/geoserver:dev`

    <br/>

6. The image name was updated in the Docker extension within the ARM template. 

7. To enable Docker on the provisioned virtual machine to access the registry, we also added registry credentials to the extension:

    <script src="https://gist.github.com/marrobi/2f84928824c7c4d0c545e3c1bd04efc2.js"></script>

    <br/>

8. The finalized ARM template was deployed to Azure from Visual Studio and tested. 

The next step was to automate the deployment of the GeoServer through multiple environments.

#### Create release pipeline

Deployment into the first environment, dev, was relatively straightforward and included copying the scripts into Blob storage and deploying the template.

<img alt="GeoServer dev environment" src="{{ site.baseurl }}/images/somersetcc/geoserverdev.png" />

Prior to deploying to the test environment, two things needed to happen:

1. The GeoServer dev configuration needed to replace the existing test configuration.
2. Database connection strings needed to be updated to point to a different PostgreSQL server.

<br/>

We configured the following steps to deploy GeoServer into the test environment:

1. Map `DevDataDir` and `TestDataDir`.

    ```
    # map test share
    net use T: \\$(TestDataDirPath) /u:$(TestDataDirUsername) $(TestDataDirPassword) 

    # map dev share 
    net use X: \\$(DevDataDirPath) /u:$(DevDataDirUsername) $(DevDataDirPassword)
    ```


2. Remove the existing contents of the test configuration share. Because GeoServer does not actively access the data directory, only during startup or if configuration is carried out, we can work on it while the service is running.

    ```
    # empty test share
    Get-ChildItem T:\ -Recurse | Remove-Item -Force   -Recurse
    ```

3. Copy the dev configuration into the test share.

    ```
    # copy from dev share to test share
    Copy-Item -Path X:\* -Destination T:\ -Recurse -Force
    ```

    Some scripts that will synchronize two directories may be more efficient. This process takes around 20 minutes, which is far from ideal. But for the purposes of the hackfest, a clean out and copy works and does not lead to any additional downtime.

    During this stage it became apparent that GeoServer stores its log file in the data directory. Because the file is in use, it cannot be removed or moved. To resolve this, we specified the path to the log file as an environment variable within the Dockerfile and created a new image.

    ```
    ENV GEOSERVER_LOG_LOCATION=/var/log/geoserver.log
    ```

4. Update database connections. Updating the database connections is a little more complex. GeoServer stores the PostgreSQL database connection details within an XML file associated with each data store.

   The elements look like the following.

    ```
    <entry key="database">roam-dev</entry>
    <entry key="port">5432</entry>
    <entry key="passwd">PASSWORD</entry>
    <entry key="min connections">1</entry>
    <entry key="dbtype">postgis</entry>
    <entry key="host">192.192.192.192</entry>
    ```

    We created the following PowerShell script to carry out the update.

    <script src="https://gist.github.com/marrobi/806f464c47b81bddc297442d5d1d8e72.js"></script>
    

5. Reload the GeoServer configuration by using REST API. GeoServer has a REST API; by using this, the configuration can be reloaded.

6. Unmount the shares. 

Each environment after the dev environment will follow this format.

<br/>

#### Make Azure Files mounts persistent

It became apparent that the mounted file shares were lost after a reboot of the Linux host virtual machine, so they needed to be made persistent. To resolve this, we added the file shares to `/etc/fstab` within the script as follows.

```
if [[ $(grep -q "/media/geoserverdatadir" "/etc/fstab") ]]
then
    echo "${1} /media/geoserverdatadir cifs vers=3.0,username=${2},password=${3},dir_mode=0777,file_mode=0777" >> /etc/fstab
else
    echo "/media/geoserverdatadir entry in fstab exists."
fi
```

<br/>

### Migrating GeoServer to Web App on Linux using an Azure Blob storage plug-in

Both GeoServer and GeoWebCache are actively maintained Java and [Maven](https://maven.apache.org/)-based open source projects. To write a new caching layer that supported Azure Blob storage, I needed to contribute to both projects.

By default, GeoWebCache stores the data on a local disk. However, for improved scalability, availability, and resiliency, options exist to use an external storage plug-in. There are currently several external storage providers, including Amazon Simple Storage Service (Amazon S3), but no Azure support. As we ultimately want to run GeoServer in Web App on Linux, writing the data to disk was not an option. We therefore needed a remote storage provider, a good opportunity to support the GeoWebCache community by contributing our own plug-in extension for Azure Blob storage.

#### Create the GeoWebCache plug-in

Because GeoServer has a dependency on GeoWebCache, I began by writing the [GeoWebCache plug-in](https://github.com/jjcollinge/geowebcache), which is currently hosted on a forked GitHub repo.

GeoWebCache exposed a number of interfaces that needed to be implemented in our plug-in:

- `BlobStoreConfig` – This is a very small interface, but assumes that you will extend it to support all the configuration settings required by your provided blob store. For the sake of Azure Blob storage, I kept this very simple and simply stored the following properties.

    ```
    blobContainer // Azure Blob Storage container name
    prefix // Blob path prefix
    azureAccountName  // Azure Blob Storage account name
    azureAccountKey // Azure Blob Storage primary key
    ```

- `XMLConfigurationProvider` – This interface is required because GeoServer is going to gather the user's blob settings from an HTML form and then use the implementation of this interface to create a `BlobStoreConfig`.

- `BlobStore` – This is the main interface that provides CRUD (create, read, update, delete) methods to your desired blob storage, in this case Azure Blob storage. The `BlobStoreConfig` will be injected into this class so that it is configured as requested by the user in GeoServer. GeoWebCache stores both blobs and metadata and exposes more complex CRUD methods such as the following.

    ```
    public boolean delete(final TileRange tileRange) {...} // delete all tiles in a given range
    public boolean deleteByGridsetId(final String layerName, final String gridSetId) {...} // delete all tiles within a given gridset
    ```

<br/>

I decided to isolate as much of the `com.microsoft.azure.storage.blob` dependent code as possible into its own class called `AzureOps`.
This class abstracted the *actual* Azure-specific CRUD operations for blobs, properties, and metadata.

**Test the plug-in**

I have provided a basic set of [integration tests (GeoWebCache and Azure Blob storage)](https://github.com/jjcollinge/geowebcache/tree/jjcollinge/gwc-azure-blob/geowebcache/azurestorage/src/test/java/org/geowebcache/azure)  that cover *most* methods.

**Build the plug-in**

GeoWebCache uses Maven as its project management tool. The structure of the project is as follows.

```
geowebcache/
    geowebcache/
        ...
        pom.xml // Parent project POM
        azurestorage/
            ...
            pom.xml // Module POM
            src/
```

<br/>

I created a simple build definition on Visual Studio Team Services to build and package the project.

<br/>

<img alt="GeoWebCache build 1" src="{{ site.baseurl }}/images/somersetcc/gwc_build_1.PNG" />

<br/>

The Maven build task was configured to point to the project pom.xml with the default X64 Java version set.
I added the `-DskipTests` option to speed up development, because tests usually do run on the build.

<br/>

<img alt="GeoWebCache build 2" src="{{ site.baseurl }}/images/somersetcc/gwc_build_2.PNG" />

<br/>

After GeoWebCache had successfully been built, I copied the `gwc-azure-blob-1.11-SNAPSHOT.jar` file to the `$(Build.ArtifactStagingDirectory)`.

#### Add a community module to GeoServer

An additional community module needed to be added to GeoServer so that it could load the correct blob store config file to use with GeoWebCache. The forked version of GeoServer is currently hosted on [GitHub](https://github.com/jjcollinge/geowebcache).

The additions included a `gwc-azure` folder, an associated pom.xml, and the required HTML and Java source files. The community, src, app, and release pom.xml files then had to be updated to include the correct `gwc-azure` artifacts along with their dependencies.

**Test the module**

I added some basic [automated UI tests and unit tests](https://github.com/jjcollinge/geoserver/tree/jjcollinge/gwc-azure/src/community/gwc-azure/src/test/java/org/geoserver/gwc/web/blob) on GitHub.

**Build the module**

My forked GeoServer requires that the build artifacts from the forked GeoWebCache build be available in an in-scope Maven repository.
When building locally, I installed the output of the GeoWebCache build in my local Maven repository and then built GeoServer in Bash on Windows.

However, this initially threw the exception `error=12, Cannot allocate memory`. After discussing this issue with one of the WSL Program Managers, we realized this was an issue with Bash on Windows (Anniversary Update) and that a later version of Windows had fixed this issue.

When moving to a hosted build, due to time limitations, I just decided to extend the existing build definition and install the JAR file and WAR files into the local Maven repository of the hosted Linux build agent. Ideally these would be split into two distinct build definitions.

<br/>

<img alt="GeoServer build 1" src="{{ site.baseurl }}/images/somersetcc/gs_build_1.PNG" />

<br/>

The **Install GeoWebCache in local Maven repository** task simply uses the MVN command-line tool to copy the JAR file from the `$(Build.ArtifactStagingDirectory)` and put them in the local `~/.pm2` folder with some additional metadata.

<br/>

<img alt="GeoServer build 2" src="{{ site.baseurl }}/images/somersetcc/gs_build_2.PNG" />

<br/>

To build GeoServer with the Azure extension and with the correct config file, I needed to add a few extra options.

<br/>

<img alt="GeoServer build 3" src="{{ site.baseurl }}/images/somersetcc/gs_build_3.PNG" />

<br/>

After everything was built, I zipped up all the necessary files and published them to a Visual Studio Team Services artifact server.

I also wanted to make the artifacts available publicly so that I could easily download them onto the GeoServer virtual machine and install the extensions at runtime. However, as the hosted Linux build agent doesn't support the PowerShell-based AzureCopy task by default, I had to create a release definition on Windows with a single task to copy the artifacts from the Visual Studio Team Services artifact server to Azure Blob storage.

To make this simpler in the future, I also wrote a simple cross-platform [AzureCopy Visual Studio Team Services task in Typescript](https://github.com/jjcollinge/xplat-copytoazure-task). I haven't published the task to Marketplace, but it is available on GitHub.

**Deploy the module**

Now that I have access to the artifacts of both the GeoServer extensions and GeoWebCache, I needed to test that they were functioning correctly. I therefore needed to spin up a new GeoServer; the way we had been doing this up until now was to use the latest `winsent/geoserver` Docker image that installed Java 7.

However, as the Visual Studio Team Services hosted build agent was targeting Java 8, the GeoServer extension and GeoWebCache artifacts required Java 8. At this point, we considered forking the Dockerfile to add Java 8 support; luckily, on inspection, it turned out that the maintainer had already published a newer version 2.10.1 with Java 8 support but hadn't tagged it latest. After we targeted this specific version `winsent/geoserver:2.10.1`, I could install the additional JAR files and see `AzureBlobStore` functioning in GeoServer.

**Add the module into the Docker image**

To include the new extension into the GeoServer installation, the JAR files created by the build process needed to be copied into the `/opt/geoserver/webapps/geoserver/WEB-INF/lib/` folder. We did this my adding the following line to the Dockerfile.

`COPY ./azure_extension/ /opt/geoserver/webapps/geoserver/WEB-INF/lib/`

<br/>

#### Configure GeoServer on Web App on Linux

It proved relatively easy to get GeoServer up and running on the Web App on Linux service. As the service is still in preview and there is limited documentation around ARM templates, we used the Azure portal to configure the service.

The Docker container needed to be set to our custom GeoServer container as follows.

<br/>

<img alt="App Service Docker container" src="{{ site.baseurl }}/images/somersetcc/appservicedockercontainer.png" />

<br/>

Because GeoServer is served on port 8080, we needed to redirect port 80 on the web app to 8080 on the container image. To do this we added the application setting `PORT` as follows.

<br/>

<img alt="App Service port" src="{{ site.baseurl }}/images/somersetcc/appserviceport.png" />

<br/>

Our custom GeoServer (with the Azure plug-in) can be seen running on Azure App Service for Linux here (note the *.azurewebsites.net* URL denoting Azure App Service):

<br/>

<img alt="GeoServer on App Service" src="{{ site.baseurl }}/images/somersetcc/geoserveronappservice.png" />

<br/>

In the IaaS model, we are presenting the GeoServer configuration and raster data as Docker volumes. We were also using local storage for the GeoWebCache, but with the Blob storage plug-in this is no longer needed. Because we cannot mount custom volumes in Web App on Linux, we need to find ways to handle the configuration and raster data. 

**Manage GeoServer configuration**

When a web app scales, all server instances need to be accessing the same configuration. We discussed potential solutions for managing the GeoServer configuration data. Linux web apps all share the same directory `/home/`, which is mounted into the custom Docker container. The GeoServer configuration could be stored in a subfolder of this directory, such as `/home/geoserver_data_dir/`. This would leave us with two options:

1. Modify the `GEOSERVER_DATA_DIR` to point to this location.
2. Synchronize the contents of this folder to the default GeoServer data directory location.

**Manage raster data**

The possibility of splitting out raster and vector data into separate GeoServer instances was discussed. The GeoServer that uses raster data, which cannot be mounted into the App Service, would remain on IaaS, whereas the vector data servers could use Web App on Linux. The benefit of this would be that a common raster GeoServer could be used for all deployment because background mapping is consistent across deployments. Each vector-based GeoServer could be customized to particular deployment requirements. Longer term, the raster data is likely to be supplied via a third-party mapping service, and in this case the requirement for the IaaS element would be removed.

Unfortunately, we ran out of time on the hackfest to progress any further with the Web App on Linux scenario.

## Conclusion

During the hackfest we successfully demonstrated that the ROAM application can be migrated to Azure, with the majority of services running on PaaS offerings. The value stream mapping exercise identified multiple manual steps that added lengthy wait time between development and release. The introduction of DevOps practices such as continuous integration, continuous delivery, and release management will vastly reduce the time required to get a feature into production. 

Somerset County Council is now putting plans in place to transition their live ROAM environments to Azure.

