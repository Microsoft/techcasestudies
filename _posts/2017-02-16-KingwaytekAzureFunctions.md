---
layout: post
title: "Using Azure Functions to enable real-time notifications on demand"
author: "Eric ShangKuan"
author-link: "https://github.com/ericsk"
#author-image: "{{ site.baseurl }}/images/authors/ericsk.jpg"
date: 2017-03-21
categories: [Azure App Service, Azure Functions]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: "KingwayTek, the top navigation and map-service provider in Taiwan, is developing a personalized driving-experiences system. With Microsoft, they identified the appropriate Azure services to enhance data flow."
language: English
verticals: [Transportation & Logistics]
---

With the popularity of 4G/LTE technology, getting data from connected things becomes easier and more efficient. As we collect huge amounts of data, we not only gain more insights but also help make future decisions on services and development. KingwayTek was looking for the appropriate platform to build a connected-vehicle system to provide more personal driving experiences to the public.

KingwayTek selected Microsoft as their partner to build the system. We organized a taskforce to initiate and enable the project. The following are members of the core team:

- Knight Tseng – Techical Manager, KingwayTek
- Eric ShangKuan – Sr. Technical Evangelist, Microsoft Taiwan

## Customer profile ##

[KingwayTek Technology Co., Ltd.](http://www.kingwaytek.com/), was founded in late December 2007. The company began by producing 1:5000 digital maps (1:10000 in mountain areas); KingwayTek now develops a series of digital map database products. It owns the majority of market share and is the biggest map provider in Taiwan.

Based on their existing digital maps, Kingwaytek extended their business into navigation software in 2010. They launched a product named [Naviking (導航王)](https://naviking.localking.com.tw/), which offers several location-based web services (such as weather, car-park locations, and movie info) and is bundled with all cell phones sold by Chunghwa Telcom. NaviKing became well-known, successfully bringing users multiple new and useful online services in the 3G era.

In the future, besides providing its existing digital products, KingwayTek plans to offer more high-quality products to customers by accumulating resources to develop professional map information, digital content, software tool development, and integrated software techniques.

## Problem statement ##

In KingwayTek's recently-launched product, [Autoking (樂客車聯網)](http://www.autoking.com.tw/), a personal driving-experience service, plenty of data is sent from the user's vehicle to the cloud. KingwayTech wanted to enable the real-time data-processing unit to proactively raise an incident that can trigger vehicle reconfiguration (such collecting more data or changing the data-transmission frequency).

## Solutions and architecture ##

The engineers in KingwayTek have adopted Azure IoT Hub for data ingestion and Azure Stream Analytics for real-time processing. As their Azure partner on the Microsoft side, I proposed to use Azure Functions as the "glue" between IoT Hub and Stream Analytics. Then we facilitated a hackfest to adopt Azure Functions app.

*Architecture discussion; from left, Eric (Technical Evangelist, Microsoft) and Knight (Technical Manager, KingwayTek)*

<img alt="Photo of architecture discussion at whiteboard" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/discussion.jpg" width="600">

Here is how Azure Functions works in their current architecture:

<img alt="Diagram showing the use of Azure Functions to send command messages to vehicles" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwfuncapp.png" width="688">

1. When Stream Analytics detects an abnormal status in the data, it sends a message to a Service Bus topic, which triggers an Azure Functions app.

2. The Azure Functions app uses an Azure IoT Hub device SDK to send a reconfiguration command message to IoT Hub, which sends the command message to the specified vehicle.

## Code artifacts ##

In this hackfest, there are two main parts in this solution.

- How Stream Analytics sends a message to a Service Bus topic
- How an Azure Functions app triggered by the Service Bus topic uses the Azure IoT device SDK to send a message to a vehicle

### Send message from Stream Analytics to Service Bus topic ###

First, create a Service Bus instance on Azure and add a topic in it.

<img alt="Adding a topic to a Service Bus instance" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwsbcreation.png" width="768">

Open the topic and add two policies: a _Send_ policy for Stream Analytics message-sending and a _Listen_ policy for the Azure Function app.

<img alt="Adding two policies to the topic" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwsbtopicpolicy.png" width="640">

After creating the Service Bus instance, go to the Stream Analytics management blade. Add an output alias that specifies the Service Bus instance. Check the connected Service Bus and the policy carefully.

<img alt="Adding an output alias in Stream Analytics" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwasasb.png" width="327">

Now we can use the following query to send data to the Service Bus topic.

```sql
SELECT (status_data) INTO StatusTopic FROM TheHub
```

### Azure Functions app development ###

In Azure Functions, we can add a function that is triggered by the Service Bus topic. Select **Service Bus** as the trigger and add appropriate parameters. The connection string for Service Bus can be found in the Service Bus topic policy. In this case, we have to use the Listen policy.

<img alt="Setting trigger for Azure Functions app" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwfuncintegrate.png" width="720">

We also need to add the IoT Hub device SDK to this app. In App Service Editor, open the **Function app settings** panel. Add a file named _project.json_ to the app directory to make Azure Functions restore the specified packages.


<img alt="Adding project.json to restore packages" src="{{ site.baseurl }}/images/KingwayTekAzureFunctions/kwfuncpkg.png" width="600">

The content of project.json is as follows:

```json
{
  "frameworks": {
    "net46":{
      "dependencies": {
        "Microsoft.Azure.Devices": "1.2.3"
      }
    }
  }
}
```

After you save the file, Azure Functions will automatically restore the Azure IoT Hub device SDK packages (and its dependencies).

Finally, we can use the following code in _run.csx_ to extract the message from the Service Bus topic and compose a command message send to IoT Hub.

```csharp
using System.Text;
using Microsoft.Azure.Devices;
using Newtonsoft.Json;

public class VehicleStatus
{
    public string VehicleId;
    public string Status;
}

public static void Run(string mySbMsg, TraceWriter log)
{
    VehicleStatus status = JsonConvert.DeserializeObject<VehicleStatus>(mySbMsg);

    // create IoT Hub connection.
    ServiceClient serviceClient = ServiceClient.CreateFromConnectionString("{IOTHUB_CONNECTIONSTRING}");

    // Composing the command message
    // reconfiguration command and set the frequency to 30,000 ms.
    var commandMessage = new Message(Encoding.ASCII.GetBytes("RECONF 30000"));
    // send command to specified device.
    serviceClient.SendAsync(status.VehicleId, commandMessage);
}
```

## Conclusion ##

Before this hackfest, engineers at KingwayTek planned to use virtual machines (VMs) or WebJobs in the Web Apps feature of Azure Web Services to create a scheduled job to accomplish the notification feature. However, the VM solution increases the maintenance cost (including human resources), and scalibility is hard to manage. Using WebJobs is not a good fit for this architecture because the app on WebJobs can be triggered only by Azure Queue storage or HTTP-based requests. Azure Functions is perfect for this situation because it gives the development team the flexibility to select the trigger, and the consumption-based pricing and scalibility also helps them manage the cost and performance. Finally, we decide to use a Service Bus topic as the trigger because it can not only trigger the Azure Functions app but also route messages to potential subscribers.

After the hackfest, the KingwayTek engineering team will integrate and test the feature in their current product; they plan to launch this feature in 2017 Q2. In the next step, KingwayTek wants to incorporate Azure Functions in every project that contains scheduled tasks. The (auto) scalibility and serverless features are the key reasons for this decision. 
