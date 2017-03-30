---
layout: post
title:  "Enhancing the software development process by implementing DevOps at IMSS"
author: "José Raúl Guerrero Carreón"
author-link: "http://www.superneurona.com"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-27
categories: [DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Mexico's IMSS agency is implementing processes to manage its complex systems and comply with government requirements. In this project, Microsoft worked with an IMSS team to adopt DevOps practices for Java projects.
language: [English]
verticals: [Public Sector, Healthcare]
---


The Instituto Mexicano del Seguro Social (IMSS) has been working to define a process to help it manage the complex systems that administer all information around the healthcare services it provides. All of this must comply with MAAGTICSI (the Administrative Manual of General Application in Information Technology and Communications and Information Security), which extracts a common framework from CMMI, COBIT, ITIL, and others. This framework regulates how government agencies operate their IT organizations.

Microsoft worked with IMSS to validate the actual application life management process and to identify gaps within the practice. We also wanted to help them implement DevOps practices as part of their strategy.

Our solution would implement a continuous integration process to help them identify gaps in their practice and to validate the generation of needed outputs. In addition, we would implement validation with code analysis and train the team on unit testing.

To help reduce the time to deliver against the development environment on Azure, we would collaborate with IMSS on the creation of script for continuous delivery using Team Foundation Server.

At the end of this execution, we would train them on how to implement Azure Application Insights to gain information about the application in development. 

**Key technologies used:**

- Java
- Red Hat Enterprise Linux
- GlassFish application server
- Maven
- Team Foundation Server 2015
- Visual Studio Team System Build
- SonarQube
- Azure Virtual Machines

**The core team:**

IMSS:

- Fabiola Lizette Cardenas Delgado – Architecture and Repositories
- José Luis Alavez Manzano – Software Architecture
- Monserrat Alvarado Martínez – TFS
- Nallely Meza Bustamante – Functional, Regression, and Automated Testing
- Lydia Nogueron – Repositories
- Samuel Partida Contreras – Development
- Gustavo Ríos Pérez – Development
- Silvia Sánchez González – Development Lead/Product Owner
- Francisco Figueroa Guadarrama – Environment Configuration
- Rocio Bravo – Testing
- Gerardo Romero – DBA

Microsoft:

- Raul Guerrero – Technical Evangelist 
- Eduardo Sánchez – Technical Evangelist

## Customer profile ##

The [Instituto Mexicano del Seguro Social](http://www.imss.gob.mx) (IMSS) is a Mexican government organization that has managed public healthcare and social protection of Mexicans since its founding in 1943. It combines research and medical practices with the administration of resources for retirees, workers, and their families, in accordance with Mexico's Social Security Law. Today, more than half of the Mexican population has a connection to the institute, the largest of its kind in Latin America. 

## Problem statement ##

IMSS has a large development team, principally subcontracted to Softtek, one of the country's biggest developer outsourcers. They want to guarantee that their source code is current and working, and to validate that all components are available in source control. If a team member breaks the code compilation, they want to receive an early warning to take action. In addition, they want to implement code analysis of the repository to enhance the quality of the solution and use unit tests to protect their application from changes.

The principal benefit they seek is to reduce the time spent creating the development environment, which in most cases takes three weeks because of the internal process they need to follow. They also want to enhance the quality and reliability of the development process.

## Architecture overview ##

The architecture consisted of a web application developed on Java and running on a GlassFish application server. The application had a database layer using MongoDB. For scaling purposes, the application layer load was distributed with an F5 load balancer in the production environment.

![Architecture Exercise]({{ site.baseurl }}/images/IMSSDevOps/VSMArchitectureSession.png)


![IMSS System Architecture]({{ site.baseurl }}/images/IMSSDevOps/2017-02-12_15-44-23architecture.png)


### Development frameworks ###

- JSF 2.2 for the presentation layer
- Spring Framework to implement security and data access
- Morphia for mapping Java objects to MongoDB
- Jasper for report generation

## Solution, steps, and delivery ##

As the project got under way, the first activity was to conduct a value stream mapping (VSM) exercise that let the group investigate the existing process and find the principal areas of wasted time.

### Value stream mapping ###

![IMSS Value Stream Mapping]({{ site.baseurl }}/images/IMSSDevOps/20170110_161917.jpg)


![IMSS Value Stream Mapping]({{ site.baseurl }}/images/IMSSDevOps/20170110_170116.jpg)


![IMSS Value Stream Mapping]({{ site.baseurl }}/images/IMSSDevOps/IMG_6816.JPG)


As a result of this exercise, the team was able to determine that the total time needed to get a requirement into production was 1,580 hours, or 197 days from analysis to production.

![IMSS Value Stream Mapping]({{ site.baseurl }}/images/IMSSDevOps/20170110_184321.jpg)


The root causes of the wasted time were related to provisioning the infrastructure to the actual datacenter and to a heavy process for change authorization that required synchronization of different areas of IT as well as approval by a change committee.

As a result of the analysis, we found the following activities that, if improved, could have a positive impact on the lead time:

- Requests for developer environments and production infrastructure could delay up to 3 months.
- Once a change is ready for production, the organization spends 144 hours on authorization and implementation of environments.
- The requirements definition follows MAAGTICSI, a process based on Capability Maturity Model Integration (CMMI), resulting in a document-heavy process that delays execution.
- A lot of time was lost on bugs reported in production and on QA-related activities.

During the project activities, our focus would be on the following:

- Enable code analysis and test-driven development activities validated during a build in a **continuous integration** strategy.
- Reduce time required to test deploying to a developer environment defined on Azure implementing **continuous deployment**.
- Facilitate the committee authorization and deployment between environments using **release management**.
- Help the team implement **automated testing** of their functional scenarios using web-oriented tools to reduce wasted time related to this task.

Continuous integration and continuous deployment would be required for the execution. Release management and automated testing would be training activities during the project.

### Continuous integration ###

During the process of creating the build pipeline, it was necessary to execute and verify the Maven script, install and configure a SonarQube Server, and configure the build agent to support the Java compilation, Maven, and [cURL](https://curl.haxx.se/). 

![Maven Configuration]({{ site.baseurl }}/images/IMSSDevOps/2017-02-09_23-22-22MavenBuildStep.png)


![Agent Configuration]({{ site.baseurl }}/images/IMSSDevOps/2017-02-09_23-53-58AgentUserCapabilities.png)


### Continuous delivery ###

![Continuous Delivery Configuration]({{ site.baseurl }}/images/IMSSDevOps/2017-02-09_23-59-09ContinousDeployment.png)


In the continuous delivery scenario, the first step was to create the development environment using a CentOS virtual machine, with Apache and GlassFish configured.

The following steps were required in this process:

- Use Maven to execute the pom.xml file and get the build outputs.
- Copy only the \*\*/\*.war files to $(build.artifactstagingdirectory).
- Use a PowerShell inline script to set the cURL path inside the agent.

  ```
  echo "##vso[task.setvariable variable=PATH;]$env:PATH;C:\Program Files\curl-7.52.1-win64-mingw\bin"
  ```

- cURL upload files to copy the \*.war files to the development server using secure FTP (SFTP) service.

  ![Continuous Delivery Configuration]({{ site.baseurl }}/images/IMSSDevOps/2017-02-10_00-14-40curl.png)


- Run a shell command to undeploy and deploy the new package.

  ```
  export  PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/[localuser]/.local/bin:/home/[localuser]/bin:/home/[localuser]/jdk1.8.0_111  /bin

  /home/[localuser]/glassfish4/bin/asadmin -u admin -W /home/[localdirtopasswordfile]/passwordfile undeploy ejemplo-0.0.1-SNAPSHOT
  /home/[localuser]/glassfish4/bin/asadmin -u admin -W /home/[localdirtopasswordfile]/passwordfile deploy /home/[localuser]/ejemplo-0.0.1-SNAPSHOT.war
  ```

### Automated testing ###

Executing the automated testing required multiple activities, beginning with training for testing using Visual Studio Team Services and the distinct kinds of automation they can implement.

During the execution, some testing was automated with Selenium because some scenarios require execution on multiple browsers and some characteristics of the system, such as use of Angular, were not automated with coded UI.

This automation requires:

- The recording of the test automation with Selenium IDE that is a Firefox add-on.
- Exporting the recorded test to nUnit C# code and adding it to the current coded UI projects.
- Execution of Mozilla GeckoDriver in the machine executing the automation.

## Conclusion ##

As a result of this exercise, the Instituto Mexicano del Seguro Social was able to better understand the DevOps practices and drastically reduce the time required to validate a scenario in the dev environment from more than a week to only a few minutes.

In addition, they got relevant information about the quality of the software delivered by their contractor, helping them to use metrics to make decisions about issues such as technical debt.

All of this information is sent by email to the people who need it as a basis for making decisions and taking actions.

As an added activity, the execution includes training in automated testing using web and performance testing, coded UI, and Selenium test automation of their application. They are reducing the manual testing scenarios from 1 hour of manual tests to a couple of minutes of automated execution. Also, training in release management configuration was delivered for future implementation, as planned at the beginning of the project.

At this time, more teams are being trained by the members who took part in this event so they can implement these learnings in other Java-related projects. They also are implementing additional practices as a result of the training we delivered.

