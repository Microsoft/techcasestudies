---
layout: post
title:  "Implementing continuous deployment for Rockmetric using VSTS and Packer"
author: "Maninderjit Bindra"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-02-23
categories: DevOps
color: "blue"
#image: "{{ site.baseurl }}/images/rockmetric/rockmetric.jpg" #should be ~350px tall
excerpt: DevOps practices help B2B startup Rockmetric reduce deployment time from days to minutes using VSTS and Packer. This report details the implementation and the business impact of establishing key DevOps practices for Rockmetric. 
language: English
verticals: [Financial Services, Marketing]
---


<img src="{{ site.baseurl }}/images/rockmetric/rockmetric.jpg" width="400">

Rockmetric, an India-based business-to-business startup, implemented DevOps practices to help reduce its deployment time from 2 days to 30 minutes. This report details the implementation and the business impact of establishing key DevOps practices for Rockmetric. 

## Customer profile

[Rockmetric](http://www.rockmetric.com) is a modern customer data analytics platform powered by cognitive intelligence. The platform has a revolutionary digital analyst, *Alfred*. Alfred creates an intelligence layer driven by natural language processing (NLP) to deliver data and insights through a natural language Q&A interface.

The core platform allows unification of customer data and metrics from multiple sources such as transaction data, support tools (call, email, chat, social), internal databases, CRM, marketing campaign operations, email analytics, and so on. Teams can get visualizations, automate reports, create segments, and drive action across existing tools. Teams also can create custom alerts and notifications for important metrics. Rockmetric services receive a million hits a day.

**The hackfest team:**

* [Ruchit Rami](https://twitter.com/ruchitrami) – Director of Engineering, Rockmetric
* Anurag Shivastav – Senior Developer, Rockmetric
* Ajay Deekonda – Senior Developer, Rockmetric
* [Nimesh Mehta](https://twitter.com/mehtanimesh) – Founder, Rockmetric
* [Maninderjit Bindra](https://twitter.com/manisbindra) – Senior Technical Evangelist, Microsoft
* [Gandhali Samant](https://twitter.com/s_gandhali) – Senior Technical Evangelist, Microsoft

![Value Stream Mapping]({{ site.baseurl }}/images/rockmetric/value-stream-mapping.png)


![The team during the hackfest - image 1]({{ site.baseurl }}/images/rockmetric/rockmetric-hackfest-1.jpeg)


![The team during the hackfest - image 2]({{ site.baseurl }}/images/rockmetric/rockmetric-hackfest-2.jpeg)


## Problem statement ##

Microsoft performed a value stream mapping exercise with Rockmetric. In the process, we reviewed the solution architecture, processes, and tools used for developing and releasing new features to production. We found that some of the core technology components were deployed to virtual machine scale sets using custom Rockmetric virtual machine images (Linux-based). Several manual steps were involved in pushing application updates to virtual machine scale sets, such as baking new virtual machine images, generalizing the virtual machine, pushing the updated image to the virtual machine scale set, and manual testing (no automated testing). Scripts were available for some of the steps, but these were triggered manually.

These are some of the drawbacks we found with that release approach:

* Some of the steps had a dependency on a single individual. As a result, the lead time to deploy changes could vary depending on the availability of the individual. This would typically be around 1-2 days. 
* There were no automated UI regression tests. After new features were added, a lot of manual testing was involved to ensure no regression defects were introduced. The manual testing would take between 1-2 days.
* No automated performance and load testing was done. On one occasion, Rockmetric had seen major performance degradation to one of its services after a release, which they found when manually testing and later resolved.

As more and more customers are coming on to the platform, Rockmetric's priority was to optimize the release process.  

## Solution ##

### *Scope of hackfest* ###

We agreed that the scope of the hackfest would be to create a release pipeline based on Visual Studio Team Services (VSTS) for one of the key services deployed to virtual machine scale sets. The scope included continuous deployment, automated UI testing, and load testing. 

Rockmetric provided a prototype of the identified PHP service. (The original service contains customer IP code, so we worked with the prototype for this implementation.) We would use Packer with Ansible provisioning to automate creation of a virtual machine image with the latest application code baked in.

We also considered a single QA environment in the release pipeline. 

These are the key DevOps practices used in this scope:

* **Infrastructure as code:** We decided that all steps involved in setting up and updating the infrastructure, including steps to bake the virtual machine images, would be in configuration-managed files. This would include Azure Resource Manager templates, shell scripts, Packer files, Ansible files, and UI test and load test files. This would reduce dependency on a single individual for any of the steps.
* **Automated testing:** We decided to set up the initial UI tests using PHPUnit/Selenium, which the Rockmetric team would build upon. We also decided to set up initial load tests with JMeter, which the Rockmetric team would add to. Getting automated tests in place would bring down the lead time for manual testing to a few hours, and also improve reliability of the release.
* **Continuous deployment:** Every merge into the master branch of the repository would result in the latest application being deployed to the virtual machine scale set-based solution in the QA environment. We estimated that we would reduce the end-to-end deployment time to under an hour. 
* **Release management:** Every merge into the master branch would result in deployment to QA followed by automated testing. We expected the end-to-end release time with the initial set of tests to take around an hour. 

### *Solution overview* ###

*Overview diagram* 

![Diagram giving overview of the key application and build/release files]({{ site.baseurl }}/images/rockmetric/build-release-overview.png)


### *Description* 

The VSTS build is triggered each time code is merged into the master branch of the application GitHub repository. The VSTS build in the prototype is a simple one that just publishes files needed for the release. The successful build then triggers a release. The first step of the release pipeline packages the latest application files into an Azure VHD image using Packer. This virtual machine image (VHD) is stored in the configured storage account. 

In the next step, the Azure CLI group deployment command is used to push the new virtual machine image (with the latest application code baked in) to the virtual machine scale set. The last two steps are for UI testing (using PHPUnit and Selenium in this case), and load testing (using JMeter, with duration assertions). At the end of each successful release, the virtual machine scale set has the latest tested application code.

### *GitHub repository*

The [GitHub repository](https://github.com/maniSbindra/vsts-packer-vmss-php-webapp-release) contains the prototype application code for the PHP web application, the build/release scripts (including the Packer file, UI, and load-test files), and the VSTS release template.

### Implementation details ###

- **Step 1. Custom VSTS Linux agent configuration:**
  * The custom VSTS agent was set up by following the steps mentioned in the post [Deploy an agent on Linux](https://www.visualstudio.com/es-es/docs/build/admin/agents/v2-linux). 
  * Packer setup: Packer downloads are available at [https://www.packer.io/downloads.html](https://www.packer.io/downloads.html). The following script was used to configure Packer version 0.12.0 on the VSTS agent. 

    ```shell
sudo apt-get install unzip
wget https://releases.hashicorp.com/packer/0.12.0/packer_0.12.0_linux_amd64.zip
unzip packer_0.12.0_linux_amd64.zip
sudo mv packer /usr/bin/packer
```

  * Selenium and Xvfb setup: Xvfb and Selenium standalone were installed as services on the VSTS agent by following the steps described at [https://www.namekdev.net/2016/08/selenium-server-without-x-window-system-xvfb/](https://www.namekdev.net/2016/08/selenium-server-without-x-window-system-xvfb/). Before following these steps, Java and Selenium standalone 2.5.3 were installed on the agent using the following script.

    ```shell
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt-get update
sudo apt-get install openjdk-8-jdk
wget http://selenium-release.storage.googleapis.com/2.53/selenium-server-standalone-2.53.1.jar
sudo mkdir -p /usr/lib/selenium
sudo mv selenium-server-standalone-2.53.1.jar /usr/lib/selenium/
```

  * Firefox headless setup: Firefox headless was set up on the VSTS agent by following the steps described at [https://medium.com/@griggheo/running-selenium-webdriver-tests-using-firefox-headless-mode-on-ubuntu-d32500bb6af2#.6a3ma9zcg](https://medium.com/@griggheo/running-selenium-webdriver-tests-using-firefox-headless-mode-on-ubuntu-d32500bb6af2#.6a3ma9zcg).

    ```shell
sudo apt-add-repository ppa:mozillateam/firefox-next
sudo apt-get update
sudo apt-get install firefox
```

  * PHPUnit setup: PHPUnit was set up by following the steps described at [https://phpunit.de/manual/current/en/installation.html](https://phpunit.de/manual/current/en/installation.html). We installed PHPUnit 5.5.4 using the following script.

    ```shell
    sudo apt-get install php php-xml php-curl
    wget https://phar.phpunit.de/phpunit-5.5.4.phar
    chmod +x phpunit-5.5.4.phar
    sudo mv phpunit-5.5.4.phar /usr/bin/phpunit

    sudo curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/bin/composer

    mkdir -p ~/php-webdriver && cd ~/php-webdriver
    echo '{ "require-dev": { "phpunit/phpunit": "*", "facebook/webdriver": "dev-master" } }' > composer.json
    composer install
    sudo cp -R vendor/ /usr/lib/selenium/
    ```
  
- **Step 2. Build configuration: Build was configured to publish all files and folders in the repository.**

    ![Build configuration]({{ site.baseurl }}/images/rockmetric/vsts-build.PNG)
    

- **Step 3. Release configuration:**
    * A new release was configured to be created on each successful build.

        ![VSTS Release Triggers]({{ site.baseurl }}/images/rockmetric/vsts-release-triggers.PNG)
        

    * A single QA release environment was created.
    * Release environment variables: The environment variables that were configured for the release to QA are as follows: 
      * ARM_CLIENT_ID, ARM_RESOURCE_GROUP, ARM_STORAGE_ACCOUNT, ARM_SUBSCRIPTION_ID, and ARM_TENANT_ID are used by Packer to save the baked virtual machine image VHD into the configured Azure storage account. 
      * ADMIN_USERNAME and ADMIN_PASSWORD are the logon credentials for the virtual machine scale set virtual machines.
      * DEPLOYMENT_RESOURCE_GROUP is the resource group where the virtual machine scale set is deployed. These are passed to the different tasks in the release (as we will see in following steps).

      ![Release Environment Variables]({{ site.baseurl }}/images/rockmetric/environment-variables-for-release.PNG)
        

    * Bake virtual machine image with latest application updates baked in, using Packer: The execute shell script task invokes the build-vm-image.sh file and passes it the parameters required by Packer to create the virtual machine image in the configured Azure Storage account. The following image shows configuration of this shell script step:

        ![Bake VM Image using Packer]({{ site.baseurl }}/images/rockmetric/release-bake-vm-image-packer.PNG)
        

       The following code block shows key lines from the shell script. The environment variables are passed as positional parameters to the script in this example:

        ```shell
        export ARM_CLIENT_ID=$1
        export ARM_CLIENT_SECRET=$2
        export ARM_RESOURCE_GROUP=$3
        export ARM_STORAGE_ACCOUNT=$4
        export ARM_SUBSCRIPTION_ID=$5
        export ARM_TENANT_ID=$6
        export ADMIN_USERNAME=$7
        export ADMIN_PASSWORD=$8
        packer build ./packer-files/phpapp-packer.json 2&gt;&amp;1 | tee packer-build-output.log
        ```

       The Packer build configuration file php-packer.json has all the configuration to enable Packer to package the latest application code and bake the virtual machine image (including the base virtual machine image and Azure region). The following configuration section from the Packer configuration file demonstrates how the application dist folder is copied into the baked image using the file provisioner:

        ```json
        "provisioners": [
        .
        .
        {
        "type": "file",
        "source": "../dist",
        "destination": "/tmp/dist"
        },
        .
        .
        .
        ]
        ```
        
       Other than file and shell script provisioners, we used the Ansible provisioner, but Packer supports several other provisioners such as Chef, Puppet, PowerShell, and more. The output of the Packer build command is written to the file packer-build-command-output.log. Packer writes the image URI of the newly created VHD in the format:

        ```
        OSDiskUri: https://*********/system/Microsoft.Compute/Images/images/packer-osDisk.5a892bb0-*********-410f-8198-92c8b140390e.vhd
        ```

       The file packer-build-output.log contains the complete output of the Packer build command.

    * Replace the image URI in the Resource Manager template deployment parameters file with the URI of the VHD generated by Packer in the preceding step.
        
        ![Overwrite Image Uri Parameter value]({{ site.baseurl }}/images/rockmetric/overwrite-group-deployment-parameters.PNG)
        

        The shell script overwrite-azure-deployment-parameters.sh is invoked, which parses the Packer out log file to retrieve the URI of the newly created VHD. It then replaces the parameter value azuredeploy.parameters.json file. A code extract of the file is shown here:

         ```shell
         export imageodisk=$(cat packer-build-output.log | grep OSDiskUri: | awk '{print $2}')

        sed -i 's/@@VMUSERNAME@@/'"$ADMIN_USERNAME"'/g' azuredeploy.parameters.json
        sed -i 's/@@VMPASSWORD@@/'"$ADMIN_PASSWORD"'/g' azuredeploy.parameters.json
        sed -i 's|@@IMAGEURI@@|'"$imageodisk"'|g' azuredeploy.parameters.json
         ```
        
    * Push the updated virtual machine image to a virtual machine scale set using Azure CLI group deployment task. This task calls the deploy-vm-image.sh file and passes the resource group where the virtual machine scale set is deployed as a parameter.
        
         ![Push new vm image to vm scale set]({{ site.baseurl }}/images/rockmetric/push-new-vm-image-to-vmscaleset.PNG)
         

         This script then executes the Azure group deployment command, passing it the Resource Manager template and parameters file. The script is shown here:

         ```shell
         azure group deployment create -f azuredeploy.json -e azuredeploy.parameters.json -g $1 -n rocdashdeploy$2
         ```

    * Execute a Selenium-based smoke test on the application: The execute-site-smoke-tesh.sh script then executes the PHPUnit site smoke tests.

        ![Execute UI Smoke tests]({{ site.baseurl }}/images/rockmetric/execute-phpunit-selenium-unit-tests.PNG)
        

        ```shell
        phpunit $1site-smoke-test.php $2
        ```

    * Execute load tests on the application: The JMeter load test execution task then fires JMeter load tests against the site using the JMX file site-load-test.jmx (which has duration assertions).

        ![Execute JMeter Load tests]({{ site.baseurl }}/images/rockmetric/execute-jmeter-load-tests.PNG)
        

## Conclusion ##

The value stream mapping activity helped us prioritize the problem areas. The release pipeline created during the hackfest demonstrated how easy it was to set up continuous deployment with automated testing using VSTS on virtual machine scale sets. We achieved the following in the hackfest:

* The release time for deployment to an environment was brought down to around 30 minutes (from up to a few days).  
* Because the steps to bake the virtual machine image are now in the Packer template (IaC) and are a part of the release pipeline,  there is reduced dependency on a single individual for deployments. 
* The regression tests can now be automated as part of the release using PHPUnit and Selenium, which will reduce the duration of the manual test cycles to a few hours (from 1 to 2 days). 
* Because load tests with duration assertions are a part of the release, any increase in the response times will cause the release to fail.

With this implementation, it will be easier for Rockmetric to deploy the solution for its customers who prefer a separate deployment in a different Azure subscription by cloning and tweaking the release pipeline.

Since the hackfest, the Rockmetric team has started creating similar release pipelines for their production virtual machine scale set-based services.

A quote from the Rockmetric team:

> "Running IT systems at high scale is always challenging. With help from senior members of team Microsoft, we were able to automate a big, critical, and time-consuming process of product deployment and testing. Microsoft VSTS tools are highly integrated with other Microsoft services like Azure; the Microsoft team helped us achieve an optimal solution that makes life easy for our IT team so that we can focus on our core platform."
