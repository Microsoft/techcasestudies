---
layout: post
title: "Improving scalability and customer analytics for the Tofugear retail platform"  
author: "William Dam"
author-link: "#"
#author-image: "{{site.baseurl}}/images/authors/photo.jpg"
date: 2017-02-09
categories: [IoT, Azure App Service, Azure Functions]
color: "blue"
#image: "{{ site.baseurl }}/images/TofugearImages/tofugear-logo.jpg" #should be ~350px tall
excerpt: Tofugear, an IoT startup, worked with Microsoft to re-architect its Omnitech retail platform solution, gaining both scalability and a dashboard for improved customer analytics.
language: English
verticals: "Retail, Consumer Products & Services"
---

<img alt="Tofugear logo" src="{{ site.baseurl }}/images/TofugearImages/tofugear-logo.jpg" width="221">

Tofugear wanted to offload processing from its API gateway and deliver better dashboard viewing to their clients through their Omnitech retail platform. Microsoft Azure IoT Hub, Azure Stream Analytics, Azure Functions, and Power BI offered that opportunity.

This is the team that was involved with the project:

- William Dam – Technical Evangelist, Microsoft
- William Yeung – Tofugear Chief Architect, Tofugear

## Customer profile ##

[Tofugear](http://www.tofugear.com/) is an Internet of things (IoT) startup based in Hong Kong that specializes in providing retail, ecommerce, business engineering, logistic and supply-chain solutions. Tofugear Omnitech is a fully customized, omnichannel retailing platform that offers new and exciting opportunities ranging from capturing increased sales across channels to enhanced brand awareness and loyalty, as well as gaining keen insight into customer "trying and buying" behavior.

## Problem statement ##

The Tofugear Omnitech solution collects both end-customer analytics and transactional information from mobile (Android and iOS) and web (client-side JavaScript) clients through an API gateway (built with Ruby on Rails) and then stores the data in PostgreSQL. Another Ruby worker process regularly pulls these data from PostgreSQL and uses Azure Machine Learning to drive various AI functions for customer service and business intelligence analysis.

Tofugear wanted to improve the scalability of its existing architecture, which is limited by the API gateway, and to create a dashboard solution based on Power BI to improve delivery of customer-data insights to their clients.

Key technologies:

- IoT: Azure IoT Hub, Azure Stream Analytics, Azure Blob storage
- Web service: Web Apps feature of Azure App Service, Azure Functions, Node.js, Ruby, JavaScript
- Device: mobile (iOS/Android)
- Dashboard: Power BI

![Whiteboard sketch of architecture]({{ site.baseurl }}/images/TofugearImages/Tofugear-WhiteBoard.jpg)

## Solutions, steps, and delivery ##

In our first meeting, we discussed the solution architecture and decided to offload all customer-analytics information (mobile and web client) from the API gateway, keeping only the transactional data flow on the gateway. We planned to stream the customer analytics and (in the next phase) the store sensors data to IoT Hub. Next, Azure Stream Analytics would aggregate the real-time streaming data and join it with the product-reference data from Blob storage (a snapshot from PostgreSQL), ready to output to Power BI.

To minimize code changes in the Ruby worker process that processes the customer-analytics data from PostgreSQL, we needed to allow the Ruby worker to pull the client data from IoT Hub instead of PostgreSQL.

### Data ingestion ###

To unify all web and mobile client connections to IoT Hub, we decided to use an HTTP REST interface. We spent some time to figure out how to use JavaScript to generate the SAS token, which then needs to be set in the HTTP authorization header to connect to IoT Hub. (See "Security tokens" in [Understand Azure IoT Hub security](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-security#security-tokens).)

*Generating the SAS token for IoT Hub registration*

```js
var hubName = "xxxxxxxxxxx";        
var myKey = "xxxxxxxxxxxxxxxxxxxxxxxxxxxx";
var host = (hubName + ".azure-devices.net").toLowerCase()
var hostUrl = (host + "/" + url).toLowerCase()
var endpointUri = "https://" + hostUrl

// Create a SAS token
var expiry = 1500000000
var sigSource = encodeURIComponent(host) + '\n' + expiry
var signature, hmac;
hmachash = CryptoJS.HmacSHA256(sigSource, CryptoJS.enc.Base64.parse(myKey))
signature = CryptoJS.enc.Base64.stringify(hmachash)
var sasToken = _.template("SharedAccessSignature sig=<%= sig %>&se=<%= se %>&skn=<%= skn %>&sr=<%= sr %>")
var sas = sasToken({sr: encodeURIComponent(host), sig: encodeURIComponent(signature), se: expiry, skn: 'iothubowner'})

return jQuery.ajax({
    url: endpointUri,
    method: method,
    data: JSON.stringify(data),
    dataType: 'json',
    beforeSend: function(xhr) {
        xhr.setRequestHeader("Authorization", sas);       
    }
})   
```

When we tested on a browser client, we discovered that IoT Hub does not support cross-origin resource sharing (CORS). (See [azure_iot_sdks issue #1001](https://github.com/Azure/azure-iot-sdks/issues/1001) on GitHub.) IoT Hub returns the following “not allowed access” error:

```
*jquery-3.1.1.min.js:4 OPTIONS <https://tofugeariothub.azure-devices.net/devices/webClient> send @ jquery-3.1.1.min.js:4ajax @ jquery-3.1.1.min.js:4xhrRequest @ test.js:32sendTemperature @ test.js:11onclick @ test.html:11*

*test.html:1 XMLHttpRequest cannot load <https://tofugeariothub.azure-devices.net/devices/webClient>. Response to preflight request doesn't pass access control check: No Access-Control-Allow-Origin' header is present on the requested resource. Origin '<http://localhost:3001>' is therefore not allowed access. The response had HTTP status code 405.*
```

![IoT Hub CORS with web client]({{ site.baseurl }}/images/TofugearImages/Tofugear-WebClientCORS.jpg)

So we looked into using Azure Functions as the proxy. We created two functions: a device-registration proxy for all clients (web and mobile) to avoid exposing the IoT Hub SAS token for device creation on the client side, and a send-message proxy to be used by the web client until IoT Hub supports CORS. Both iOS and Android use the same REST interface to send device messages directly to IoT Hub; we used Postman to test the interface.

*Sample of data ingestion from the web client*

```JSON
{
    "deviceId" : "webClient101",
    "deviceMessage" : {
        "event": "browse_product",
        "event_value": "https://shop.xxxxxxxxxx.co.xx/ja/shop/products/162-spangle-bag?color=379",
        "size_name": "FREE",
        "color_name": "BLACK",
        "product_id": 162,
        "referer_product_id": null
    },
    "deviceKey": "gtNQ6rNF7m+rurHHh27w+i3D6NSEdCgUvfSVJnOnKys="
}
```

![Architecture to include Azure Functions to work around CORS issue]({{ site.baseurl }}/images/TofugearImages/Tofugear-withWebClientProxyAzFnArch.jpg)

The following Azure Functions code is used for IoT Hub device registration for all clients:

```js
var connectionString = `HostName=${process.env.IOTHUB_HOSTNAME};SharedAccessKeyName=iothubowner;SharedAccessKey=${process.env.IOTHUBOWNER_SHAREDACCESSKEY}`
var registry = iothub.Registry.fromConnectionString(connectionString)
// var device = new iothub.device(null) //remove this code as its seems the new npm package keep complain that device is not constructor
// replace with the following 
var device = {
    deviceId: null
};
device.deviceId = req.body.deviceId
registry.create(device, function (err, deviceInfo, res) {
    context.log('IoTHub connected......');
    if (err) {
        registry.get(device.deviceId, function(err, deviceInfo, res) { 
            context.res = {
                status: 500,
                body: JSON.stringify({
                    "status": 500,
                    "error": 'unable to create device',
                    "deviceInfo": deviceInfo 
                })2016-12-10-TofugearIotAzFn.md
            }              
            context.done()          
        });                     
    } else if (deviceInfo) {
        context.res = {
            status: 201,
            body: JSON.stringify({ "deviceInfo": deviceInfo })
        }
        context.log('Device registered......');
        context.done();
    }
})
```

The following code helps web clients bypass the CORS issue:

```js
var connectionString = `HostName=${process.env.IOTHUB_HOSTNAME};DeviceId=${req.body.deviceId};SharedAccessKey=${req.body.deviceKey}`  
var client = clientFromConnectionString(connectionString);
var messageSent = false;
var connectCallback = function (err) {
    if (err) {
        context.log('Could not connect: ' + err);
    } else {
        context.log('IotHub connected');
        // Create a message and send it to the IoT Hub
        var msg = new Message(JSON.stringify({ deviceId: req.body.deviceId, Data: req.body.deviceMessage }));
        context.log('Message sending.....');
        client.sendEvent(msg, function (err) {
            if (err) {
                console.log(err.toString());
            } else {
                context.log('Message sent');
                messageSent = true;
                context.res = {
                    status: 201,
                    body: JSON.stringify({Data: req.body.deviceMessage + ' from ' + req.body.deviceId + ' sent successfully'})
                }
                context.done();
            };
        });
    }
};
client.open(connectCallback);
```

### Data processing ###

We then used Stream Analytics to combine the data from IoT Hub and the product-reference data from Blob storage for richer Power BI output.

![Configuring Stream Analytics]({{ site.baseurl }}/images/TofugearImages/Tofugear-StreamAnalytic.jpg)

*Sample of Stream Analytics combining the client data and product reference to output to Power BI*

```sql
SELECT
    skus.id as sku_id,
    metrics.Data.event,
    metrics.Data.event_value,
    metrics.Data.size_name,
    metrics.Data.color_name,
    metrics.Data.user_id,
    skus.price, 
    skus.product_state,	
    skus.product_code,
    skus.can_purchase
INTO
    Tofugearpowerbidataset
FROM
    [TofugearIoTHubInput] metrics 
LEFT JOIN [Tofugear-Ref-Data] skus on metrics.Data.product_id = skus.product_id
```

Because we like to separate the IoT Hub consumer group for Azure Stream Analytics and the Ruby worker, we created two consumer groups: one consumed by Stream Analytics to output to Power BI, and another used by the existing Ruby worker to pull the web and mobile client telemetry analytic data for processing.

We spent quite some time hacking the [Apache Qpid Proton package](http://qpid.apache.org/proton/index.html) because there’s no IoT Hub SDK support for Ruby code that requires AMQP 1.0. We had no success after a few tries, and it takes too much effort if we consider using Ruby inline to wrap the [Azure IoT C library](https://github.com/Azure/azure-iot-sdk-c). To bypass the AMQP connection challenge, we decide to create another Azure function to allow the existing Ruby worker to use HTTP on demand to pull the IoT Hub (receiver side) customer-analytics data for processing instead of direct-streaming the IoT Hub data to the Ruby worker, which wouldn't be able to handle the capacity.

![Architecture to add another function to allow Ruby worker to pull from IoT Hub]({{ site.baseurl }}/images/TofugearImages/Tofugear-withRubyWorkerProxyArch.jpg)

This is the Azure Functions code that enables the Ruby worker to pull the message from IoT Hub:

```js
var connectionString = `HostName=${process.env.IOTHUB_HOSTNAME};SharedAccessKeyName=iothubowner;SharedAccessKey=${process.env.IOTHUBOWNER_SHAREDACCESSKEY}`
var printError = function (err) {
    context.log(`error occurred: ${err.message}`);
};
var messageList = []
var messageBodyList = []
var appendMessageToList = function (message) {
    messageBodyList.push({
        offset: message.offset,
        sequenceNumber: message.sequenceNumber,
        enqueuedTimeUtc: message.enqueuedTimeUtc,
        body: message.body
    })
    messageList.push(message)
    context.log(message)
    return true;
}
var client = EventHubClient.fromConnectionString(connectionString);
var closeClientAndCompleteContext = function() {
    client.close();
    context.done();
}
client.open()
    .then(client.getPartitionIds.bind(client))
    .then(function (partitionIds) {
        context.log('Connected to IoT Hub.....');
        setTimeout(function() {
            context.res = {
                status: 201,
                body: JSON.stringify({'messages': messageBodyList})
            }
            closeClientAndCompleteContext()
        }, 5000)

    return partitionIds.map(function (partitionId) {
        context.log('Retrieving data from queue.....');
        return client.createReceiver(process.env.MESSAGE_POLL_CONSUMERGROUP, partitionId, { 'startAfterOffset': (req.query.after_offset || 0) }).then(function(receiver) {
            context.log(`connected. PartitionId: ${partitionId}`)                
            receiver.on('errorReceived', printError);
            receiver.on('message', appendMessageToList);
            });
        });
    })
    .catch(printError);
```

We observed a couple of unexpected behaviors with IoT Hub on the receiver side. The first unexpected issue is that the free version of IoT Hub supports only the $Default consumer group, but the paid version is fine.

Because we’re using the offset to pull the data from the IoT Hub queue, we also found that the EventHub createReceiver function call returns an invalid response (shown in the following) for partitions that don’t contain the specific offset. However, the call still returns successfully; the error message is just a bit annoying.

```
2016-11-01T04:26:55.136 The supplied offset '336' is invalid. The last
offset in the system is '-1'
TrackingId:8c2c5345-efe2-4cf9-8952-d5ea4a62dd70\_B2,
SystemTracker:iothub-ns-tofugeario-73126-2c3dc3dc23:eventhub:tofugeariothub\~24575,
Timestamp:11/1/2016 4:26:55 AM
Reference:f1637dbe-9ec9-462b-b73e-e2c1a06488bc,
TrackingId:25df2420-84d1-47e9-a0e3-f2a4beaee67c\_B2,
SystemTracker:iothub-ns-tofugeario-73126-2c3dc3dc23:eventhub:tofugeariothub\~24575|streamanalytic,
Timestamp:11/1/2016 4:26:55 AM
TrackingId:c72526651ba4472dbb3bdb9a7fc3821a\_G0, SystemTracker:gateway2,
Timestamp:11/1/2016 4:26:55 AM.............................
2016-11-01T04:26:59.230 Function completed (Success,
Id=7a4e2e9d-5902-449c-ba4f-02c349994f0c)
```

### Performance tuning ###

We’ve established the end-to-end flow, so the next thing that we’re looking into is to improve the startup time for Azure Functions because it exhibits some sort of cold-start symptoms and we use a dynamic plan for cost savings. We decided to set up another timer-triggered KeepAlive HTTP ping function to keep these three Azure functions warm. We also moved all the npm package loading outside the function call to avoid unnecessary package loading as long as the functions are warm.

![Final architecture, including a timer-triggered function to keep all Azure functions warm]({{ site.baseurl }}/images/TofugearImages/Tofugear-FinalArch.jpg)

And this is the timer-trigger code that pings other Azure functions to keep them warm:

```js
context.log("Timer triggered at " + myTimer.next);
var pingPaths = [
    '/api/devices?action=ping&code=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    '/api/messages?action=ping&code=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx',
    '/api/message_feed?action=ping&code=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
]
context.log("timer passed?",myTimer.isPastDue)
if(myTimer.isPastDue)
{
    context.log('Node.js is running late!');     
} else {
    pingPaths.map(function (path)  {
        var url = `https://${process.env.AF_HOST}${path}`
        context.log(`ping url: ${url}`)
        var req = https.get(url)
        req.end()
    })
    context.log("Timer triggered at " + myTimer.next);
};
context.done();
```

We observed quite good response times, with averages of 400ms (or sometimes less) except during the initial start of the Azure functions. Then we start noticing some long startups--occasionally up to minutes. After some investigation, we suspect some combination of the IoT Hub connection setup and Azure Functions environment might contribute this unexpected result, which we’re working closely with the product team to investigate. It’s important to get this performance issue resolved before moving to a production environment.

![Response time for the message proxy, excluding the error cases]({{ site.baseurl }}/images/TofugearImages/AzFnPerformance.jpg)

## Conclusion ##

This combined effort from Microsoft and Tofugear delivered the proof of concept (POC) that demonstrated how IoT Suite and Azure Functions can provide easy scaling and integration with almost real-time visualization of combined data in Power BI. The project was implemented in a bit over four weeks with only one technical evangelist and one partner developer and could've been shorter if not for some unexpected technical challenges.

We accomplished our goal of making a scalable and better visualization product integration with this POC. Tofugear is willing to work with the product team to help us resolve the performance issue. We aim to bring this POC into production as soon as this issue is resolved.

Tofugear is setting high expectations and committing to work with us towards a production launch of the product. Although many of our frameworks and solutions solve a business or technical problem a customer has, we appreciate the resources and bandwidth used by the customer to maintain, debug, or troubleshoot what we put together.

![Tofugear team]({{ site.baseurl }}/images/TofugearImages/Tofugear-team.jpg)

Here's what Tofugear said about this project:

“Our partnership with Microsoft on this Tofugear Omnitech project has bring us close relationship to work as partner. This new architecture significantly changes the way tracking end customer analytic data in communicate with a central system that receives and stores data, while also allowing to visualize these close to real time data in a meaningful way. It brings performance and cost benefits and will definitively leverage our sales in this segment. This is what the market needs: solutions that add value while at the same time reducing the complexity of the integration to our platform would let us more focusing to deliver more customer value and feature delivery.”
