---
layout: post
title:  "Using DevOps practices with ADAM Software to accelerate developer ramp-up"
author: "Nick Trogh"
author-link: "https://twitter.com/nicktrog"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-02-15
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: ADAM Software and Microsoft worked together to optimize ADAM Software's build and release pipeline. This report details the DevOps practices that were implemented for a complex Smart Content Hub marketing solution.
language: English
verticals: ["Retail, Consumer Products & Services"]
---

Microsoft and ADAM Software teamed up to optimize ADAM Software's build and release pipeline. Initially we performed a value stream mapping (VSM) exercise to clearly outline the current state of the DevOps lifecycle and to agree on the optimal future state. As a result of the VSM, we identified the key areas of focus for a hackfest, where we implemented the following DevOps practices:

- Infrastructure as code (IaC)
- Release management
- Automated testing

The implementation was based on Azure Resource Manager templates, Visual Studio Team Services release management, PowerShell scripting, and Visual Studio web performance and load testing projects.

The core hackfest team included members from ADAM Software and Microsoft.

**ADAM Software:**

- Tim Cools – Agile Coach
- Wouter Demuynck – Developer
- Samuel Verstraete – Developer
- Maarten De Meyer – Developer
- Peter Rooms – Tester

**Microsoft:**

- Nick Trogh ([@nicktrog](https://twitter.com/nicktrog)) – Senior Technical Evangelist

During the VSM exercise, the core team was joined by key stakeholders from all teams within ADAM Software and by Rasmus Hald from Microsoft.

## Customer profile ##

<img src="{{ site.baseurl }}/images/adamsoftware/smartcontenthub-logo.jpg" width="524">


[ADAM Software](https://www.adamsoftware.net/) is a leading digital asset management (DAM) and product content management (PCM) software provider, based in Ghent, Belgium. Its **Smart Content Hub** creates, manages, and distributes marketing material that engages customers at every touch point in a product’s lifecycle.

ADAM Software is a Microsoft Gold Certified Partner and its Smart Content Hub is qualified for the Azure Certified Program and available for customers through the Azure Marketplace.

ADAM Software and the Microsoft team in Belgium have had a close relationship for many years and the partnership for this hackfest began with an invitation to a DevOps hackathon in the Netherlands. There the ADAM Software developer team was introduced to the Microsoft DevOps solution offering and after discussing their current way of working, we agreed to organize a value stream mapping workshop at ADAM Software, followed by a DevOps hackfest.

Before the delivery of the VSM workshop, ADAM Software gave a high-level overview of the Smart Content Hub solution architecture to ensure that everyone was aligned on the major building blocks and the solution inner workings.

![High-level conceptual architecture]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-conceptual.png)


The Smart Content Hub is a web-based application and is actually a suite of products integrated in a single web front end. The different products surface for customers as application modules that can be enabled on a per-customer basis. A central *core module* contains the common logic across the different products and there is a central data store. The solution depends on a number of third-party and OSS products and libraries (for example, for search capabilities across the solution, Elasticsearch is used) and also provides integration with existing products at a customer (for example, SharePoint).

ADAM Software is providing its solution to its customers through a series of implementation partners, which perform the actual installation and operations at the customer. Note that Smart Content Hub can be installed on-premises at the customer, at a hoster, or with Azure Cloud Services.

![Smart Content Hub home page]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-homepage.jpg)


## Problem statement ##

To identify the bottlenecks, the team conducted a value stream mapping of the entire software delivery process, from the ideation phase to the delivery and support of the released product. The VSM exercises lasted a full day and involved an extended team with stakeholders from all the teams at ADAM Software (product management, product owners, testing, development, support).

The team was able to visualize the current state of the software delivery process and identify the areas with wasteful activities. There were lively and open discussions between all team members and the exchange of ideas resulted in numerous proposals to eliminate this waste.

The following diagram shows the results of the value stream mapping workshop. We used the latest production release of the Smart Content Hub (version 5.7) as the basis for the VSM workshop.

![Value Stream Map]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-vsmdiagram.jpg)


The VSM exercise helped the team identify key points about the current delivery process:

1. Setting up a developer environment for a new developer or to work on a production issue is largely a manual process, involving many steps. Often, assistance from senior developers is needed, significantly reducing the time they can work on developer activities. Currently, provisioning a development environment can take **up to 2-3 days**.
2. Once the development sprints are completed (6), a stabilization phase is performed to consolidate the deliverables from the different product teams. From the VSM diagram, the stabilization phase has a **percentage complete and accurate (%CA) of 15%**. This effectively means that multiple iterations are needed to get to a correct outcome.
3. A continuous integration (CI) build is in place based on Team Foundation Server, which also hosts the Git-based source code repositories.
4. Testing is primarily a manual task, performed by a dedicated test team. Currently some unit and integration tests are automated using MS Test and included in the CI build, but performance and load testing is lacking.
5. The deployment process into the test environments is completely automated. A web-based UI is in place to either deploy a test environment on-premises on VMWare or Microsoft Azure. For Azure, Azure Resource Manager templates are being used.
6. Creating the final release packages for customers is mainly a manual process and can take multiple iterations, hence a long time to complete. During the VSM, the team identified a **lead time of 6 hours** for an activity with a process time of 1 hour.
7. Due to the complexity of the solution architecture, support activities are primarily handled by the senior developers. This significantly reduces the amount of developer work that can be taken on during a sprint. Currently, during a development sprint, developers spend only **4.5 hours per day** actually developing. The remaining hours are spent on non-development activities.

>*Setting up a new run-time environment for our solution was fully automated but getting a developer environment set up can take up to 2-3 days.*...

>*Automating the provisioning of a developer environment can save us multiple days per developer per release, allowing developers to focus on their key strengths.*

>—Tim Cools, Agile Coach

After discussions with the team, we decided to take on the following items during the hackfest:

- **Automate the provisioning** of a developer environment including all third-party dependencies such as Elasticsearch. The process must be flexible to deploy a developer environment on a developer laptop, virtual machine on VMWare or provision an Azure virtual machine. Azure Resource Manager templates will be used for provisioning on Azure.
- Introduce automated **performance and load testing** of the suite using Visual Studio web performance and load testing projects.
- **Automate the creation of a release package** using Team Foundation Server release management and the artifacts of the CI build.

## Solution, steps, and delivery ##

The different products that are part of the Smart Content Hub are being developed by different teams at ADAM Software. There is currently an ongoing effort to move all products to Git-based source repositories. The team decided to take only the products within the scope of the hackfest that were already in a Git repository; this includes the Core, REST API, and Product modules.

The following diagram outlines the high-level components of the overall process and architecture for the hackfest.

![Solution envisioned by the team]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-envisionedsolution.png)


ADAM Software has a local vSphere environment as well as an Azure-based environment. A key requirement for the envisioned solution is that environments must be deployable to any of these infrastructures. For the developer environment, a developer should also be able to provision it on a local developer workstation. In addition, the provisioning process may be triggered manually but should also be capable of being integrated in the CI process that is already in place.

Let's look at the key deliverables of the hackfest in more detail and focus on the technologies that were applied to achieve this.

### Infrastructure as code (IaC) ###

During the hackfest, the team decided to use a combination of infrastructure as code and scripting for setting up a developer environment. The base operating system (OS), developer tools, and SQL Server database are provided through IaC; the setup of the source code and configuration of the environment are performed through PowerShell scripting. In addition, the solution makes use of open source software (OSS) such as Elasticsearch. These software components are also installed and configured using PowerShell scripting.

**Azure Resource Manager template**

When provisioning a developer environment on Azure, Azure Virtual Machines are being instantiated through **Azure Resource Manager templates**. This allows for flexible configuring of the infrastructure components. During the hackfest, the team created a single developer environment Resource Manager template consisting of a single Azure virtual machine. However, through IaC, this could easily be modified to separate the database server from the developer machine.

The base image for the Azure virtual machine is a custom virtual machine image that was captured from a generalized virtual machine on which Visual Studio and SQL Server were installed. In the Resource Manager template, the virtual machine image is linked to using its storage URI. Similarly, when creating a developer environment on vSphere, a virtual machine snapshot containing Visual Studio and SQL Server is being used.

The following is a snippet from the Resource Manager template for creating the developer environment on Azure.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  	"resources": [
		{
			"apiVersion": "2016-03-30",
			"type": "Microsoft.Compute/virtualMachines",
			"name": "[parameters('vmName')]",
			"location": "[variables('location')]",
			"dependsOn": [
				"[concat('Microsoft.Network/networkInterfaces/', variables('vpnNicName'))]"
			],
			"properties": {
				"hardwareProfile": {
					"vmSize": "[parameters('vmSize')]"
				},
				"osProfile": {
					"computerName": "[parameters('vmName')]",
					"adminUsername": "[parameters('adminUsername')]",
					"adminPassword": "[parameters('adminPassword')]"
				},
				"storageProfile": {
					"osDisk": {
						"osType": "Windows",
						"name": "osdisk",
						"image": {
							"uri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/images/adamdevimage-osDisk.20161219.vhd')]"
						},
						"vhd": {
							"uri": "[concat('http://',parameters('storageAccountName'),'.blob.core.windows.net/',variables('vmStorageAccountContainerName'),'/',variables('OSDiskName'),'.vhd')]"
						},
						"caching": "ReadWrite",
						"createOption": "FromImage"
					}
				},
				"networkProfile": {
					"networkInterfaces": [
						{
							"id": "[resourceId('Microsoft.Network/networkInterfaces',variables('vpnNicName'))]"
						}
					]
				}
			},
			"resources": [
				{
					"type": "Microsoft.Compute/virtualMachines/extensions",
					"name": "[concat(parameters('vmName'),'/joindomain')]",
					"apiVersion": "2015-06-15",
					"location": "[resourceGroup().location]",
					"dependsOn": [
						"[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]",
						"[resourceId('Microsoft.Compute/virtualMachines/extensions', parameters('vmName'), 'bootstrap')]"
					],
					"properties": {
						"publisher": "Microsoft.Compute",
						"type": "JsonADDomainExtension",
						"typeHandlerVersion": "1.3",
						"autoUpgradeMinorVersion": true,
						"settings": {
							"Name": "[parameters('domainToJoin')]",
							"OUPath": "[parameters('ouPath')]",
							"User": "[concat(parameters('domainToJoin'),'\\',parameters('domainUsername'))]",
							"Restart": "true",
							"Options": "[parameters('domainJoinOptions')]"
						},
						"protectedsettings": {
							"Password": "[parameters('domainPassword')]"
						}
					}
				},
				{
					"type": "Microsoft.Compute/virtualMachines/extensions",
					"name": "[concat(parameters('vmName'),'/bootstrap')]",
					"apiVersion": "2015-06-15",
					"location": "[resourceGroup().location]",
					"dependsOn": [
						"[concat('Microsoft.Compute/virtualMachines/', parameters('vmName'))]"
					],
					"properties": {
						"publisher": "Microsoft.Compute",
						"type": "CustomScriptExtension",
						"typeHandlerVersion": "1.4",
						"settings": {
							"fileUris": [
								"https://<myaccount>.blob.core.windows.net/public/azure_bootstrap.ps1",
								"https://<myaccount>.blob.core.windows.net/public/makecert.exe",
								"https://<myaccount>.blob.core.windows.net/public/winrmconf.cmd"
							],
							"commandToExecute": "[concat('powershell -ExecutionPolicy Unrestricted -file azure_bootstrap.ps1 ',variables('hostDNSNameScriptArgument'),' ',parameters('sqlSaPassword'),' ',parameters('vmName'),' ',parameters('isDevMachine'))]"
						}
					}
				}
			]
		}

```

**Configure the developer virtual machine**

Once the developer virtual machine is provisioned, it can be further configured to host the Smart Content Hub developer environment.  Once configured, this developer virtual machine allows a developer to start coding on a specific version, and to run and debug the application and its dependencies.

The following software components need to be installed and configured on the virtual machine:

* Microsoft Internet Information Services (IIS): application pools, websites, ...
* Microsoft ASP.NET
* SQL Server database engine: new database and associated Smart Content Hub tables, indexes, ...

In addition, the Smart Content Hub solution also leverages open source software, namely [Elasticsearch](https://www.elastic.co/), to perform product searches in the central database. As part of the configuration of the developer environment, the Elasticsearch open source software needs to be installed and then configured. This software, as well as other third-party software components, is installed through PowerShell scripting in the same manner as the Microsoft software components.

To kickstart the configuration of the developer virtual machine, the Resource Manager template configures a [custom script extension on the virtual machine](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-extensions-customscript?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json), which will invoke a PowerShell script on the newly created virtual machine.

![Configuration PowerShell script]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-setupdev.jpg)


Currently the Git source code repositories are hosted on Team Foundation Server on ADAM Software's on-premises infrastructure. To allow the source code to be downloaded on the Azure virtual machine, the team created an **Azure VPN gateway** to connect with the on-premises VPN appliance.

Below is a snippet from the developer environment configuration PowerShell script.

```powershell
function New-AdamDevelopmentInstance(
    [parameter(Mandatory=$true, Position=1)][string]$Path,
    [parameter()][string]$Version = '5.8',
    [parameter()][string]$Site = 'Default Web Site',
    [parameter()][object]$AdamConfiguration = @{
        BasePath = 'C:\ADAM'
        RegistrationName = 'ADAMNET'
        AdministratorPassword = '******'
    },
    [parameter()][object]$DbConfiguration = @{ 
        LicenseDatabasePath = 'C:\ADAM\AdamLicensing'
        Server = '(local)' 
        UserName = '******'
        Password = '********'
    },
    [parameter()][string]$LicenseKeyFolder,
    [parameter()][switch]$Force
)
{
    if (-not (Test-IsElevated)) {
        throw 'This command must be run with elevated privileges.'
    }

    # Parse the requested ADAM version.
    $TargetVersion = ConvertTo-Version($Version);
    Write-Host "Installing a debug environment for ADAM $($TargetVersion.ToString())..."

    if (-not (Test-AdamSourceCode $Path)) {
        throw "The specified path '$Path' does not appear to contain the ADAM source code."
    }

    Install-RegistryKeys -Path $Path -Version $TargetVersion
    Install-Services -Path $Path -Version $TargetVersion
    
    # Determine an installation location for this development installation.
    $InstancePath = Join-Path $AdamConfiguration.BasePath "$($TargetVersion.Major).$($TargetVersion.Minor)"
    $DatabasePath = Join-Path $InstancePath 'Database'
    $WorkingPath = Join-Path $InstancePath 'WorkingFolder'
    $DataPath = Join-Path $InstancePath 'DataFolder'
    $StoragePath = Join-Path $DataPath 'Default\1'
    $DatabaseName = "ADAM.$($TargetVersion.Major).$($TargetVersion.Minor)"
    $LicenseDbPath = $AdamConfiguration.BasePath

    New-Item $StoragePath -ItemType Directory -Force | Out-Null

    $LicenseKeyFile = Join-Path $LicenseKeyFolder "dev$($TargetVersion.Major)$($TargetVersion.Minor).xml"
    if (-not (Test-Path ($LicenseKeyFile))) {
        throw "License file does not exist at '$LicenseKeyFile'."
    }
    
    New-AdamDatabase -Name $DatabaseName -Path $DatabasePath -Password $AdamConfiguration.AdministratorPassword -DbConfiguration $DbConfiguration -Force:$Force -Development
    Enable-AdamDatabase -Name $AdamConfiguration.RegistrationName -DatabaseName $DatabaseName -WorkingPath $WorkingPath -DataPath $DataPath -StoragePath $StoragePath -DbConfiguration $DbConfiguration -Development
    Install-AdamLicense $LicenseKeyFile -DatabaseName $DatabaseName -DatabasePath $LicenseDbPath -DbConfiguration $DbConfiguration -Force:$Force -Development

    $StudiosPath = Join-Path $Path 'Source\Studios'
    $VirtualDirectory = "ADAM$($TargetVersion.Major)$($TargetVersion.Minor)"
    Install-AdamStudio `
        -Name 'Central' `
        -Type 'StudioSelector' `
        -Path (Join-Path $StudiosPath 'Adam.Web.StudioSelector') `
        -RegistrationName $AdamConfiguration.RegistrationName `
        -VirtualDirectory $VirtualDirectory `
        -AdamConfiguration $AdamConfiguration `
        -DbConfiguration $DbConfiguration `
        -Force:$Force `
        -Development

    $BinariesDir = (Join-Path $Path 'bin\Debug')
    $AssetStudioDir = (Join-Path $StudiosPath 'Adam.Web.AssetStudio\bin')
    $AssembliesToCopy = @(
        'Adam.Web.Extensions.Design.dll',
        'Adam.MediaEngines.dll',
        'Adam.Core.GraphicEngines.dll',
        'Adam.Core.MediaEngines.dll'
    )

    foreach ($AssemblyToCopy in $AssembliesToCopy) {
        Copy-Item (Join-Path $BinariesDir $AssemblyToCopy) (Join-Path $AssetStudioDir $AssemblyToCopy)
    }
}

Set-Alias SetupDev New-AdamDevelopmentInstance
```

### Automate the developer provisioning ###

The Resource Manager template and associated PowerShell scripts for provisioning the developer environment were then integrated in **vManager**, a web-based provisioning tool that was developed in-house at ADAM. Using vManager, anyone in ADAM Software can provision an environment. Using a set of input parameters, a specific release version, OS, and SQL Server version can be chosen, as well as the target environment (vSphere or Microsoft Azure).

![vManager: Web-based provisioning tool]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-vmanager.jpg)


To allow environments to also be created automatically as part of the CI process, a Team Foundation Server build definition is created that will invoke the Resource Manager deployment or on-premises deployment. An important learning from the hackfest is that creating a virtual machine from a custom image can take a considerable amount of time when the virtual machine image is not on the same storage account. The team has decided to put the image and the target virtual machines on the same storage account for performance reasons.

The following code snippet shows the PowerShell script that is invoked by the build definition to perform a Resource Manager deployment. 

```powershell
if($isDevMachine -gt 0)
{
	# dev machines are all deployed on the same rg/sa to speed up instantiation from disk
	$saName = '****'
	$rgName = '****'
	$domainPassword = ConvertTo-SecureString "********" -AsPlainText -Force

	#note: parameters NEED to be created like this to allow for securedstring to be passed
	$parameters = New-Object -TypeName Hashtable
	$parameters.Add('storageAccountName', $saName)
	$parameters.Add('adminUsername', $adminUserName)
	$parameters.Add('adminPassword', $adminPassword)
	$parameters.Add('vmName', $hostname)
	$parameters.Add('vmSize', $vmSize)
	$parameters.Add('sqlSaPassword', $sqlSaPassword)
	$parameters.Add('domainPassword', $domainPassword)
	$parameters.Add('isDevMachine', $isDevMachine)
		
	Write-Host "### Starting deployment of resource group $rgName from template $templatePath"
	New-AzureRmResourceGroupDeployment -ResourceGroupName $rgName -TemplateFile $templatePath -TemplateParameterObject $parameters -Name $deploymentName -Verbose
}
```

![Provisioning build definition]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-builddefinition.jpg)


The vManager service in turn will use the [TFS client API](https://www.visualstudio.com/en-us/docs/integrate/extensions/reference/client/api/tfs/build/restclient/buildhttpclient2_1) to trigger a new build of the above mentioned build definition.

```csharp
	public static class TfsManager
	{
		public static Build QueueDeployBuild(string instanceName, string templateName, string b64Json, bool isAzure, bool isDevMachine, int extraDiskSize, string requesterName)
		{
			var buildDefinition = isAzure ? ConfigurationManager.AppSettings["AzureDeployBuildDefinition"] : ConfigurationManager.AppSettings["vSphereDeployBuildDefinition"];

            var build = new BuildHttpClient(new Uri(ConfigurationManager.AppSettings["tfsUri"]), new VssAadCredential());

			var definitions = build.GetDefinitionsAsync(project: ConfigurationManager.AppSettings["projectCollection"], type: DefinitionType.Build).Result;

			var target = definitions.First(d => d.Name == buildDefinition);

			Dictionary<string, object> parameters = new Dictionary<string, object>();
			parameters["VMName"] = instanceName;
			parameters["TemplateName"] = templateName;
			parameters["DeploymentTemplate"] = b64Json;
			parameters["extraDiskSize"] = extraDiskSize;
			if (isAzure)
			{
				parameters["isDevMachine"] = isDevMachine ? 1 : 0;
			}

			string parameterString = JsonConvert.SerializeObject(parameters);

			var res = build.QueueBuildAsync(
				new Build
				{
					Definition = new DefinitionReference
					{
						Id = target.Id
					},
					Project = target.Project,
					Parameters = parameterString
				}).Result;

			var res2 = build.AddBuildTagAsync(ConfigurationManager.AppSettings["projectCollection"], res.Id, $"Requested by {requesterName}").Result;

			return res;
		}
  }
```


### Automate release package creation ###

Another wasteful activity that was identified by the team during the VSM exercise was the creation of the release packages. As mentioned, the Smart Content Hub is a suite of products. For each of these products there is a separate installer package. During the release packaging, the different installer packages need to be selected, included in a consolidated zip archive, and be stored on a central publishing location, ready for distribution to customers.

These steps were previously manual steps, which required several repetitive actions to be performed by the person responsible for the packaging. During the last production release, several iterations had to be performed of this operation, resulting in significantly increased lead times.

The team decided to automate these tasks during the hackfest. The approach that the team chose was to leverage **TFS Release Management**. A release definition was created that invokes a PowerShell script that performs the selection of the appropriate installer packages, using the correct versions, and does the zip file creation and copying to the correct publish location. As of the current state, the release definition is triggered manually. This may become an automated process in the future.

![Release definition for release package creation]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-releasedefinition.jpg)


## Conclusion ##

It was a pleasure to contribute to this hackfest with such a talented technical team and to see such a significant positive impact on the software delivery process. The goal set forward at the beginning of the hackfest was to reduce waste activities during the development and release phases. This would then consequently result in shorter lead times, more time being spent on core development activities and eventually in more value to be created for the end customer.

During the hackfest the team introduced **infrastructure as code** and added **automation** to remove manual operations during the delivery process. These activities will result in significantly **less time spent in non-development activities** by all developers and especially free up more time from the senior development team.

The hackfest also demonstrated that **both Microsoft and open source** components could be seamlessly integrated in a common automated delivery process, deploying flexibly across on-premises, private, or public cloud infrastructures.

Some highlights:

* The time to set up a developer environment was reduced **from 2-3 days to less than one hour**.
* The time to create a release package was reduced from **hours to minutes**.
* A start was made to introduce performance and load testing. This will result in proactively resolving performance issues and higher customer satisfaction.

In the meantime, the team has outlined a list of future improvements to the software and delivery process. Keep the hackfest spirit alive in the organization, team!

## Appreciation ##

All of this would not have been possible without the dedication of the core team during the hackfest, but also the commitment from the entire ADAM Software management to support ongoing improvement activities to build better software. It was an honor to work with all of you!

![Demoing the hackfest results for the entire team]({{ site.baseurl }}/images/adamsoftware/smartcontenthub-demotime.jpg)


## Resources ##

- [Authoring Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates)
- [How to capture a VM image from a generalized Azure VM](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-capture-image)
- [Windows VM Custom Script extensions with Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/virtual-machines/virtual-machines-windows-extensions-customscript?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
