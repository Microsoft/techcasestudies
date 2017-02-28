---
layout: post
title:  "Future Standard changing image analytics with serverless platform"
author: "Masayuki Ota"
author-link: "https://twitter.com/masota0517"
#author-image: "{{ site.baseurl }}/images/authors/masota.jpg"
date:   2017-02-04
categories: [IoT]
color: "blue"
#image: "{{ site.baseurl }}/images/futurestandard/logo.png" #should be ~350px tall
excerpt: Future Standard offers a one-stop IoT solution for collecting, analyzing, viewing, and acting on data with ease. To offer customers an Azure-based solution, Future Standard and Microsoft created a prototype during a two-day hackfest.
language: English
verticals: [Manufacturing, "Retail, Consumer Products & Services", Infrastructure]
---


Future Standard offers a one-stop IoT solution for cross-industry uses to enable collecting, analyzing, viewing, and acting on data with ease. In order to offer customers an Azure-based solution, Future Standard teamed up with Microsoft to create a prototype during a two-day hackfest.

### Hackfest core team

- Hideyuki Suzuki – CTO, Future Standard
- Mikihisa Hayashi – COO, Future Standard
- Takashi Kaneda – CAO (Chief Analytics Officer), Future Standard
- Kousuke Ban – Development Manager, Future Standard
- Hiroshi Ota – Technical Evangelist, Microsoft
- Kazuyuki Nomura – Technical Evangelist, Microsoft
- [Masayuki Ota](https://twitter.com/masota0517) – Technical Evangelist, Microsoft


![hack image]({{ site.baseurl }}/images/futurestandard/hack1.jpg)
<!--
![test]({{site.baseurl}}/images/futurestandard/hack1.jpg)
-->

## Customer profile ##

[Future Standard Co., Ltd.](http://www.futurestandard.co.jp/about/) is a startup headquarted in Tokyo that provides a vision analysis platform called [SCORER](https://peraichi.com/landing_pages/view/scorer). Consumers and developers leverage it along with their own solutions in many environments. For example:

* Manufacturing – reading analog meters in factories.
* Retail – counting product views in stores.
* Infrastructure – monitoring and reporting on traffic.

![scorer]({{ site.baseurl }}/images/futurestandard/scorer.png)

Developers usually must rely on special hardware and analytics software when they want to perform image analytics. But with SCORER, they can easily prepare image analytics for any scenario. For greater ease, they can release OEM sensors. This is already in use in one of Tokyo’s largest department stores, Tokyu Hands. 

## Problem statement ##

Future Standard had made SCORER available on AWS, but many enterprise customers wanted to use it on Azure, primarily because of Azure’s high level of security and solid compliance standards.

To expand its market share, Future Standard needed to provide a solution that runs on Azure to meet these enterprise customers’ demands.

By providing solutions on Azure, Future Standard expects to gain 30,000 device sensor units within two years.
 
## Solution and steps ##

Microsoft and Future Standard agreed to a two-day hackfest to develop a prototype on Azure. The work would be divided as follows: 

- Before the hackfest:
  - Discuss objectives and focus points.
  - Short seminar on IoT Hub and Azure Functions.
  - Discuss architecture.
- The hackfest (hack, hack, and hack!).

![Discussion]({{ site.baseurl }}/images/futurestandard/discussion.png)


![Architecture]({{ site.baseurl }}/images/futurestandard/architecture.png)

For rapid development, we used PaaS and serverless services in Azure such as: 

- [Web Apps feature of Azure App Service](https://azure.microsoft.com/en-us/services/app-service/web/) 
- [Azure IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/) 
- [Azure Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) 
- [Azure DocumentDB](https://azure.microsoft.com/en-us/services/documentdb/) 
- [Azure Storage](https://azure.microsoft.com/en-us/services/storage/) 
- [Azure Functions](https://azure.microsoft.com/en-us/services/functions/) 
- [Azure Active Directory](https://azure.microsoft.com/en-us/services/active-directory/) 
- [Azure API Management](https://azure.microsoft.com/en-us/services/api-management/)

By the end of the two-day hackfest, we finished developing the prototype and posted the new release.

## Technical delivery ##

SCORER is using Azure in the following manner:

* Collect values from network cameras, smart phones, and other sensors with Raspberry Pi 3 using Raspbian.
* Send these values to Azure with Azure IoT Hub.
* Process these values with Stream Analytics and store them in DocumentDB.
* Upload camera data to Blob storage.
* Use Azure AD to secure web app used by end users.
* If users want to see logs and movies from the camera, Web Apps calls Functions to get the data.

As a result of the hackfest, we learned:

* Data flow with IoT Hub, Stream Analytics, and DocumentDB is very easy.
* Methods of checking data duplication in DocumentDB should be developed with your own code (not in Stream Analytics).
* Azure Functions is good to use, but we couldn't get data from DocumentDB with our own function. In the hackfest we wrote C# code for getting data from DocumentDB, but we expected to get data with an Azure function.

### Code for DocumentDB ###

As described above, we got data from DocumentDB with our code. In the hackfest we used C#, but Future Standard now uses Node.js because that is used by more developers at Future Standard.

```js
var TaskDao = require("./taskDao"); //DocumentDB wrapper

var config = {};
config.host = "YOUR_DOCUMENTDB_HOST_ADDRESS";
config.authKey = "YOUR_AUTH_KEY";
config.databaseId = "YOUR_DATABASE_ID";
config.collectionId = "YOUR_COLLECTION_ID";


var DocumentDBClient = require("documentdb").DocumentClient;
var docDbClient = new DocumentDBClient(config.host, {
    masterKey: config.authKey
});

var taskDao = new TaskDao(docDbClient, config.databaseId, config.collectionId);


new Promise((resolve, reject) => {
    taskDao.init((err) => {
        if (err) {
            reject(err);
            return;
        }
        resolve();
    });
})
.then((data) => {
    console.log("data: ");
    console.log(JSON.stringify(data));
    return new Promise((resolve, reject) => {

        //Set Queryspec for querying to DocumentDB
        var querySpec = {
            query: "SELECT TOP @limit * FROM root r WHERE r.TimeStamp BETWEEN @start_date AND @end_date",
            parameters: [{
                name: "@limit",
                value: 10
            },{
                name: "@start_date",
                value: "2016-10-15 00:00:00"
            },{
                name: "@end_date",
                value: "2016-10-20 00:00:00"
            }]
        };

        //Querying to DocumentDB with query set above.
        taskDao.find(querySpec, function (err, items) {
            if (err) {
                reject(err);
                return;
            }
            resolve(items);
        });

    });
})
.then((data) => {
    console.log("data :");
    console.log(JSON.stringify(data));
})
.catch((err) => {
    console.log("err :");
    console.log(JSON.stringify(err));
});
```

## Security ##

- For security with Blob storage and IoT Hub, we used shared access signatures (SAS) in each connection. By using SAS, we could limit the time range to make it more secure.
- IoT Hub allows only registered devices to communicate with it; each registered device gets its own “personal” connection string.
- For security with Web Apps, we used Authentication/Authorization with the Authentication function in Web Apps.

As described above, we used Blob SAS to securely get and send data to the blob. If you want to write programs with Node.js, you can do it this way and use it in Azure Functions.


```js
var azure = require("azure");

//Plese insert your credentials
var storageAccount = "your-storage-account";
var storageAccessKey = "your-storage-access-key";
var container = "your-container-name";

//Make blobService instance fo controll your storage.
var blobService = azure.createBlobService(storageAccount, storageAccessKey);

//The function for getting SAS.
function getSAS(context, blob) {
    var startDate = new Date();
    var expiryDate = new Date(startDate);
    expiryDate.setMinutes(startDate.getMinutes() + 5);//Set expire time 5 minutes later.

    context.log("blobService: " + JSON.stringify(blobService.host));
    context.log("startDate: " + startDate.toISOString());
    context.log("expiryDate: " + expiryDate.toISOString());

    var sharedAccessPolicy = {
        AccessPolicy: {
            Permissions: azure.Constants.BlobConstants.SharedAccessPermissions.READ,
            Start: startDate.toISOString(),
            Expiry: expiryDate.toISOString()
        }
    };
    context.log(JSON.stringify(sharedAccessPolicy));

    var signature = blobService.generateSharedAccessSignature(container, blob, sharedAccessPolicy);
    console.log("signature: " + signature);
    return signature;
}

//Main function triggered by HTTP in Azure Functions
module.exports = function(context, req) {
    context.log("Node.js HTTP trigger function processed a request. RequestUri=%s", req.originalUrl);

    var fileKey;

    if (req.query.fileKey|| (req.body && req.body.fileKey)) {
        fileKey = (req.query.fileKey || req.body.fileKey);
    }

    if (!fileKey) {
        context.res = {
            status: 400,
            body:  { result: "ng", error: "Please pass a fileKey on the query string or in the request body" }
        };
        context.done();
        return;
    }

    //Get SAS
    var sas = getSAS(context, fileKey);

    //Make URL  with SAS for handling blob storage.
    var url = blobService.getUrl(container,fileKey,sas);

    console.log(url);
    var responceBody = {
        url: url
    };

    context.res = {
        // status: 200, /* Defaults to 200 */
        body: responceBody
    };
    context.done();
};
```


## Conclusion ##

With the new Azure-based solution, Future Standard is positioned to expand its customer base. 

General lessons:

- PaaS and serverless services will accelerate development.
- The developers found it easy to use Azure during the hackfest.
- The combination of IoT Hub, Stream Analytics, and DocumentDB can be applied in a variety of other environments.

Opportunities going forward:

- We may use an Intel Stick PC for edge devices. This would bring computing capability to the edge devices.
- We may use Azure AD B2C for additional users. This would help provide a highly available, global, identity management service for consumer-facing applications that can scale to hundreds of millions of identities.

## Additional resources ##

- [News release by Future Standard](http://www.futurestandard.co.jp/news/161/) (Japanese)
- Kosuke Ban's tech document for [DocumentDB](http://qiita.com/Banchi0123/items/cc1abb74c396dd91b91f) (Japanese) 
- Kosuke Ban's tech document for [Storage Service](http://qiita.com/Banchi0123/items/e1f02104f1fea6ebe34e) (Japanese)
