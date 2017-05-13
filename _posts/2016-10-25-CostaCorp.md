---
layout: post
title:  "IoT pH Flow Controls and Automation"
author: "Blain Barton"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/costaauthors.png"
date:   2016-10-25
categories: IoT
color: "blue"
#image-device: "{{ site.baseurl }}/images/costadevice.png" #should be ~350px tall
excerpt: This article showcases IoT setup for Costa Farms using the Atlas Scientific PH Sensor and Adafruit Feather M0 Wifi using the Arduino IDE, sending PH messages to Microsoft Azure.
language: English
verticals: Agriculture/Farming
---

## IoT - pH Flow Controls and Automation ##
 
## Costa Farms ##
[Costa Farms](http://http://www.costafarms.com/) is a third-generation, family-owned group of companies headquartered in beautiful Miami, Florida (aka, plant paradise!). The company sprouted in 1961 when the founder, Jose Costa Sr., purchased 30 acres south of Miami to grow fresh, vine-ripened tomatoes in the winter and calamondin citrus in the summer. That soon morphed into houseplants, and the Costa Farms family started innovating and introduced new houseplants such as the canela tree, Orchids and Cecilia Aglaonema. 
 
## Pain point ##
It's time consuming, costly, and difficult for growers to measure and regulate pH throughout the day and across the lifecycle of a plant.  Indeed pH levels are one of the key factors in plant health. Furthermore pH levels in the water and nutrient streams change constantly.  To increase plant health via nutrient uptake in turn promoting higher yields, pH needs to be more closely monitored and adjusted in real time.

Impact Statement:
The Azure loT system will help Costa Farms increase revenue and profitability in using modern technology in Agriculture/Farming. Recall optimal pH levels are essential for nutrient uptake. Increased nutrient uptake directly effects yields in turn driving improved revenue per plant harvested. On the cost side, the system will allow growers to be more productive and utilized across all growing activities as they spend less time constantly checking pH by hand.
 
## Solution ##
Use Azure IoT Suite and Azure IoT Hub with PH sensor devices on hydro water systems for testing the correct PH. The goal was to use PH sensors for testing water entering and leaving their hydro systems. Build a foundation that customer can use in their future expansions

Core Team:

1. Mani Peddada - BI Architect, mpeddada@costafarms.com
2. Rodney Sanchez - Software Developer, rsanchez@costafarms.com
3. Blain Barton - Microsoft Senior Technical Evangelist, blainbar@microsoft.com
4. Joe Raio - Microsoft Senior Technical Evangelist, Joe.Raio@Microsoft.com
5. David Crook - Microsoft Technical Evangelist, dcrook@microsoft.com

There are multiple processes for this particular IoT solution.  

1. Obtaining IoT Hardware, Adafruit Feather MO Wifi, breadboard, Atlas Scientific pHSensor, and wires to connect to wifi and Microsoft Azure. IoT Hub -> Stream Analytics -> Power BI
2. Configuring software for Adafruit Feather using Arduino IDE and compiler. Code was written in C and referenced sources from Atlas and Arduino were custom code for Azure. 
3. IoT Hub -> Stream Analytics -> SQL Server
4. IoT Hub -> Stream Analytics -> Event Hub -> Azure Functions - > Twillio to phone

Here's a link to the technical video (video) that explains the whole hardware and software processes.

Below is a screen shot of the hardware used for the Costa "Proof of Concept".
(pic)

##Solution Architecture##
#image-Arch: "{{ site.baseurl }}/images/costadiagram.png" 

##Device used & Code artifacts
Article 1 of 2 - Setting up the hardware for pHSensor - [https://blogs.msdn.microsoft.com/blainbar/2016/10/25/hardware-assembly-for-the-adafruit-feather-m0-wifi-with-the-atlas-scientific-ph-sensor-for-remotely-monitoring-ph-water-levels-in-microsoft-azure-article-1-or-2/](https://blogs.msdn.microsoft.com/blainbar/2016/10/25/hardware-assembly-for-the-adafruit-feather-m0-wifi-with-the-atlas-scientific-ph-sensor-for-remotely-monitoring-ph-water-levels-in-microsoft-azure-article-1-or-2/)


Article 2 of 2 - Setting up the software for pHsensor - [https://blogs.msdn.microsoft.com/blainbar/2016/10/25/setting-up-software-for-the-adafruit-feather-m0-wifi-using-the-arduino-ide-and-c-code-for-remotely-monitoring-ph-sensors-in-microsoft-azure-article-2-of-2](https://blogs.msdn.microsoft.com/blainbar/2016/10/25/setting-up-software-for-the-adafruit-feather-m0-wifi-using-the-arduino-ide-and-c-code-for-remotely-monitoring-ph-sensors-in-microsoft-azure-article-2-of-2)/


##Opportunities going forward

Costa Farms is wanting to write an article for one of their magazines, as well as have two more projects to complete using Microsoft Technologies. One is with Xamarin mobile app for sensor readings, the other is with Power BI. 

##Conclusion##

The partnership between Costa Farms and Microsoft is strong and we have agreed to build "Proof of Concept" technology driven set of best practices for their new hydroponic crops, increasing remote monitoring, preventive maintenance and power of the Microsoft IoT Suite and the latest sensor hardware.

#Testimonials#
“I have been able to learn some new and interesting technologies. It has been an amazing experience thanks to the Microsoft team that has worked with us: Blain Barton, Joe Raio and David Crook.
The collaboration has been very good and it has been fun to build something from scratch that works and serves its purpose”
*

Rodney Sánchez
-*Software Developer*

“Through this POC I’ve learned about IoT solution architecture that gets deployed using Azure services. Besides the obvious technical learnings from this process, I have also developed a broad understanding on ways to evaluate and design IoT device requirements, infrastructure requirements and Structure engagements. This has been an incredible learning experience, thank you David, Blain and Joe for coming out here and helping us expand our horizons”
*

Mani Mihira Peddada
- *BI Architect*

##Resources##
Video link is located on Channel 9 at: [https://channel9.msdn.com/Blogs/raw-tech/Setting-up-the-Adafruit-Feather-M0-Wifi-with-a-pHSensor-running-in-Microsoft-Azure](https://channel9.msdn.com/Blogs/raw-tech/Setting-up-the-Adafruit-Feather-M0-Wifi-with-a-pHSensor-running-in-Microsoft-Azure) 
The 2 article links from Blain Barton's blog on the hardware and software setup can be found - [https://blogs.msdn.microsoft.com/blainbar/ 
](https://blogs.msdn.microsoft.com/blainbar/ )

