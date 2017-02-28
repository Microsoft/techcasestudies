---
layout: post
title:  "Add title with customer name here"
author: "Add author name here"
author-link: "# Include twitter link here"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2016-05-19
categories: [IoT]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Add a short description of what this article is about, helping a fellow developer understand why they would want to read it. What value will they get out of reading it? Focus on the problem or technologies and let that be the guiding light.
language: The language of the article (e.g.: [English])
verticals: The vertical markets this article has focus on (e.g.: [Energy, Manufacturing & Resources, Financial Services, Public Sector, “Retail, Consumer Products & Services”, Environmental, Communications/Media, Transportation & Logistics, Smart Cities, Agricultural, Environmental, Healthcare, Other])
---

Begin with an intro statement with the following details:

- Solution overview
 
- Key technologies used
 
- Core Team: Names, roles and Twitter handles 


## Customer profile ##
This section will contain general information about the customer, including the following:

- Company name and URL

- Company description

- Company location

- What are their product/service offerings?



 
## Problem statement ##


This section will define the problem(s)/challenges that the customer wants to address with an IoT solution. Include things like costs, customer experience, etc.
 
*If you’d really like to make your write-up pop, include a customer quote that highlights the customer’s problem(s)/challenges. Attribute all quotes with Name, Title, Company.*


 
## Solution and steps ##


The majority of your win artifacts will be included in this section, including (but not limited to) the following: Pictures, drawings, architectural diagrams, value stream mappings and demo videos.

This section should include the following details:

- What was worked on and what problem it helped solve.

- Architecture diagram/s (**required**). Example below:

 ![IoT Architecture Diagram](/images/templates/iotarchitecture.png)

**Directions for adding images:**

1. Create a folder for your project images in the “images” folder in the GitHub repo files. This is where you will add all of the images associated with your write-up.
 
2. Add links to your images using the following absolute path:

  `![Description of the image]({{site.baseurl}}/images/projectname/myimage.png)`
    
  Here’s an example: 

  `![Value Stream Mapping]({{site.baseurl}}/images/orckestra/orckestra2.jpg)`

 Note that capitalization of the file name and the file extension must match exactly for the images to render properly.

*If you’d really like to make your write-up pop, include a customer quote that highlights the solution. Attribute all quotes with Name, Title, Company.*


## Technical delivery ##
This section will include the following details of how the solution was implemented:

- Security details

- Device used (be specific, details if PLC, microcontroller, etc.)
	- **IoT Device Information:**


		Customer had X design their devices for them, which were manufactured by Y.  The device has X MB/KB’s of memory, X GB/MB of storage and uses (line/battery) power and X for connectivity.  It is running (Firmware | RTOS | Linux | Windows IoT | Other).  They are using the (Azure IoT Device SDK | Azure IoT Gateway SDK) and writing their device app in (Node | Java | C# | C | Python)

- Device messages sent (packet size, frequency of send/day/device, number of messages)

- SDKs used, languages, etc.

- Code artifacts

- Pointers to references or documentation

- Learnings from the Microsoft team and the customer team


 
## Conclusion ##

This section will briefly summarize the technical story with the following details included:

- Measurable impact/benefits resulting from the implementation of the solution.

- General lessons:

  - Insights the team came away with.

  - What can be applied or reused for other environments or customers.

- Opportunities going forward:

  - Details on how the customer plans to proceed or what more they hope to accomplish.

*If you’d really like to make your write-up pop, include a customer quote highlighting impact, benefits, general lessons, and/or opportunities. Attribute all quotes with Name, Title, Company.*


## Additional resources ##
In this section, include a list of links to resources that complement your story, including (but not limited to) the following:

- Documentation

- Blog posts

- GitHub repos

- Etc…
