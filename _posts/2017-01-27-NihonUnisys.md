---
layout: post
title:  "Building IoT solutions for Manufacturing Industry with Nihon Unisys"
author: "Hiroyuki Watanabe"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-01-27
categories: [IoT]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: In this IoT Hackfest, Microsoft worked with Nihon Unisys to develop an IoT solution for Manufacturing Industry using Azure Data Factory, Azure ML and Power BI.
language: The language of the article (e.g.: [English])
verticals: The vertical markets this article has focus on (e.g.: [Manufacturing & Resources])
---

Begin with an intro statement with the following details:

- Solution overview: They are SI for Manufacturing industry. They would like to create a standard (modeling) method for utilizing it for proposing Azure IoT solution to customers in manufacturing industry.
 
- Key technologies: Azure Data Factory (On-premise DB to Azure SQL Database), Azure ML, Power BI
 
- Core Team: 
Nihon Unisys:
- Tadashi Komorizono - Manufacturing System Department
- Yasuhiro Kurita - Manufacturing System Department
- Mieko Matsui - Manufacturing System Department
- Shigeo Yamagishi - Advanced Technology Department

Microsoft DX Japan:
- Hiroyuki Watanabe - Technical Evangelist
- Shigeo Fujimoto - Technical Evangelist

Nihon Unisys and Microsoft DX teams at the hackfest
 ![Hackfest](/images/2017-01-27-NihonUnisys/NihonUnisys4.png)
 ![Hackfest](/images/2017-01-27-NihonUnisys/NihonUnisys5.png)
 ![Hackfest](/images/2017-01-27-NihonUnisys/NihonUnisys6.png)
 ![Hackfest](/images/2017-01-27-NihonUnisys/NihonUnisys7.png)




## Customer profile ##

- Nihon Unisys Ltd. http://www.unisys.co.jp/

- Nihon Unisys Group is a Solution Implementer that creates value for its customers by providing integrated IT solutions services that extend from the identification to the solution of corporate issues. We do this by domain knowledge that arcs across business endeavors: from finance and manufacturing to transportation and the public sector, a cross-industry know-how that we integrate with the total capabilities of the Group. 

- The Group's consulting services assure seamless execution of all processes from formulation of business strategies to implementation of IT systems. The Group's expertise in IT supports companies in fully realizing their business strategies using IT, in a three-stage process that opens with "Business Innovation" proposals that recommend management reforms and business strategies. In the second stage, "Business Consulting," we elaborate on these proposals to develop business process flows and IT strategies. The third stage, "IT Consulting," sees the implementation of these plans. 

- The Group's strength lies in the ability to swiftly "Materialize" customers' management strategies and corporate vision through the smooth coordination and integration of all necessary processes, from upstream business consulting to downstream implementation and system integration. 



 
## Problem statement ##


- Business Problem: They are SI for Manufacturing industry. They would like to create a standard (modeling) method for utilizing it for proposing Azure IoT solution to customers in manufacturing industry. Especially, they would like to find out what Azure ML can do. For data analysis, there is another department using R Language. They would like to find out whether it is possible to collaborate with the analysis result and Azure ML. This IoT solutions standard modeling method for Manufacturing Industry saves time and cost of Unisys to deliver IoT solution to customer.
 
- There are three technical problems they would like to resolve during this Hackfest. 

1. Data transfer from Bluetooth Sensor Device to Azure IoT Hub.(Device sensors: ALPS) 
 
2. Data transfer from on-premises DB to DB on Azure.(Data that was collected through batch processing is transferred from On Perm to Azure) 

3. Analyze data acquired from manufacturing equipment in the manufacturing industry using Azure ML.(Cable Manufacturer.) 
 
 

 
## Proposed Solution ##


Regarding the above technology problem, for (1) above they will collect related information about conference space area via the devices and send to Azure IoT Hub, and transfer the data from On Premises DB to Azure DB the data will be analyzed from the data collected where ML will be applied to help bring predicitive analytics to the solution. Establish a foundation of IoT Solution for Manufacturing Industry from data collection of Bluetooth devices through IoT Hub, data analysis on Azure ML, output using Power BI.  As SIer, Unisys will expand the opportunity of Azure proposal to the end customer (Manufacturing industry). 

- Architecture diagram/s (**required**). Example below:

 ![IoT Architecture Diagram](/images/2017-01-27-NihonUnisys/NihonUnisys1.png)
 ![IoT Architecture Diagram](/images/2017-01-27-NihonUnisys/NihonUnisys2.png)
 ![IoT Architecture Diagram](/images/2017-01-27-NihonUnisys/NihonUnisys3.png)


## IoT Device Information ##
- Customer had Sensor Network Module design their devices for them, which were manufactured by ALPS.  It is running Firmware.  They are using the Azure IoT Device SDK and writing their device app in C.

- URL
http://www.alps.com/j/iotsmart-network/


## Technical delivery ##
This section will include the following details of how the solution was implemented:

- Pointers to references or documentation

  - MVP Summit 2016 IoT Workshop
  https://github.com/Microsoft/mvp-summit-iot-workshop#mvp-summit-2016-iot-workshop

  - IoT Kit Hands on material
  https://doc.co/ip7K2W/NsXXfD

  - Move data between on-premises sources and the cloud with Data Management Gateway
  https://docs.microsoft.com/ja-jp/azure/data-factory/data-factory-move-data-between-onprem-and-cloud

  - Stream Analytics & Power BI: A real-time analytics dashboard for streaming data
  https://docs.microsoft.com/ja-jp/azure/stream-analytics/stream-analytics-power-bi-dashboard


 
## Conclusion ##
Manufacturing Industry need to consider a lot of works to introduce IoT solutions. For example, they need to consider sections on each technology used in code artifacts, device details, code language, device security, data communication and so on. 
Nihon Unisys has a deep knowledge and experience of introducing IoT solution for manufacturing industry. And they were able to deepen their knowledge of Azure Data Factory, Azure ML and Power BI through Hackfest. Thye have built their IoT solution using Microsoft IoT Technology and they can speed up their preparation of their proposal for their customer in Manufacturing Industry.


