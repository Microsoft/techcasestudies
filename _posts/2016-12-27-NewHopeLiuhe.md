---
layout: post
title:  "Developing the next generation of smart monitoring solutions for chicken farming"
author: "Qixiao Wang"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-02-17
categories: [IoT]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: New Hope Liuhe, a leading Chinese agricultural company, reached out to Ye Fan Technology and Microsoft for an IoT solution that could provide automatic, real-time remote monitoring and environmental control of chicken farming facilities.  
language: [English]
verticals: [Agricultural]
---

Microsoft joined up with Ye Fan Technology, a startup company that focuses on industrial IoT system integration and development, to help Microsoft customer New Hope Liuhe design and develop its next-generation smart-farming IoT solution based on Windows 10 IoT Core and Azure.

**The project team:** 

- Xia ZanHui – GM, New Hope Liuhe
- Liu HongFeng – CEO, Ye Fan Technology
- Zhang Xianbo – System Integration Lead, Ye Fan Technology
- Michael Sh Chi – SDE2, TED, Microsoft 
- Qixiao Wang – Principal Technical Evangelist, Microsoft
- Yan wei – Technical Evangelist, Microsoft

## Customer profile ##

[New Hope Liuhe](http://www.newhopeliuhe.com/) is a Chinese company founded in 1998 that focuses on agriculture and animal husbandry products. This includes animal feed, breeding, and meat products. The company's business covers all of China and has branches in Vietnam, the Philippines, Bangladesh, Indonesia, Sri Lanka, Kampuchea, Singapore, and Egypt. By the end of 2013, the company controlled more than 500 subsidiaries and branches, with total assets exceeding $20 billion and some 80,000 employees.

## Partner profile ##

[Ye Fan Technology](http://www.sky-walker.com.cn/) was established by former Microsoft engineer Hongfeng Liu, who has worked in the field of industrial automation systems for the past decade. During his career at Microsoft, he developed the first version of FAT32 for .NET Micro Framework. He is also the first person to migrate .NET Micro Framework onto ARM Cortex-M3 worldwide. Hongfeng Liu set up Ye Fan to provide an integration solution including hardware and software for the industrial IoT application.
 
## Pain point ##

The chicken farming industry is a big part of New Hope Liuhe's business. One key priority is to help farmers accurately understand and control the feed, water, and environmental data such as temperature and air quality to maximize the return on investment. New Hope had tried a mobile app solution that relied on manual input from farmers' smart phones, but the results were very poor because the farmers were too busy to input the data in real time. New Hope reached out to Ye Fan and Microsoft to ask for an IoT solution that could satisfy the need for automatic, real-time remote monitoring and environmental control. One typical scenario could be: 

> Ensuring the air quality within a chicken farming facility. It's very important to monitor the CO2 level within a facility. If the CO2 level surpasses a certain threshold, the chickens can die.
> 
> A CO2 sensor will collect the air quality data and send it through a local gateway onto the cloud side. Then, with the preconditioned logic, if the data exceeds a certain threshold, a notification is sent to the farmer’s smart phone and the famer will use a mobile app to control the speed/status of the fan to adjust the air condition. 

The main goal of this project is to incubate and pilot the IoT solution on some select farms. After 2-3 months of reliability testing at the current farm, the solution will be deployed to another five target farms. After the reliability testing is completed at those farms with acceptable results, then both New Hope and Ye Fan plan a more scalable deployment.
 
## The solution ##

A Windows device running Window 10 IoT Core acting as the local gateway will collect and aggregate all data from different sensors and send the data to an Azure IoT Hub. A UWP application is deployed onto the gateway for the data communication. The hardware is the gateway product from NexCom, which already passed Microsoft IoT certification. 

- [Microsoft IoT certification](https://catalog.azureiotsuite.com/details?title=NISE-50C&source=home-page) 
- [NexCom hardware](http://www.nexcom.co.uk/Products/industrial-computing-solutions/industrial-fanless-computer/atom-compact/fanless-computer-nise-50c) 

During the early design stage, several key requirements and constraints make Windows 10 IoT Core a good choice:

- In development, the field gateway device also will need to be connected to an external monitor for visualization purposes. The UWP that communicates with the IoT Hub could also achieve this part of the requirements.
- Partner skillset readiness. The Ye Fan team has very strong C# skills around .NET Micro Framework, so the UWP platform is a natural fit. The C# UWP SDK makes it very easy to kick off the development of the app.

On the cloud side, Azure IoT Hub will collect the sensor data and, through Azure Stream Analytics, the data will be archived into Blob storage for backup purposes. Also, an Azure WebJob will directly collect the sensor data from IoT Hub and forward it onto the external API gateway, which will be consumed by the mobile app part. All the location and project information of different deployments is saved in an Azure SQL database.

Technologies used:
    
- Windows 10 IoT 
- Azure IoT Hub
- Azure Stream Analytics
- Azure Service Bus Queue
- Azure Storage 
- Azure SQL Database
- Azure App Service (WebJob)

The sensors and devices that need to be controlled, such as fans and lights, are connected to the NexCom device. Through the aggregation, all the sensor data and device status are sent to the IoT Hub through the UWP running on the gateway. Certain parts of the back end, like the back-end system of the mobile app, are still running on the third-party public cloud, so we need to host an external API gateway for the data exchange and pull back the user command.

A key reason why both New Hope and Ye Fan agreed to try Microsoft Azure for this pilot project was its full feature spectrum to support the IoT application, from data ingestion, storage, and real-time analytics to visualization. At the connection part, after comparing the differences between Event Hub and IoT Hub, we chose IoT Hub for two reasons:

- Bi-directional communication. It is not just about receiving the data from the sensor/device, but enabling the farmer to control the status of the device, such as the fan or the light in the facility.
- Device identity and management. After the solution is scaled out to more and more farms, the solution needs to be able to identify the different device information from different farms.

## Architecture ##

The architecture of the solution is described here.

- On the device side, the NexCom gateway to be connected to the sensor hub through a COM port. This sensor hub directly integrates different sensors on board or connects to the sensor through RS0485 protocol.
- On the cloud side, Stream Analytics will transfer the message data into the Storage blob for archiving.
- Multiple different WebJobs are running to: 
  - Forward the incoming messages onto the remote API server. 
  - Regularly pull the updates from the external API server and send the update commands onto the remote sensors if needed.

The message is a JSON-based format. Below is one example of the message being used.

`{"ProjectId":"23","Cmd":"SetRly","DQ8Q1":"1","DQ8Q2":"1","DQ8Q3":"1","YFSENH3":"25"}`

- **ProjectId:** the farm from which the data is coming
- **Cmd:** the command being sent
- **DQ8Q1:** sensor type
- **DQ8Q2:** sensor type 
- **DQ8Q3:** sensor type
- **YFSENH3:** sensor type

Because data security is a key principal of this solution, the team set up several simple rules to follow from the very beginning:

- NexCom devices were selected not only for technical reasons (the OS, development language), but also because they passed the Microsoft IoT certification. This certification indicates that secure communication is supported.
- Select the secure communication protocol. Although HTTP is chosen at this stage, AMQP is the direction. We will update the code when the issue of AMQP support of UWP is addressed.
- Use the correct permissions to access the IoT Hub. Only assign the DeviceConnect permissions to the UWP on the gateway device.
- Secure the communication between the WebJob and the external API gateway by using HTTPS.

*Overall system architecture*

![NewHopeLiuHe Smart Chicken Farm solution architecture]({{ site.baseurl }}/images/NewhopeLiuhe/Architecture.png)


## Code artifacts ##

### Device part ###

On the Windows 10 gateway, a UWP application is deployed to:

- Communicate with the sensor hub to revive data.
- Pack the data into the JSON stream.
- Set up the connection with IoT Hub to send the JSON messages.
- Receive the update commands from IoT Hub and forward them to the sensor hub.

Here is the code for sending data to the IoT Hub:

```csharp
        static string IOTHUB_RUI = "**********.azure-devices.net";
        private const string DeviceConnectionString ="HostName=********.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=************************************";

        DeviceClient _deviceClient = null;
        YFSensorModel YFSensorModelRTU = null;
        YFDI8Model yfDI8Model = null;
        YFDQ8Model yfDQ8Model = null;
        Standardel_SDT870 SDT870RTU = null;
        JBSB_LXSGZ20 LXSGZ20 = null;

        public readonly BlockingCollection<JObject> _blockQueue = new BlockingCollection<JObject>();

        public const string DEVICEID = "Device006";
        public const string PROJECTID = "23";

        public MainPage()
        {
            this.InitializeComponent();
            try
            {
                _deviceClient = DeviceClient.Create(IOTHUB_RUI, new DeviceAuthenticationWithRegistrySymmetricKey("Device006", "f04NqhzRUXYf8MDkem642M22WSN7MbEmK9Ozj2HIOYk="), TransportType.Amqp);
            }
            catch { }

            Task.Run(async () =>
               {
                   JObject rmtd = null;
                   string msgString = null;
                   Message msg2 = null;
                   while (true)
                   {
                       rmtd = new JObject();
                       if (!_blockQueue.TryTake(out rmtd, 3000))
                       {
                           rmtd = new JObject();
                           rmtd.AddFirst(new JProperty("ProjectId", PROJECTID));
                           rmtd.AddFirst(new JProperty("DeviceId", DEVICEID));
                       }
                       if (rmtd != null)
                       {
                           Debug.WriteLine("get data from  queue \r\n");
                           try
                           {
                               msgString = JsonConvert.SerializeObject(rmtd);
                               msg2 = new Message(Encoding.ASCII.GetBytes(msgString));

                               await _deviceClient.SendEventAsync(msg2);
                               Debug.WriteLine("\t{0}> Sending message: {1}, Data: [{2}]", DateTime.Now.ToLocalTime(), 0, msgString);
                           }
                           catch (Exception e) { 
                                LoggerHelper.Metric($"{source}-Error", 1);
                                LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
                           }
                       }
                   }
               });
        }
```

For each supported sensor, there is a special data class used to pack/unpack the in-memory data transferred from the sensor through the serial port. Below is one of the sensor data objects.

```csharp

public class YFDI8Model : ModelBase
    {
        private YFIOCustomProtocol protocol = null;

        private UInt16 deviceAddr = 1;

        public string this[int index]
        {
            set
            {
                switch (index)
                {
                    case 0:
                        DI8I1 = value;
                        break;
                    case 1:
                        DI8I2 = value;
                        break;
                    case 2:
                        DI8I3 = value;
                        break;
                    case 3:
                        DI8I4 = value;
                        break;
                    case 4:
                        DI8I5 = value;
                        break;
                    case 5:
                        DI8I6 = value;
                        break;
                    case 6:
                        DI8I7 = value;
                        break;
                    case 7:
                        DI8I8 = value;
                        break;
                    default:
                        throw new ArgumentOutOfRangeException("index");
                }
            }
            get
            {
                switch (index)
                {
                    case 0: return DI8I1;
                    case 1: return DI8I2;
                    case 2: return DI8I3;
                    case 3: return DI8I4;
                    case 4: return DI8I5;
                    case 5: return DI8I6;
                    case 6: return DI8I7;
                    case 7: return DI8I8;
                    default:
                        throw new ArgumentOutOfRangeException("index");
                }
            }
        }
        public string DI8CommState { get; set; } = "0";
        public string DI8I1 { get; set; } = "0";
        public string DI8I2 { get; set; } = "0";
        public string DI8I3 { get; set; } = "0";
        public string DI8I4 { get; set; } = "0";
        public string DI8I5 { get; set; } = "0";
        public string DI8I6 { get; set; } = "0";
        public string DI8I7 { get; set; } = "0";
        public string DI8I8 { get; set; } = "0";

        override protected JObject WriteAndRead()
        {

            JObject rmtd = new JObject();
            try
            {
                byte[] buffer = Block_Read(0, 33);
                rmtd = ParseData(buffer[0]);
                rmtd.AddFirst(new JProperty("DI8CommState", "0"));
            }
            catch (Exception e) { 
                                LoggerHelper.Metric($"{source}-Error", 1);
                                LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
            }

            return rmtd;
        }

        private JObject ParseData(byte buffer)
        {
            JObject changedDataProps = new JObject();
            string res;

            for (int i = 0; i < 8; i++)
            {
                res= ((buffer >> i & 1) > 0) ? "1" : "0";

                if (!this[i].Equals(res))
                {
                    this[i] = res;
                    changedDataProps.Add("DI8I"+(i+1), res);
                }
            }
            return changedDataProps;
        }

        /// <summary>
        /// 构造函数
        /// </summary>
        /// <param name="modbus">具体的模块类</param>
        public YFDI8Model(YFIOCustomProtocol protocol, UInt16 addr = 1)
        {
            this.deviceAddr = addr;
            this.protocol = protocol;
        }

        /// <summary>
        /// 开关量输入状态读取
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <returns>True - 高电平 Flash - 低电平</returns>
        public bool I_Read(int addr)
        {
            return protocol.Read(addr, 1, IOType.I, deviceAddr)[0] == 1;
        }

        /// <summary>
        /// 开关量输出状态读取
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <returns>True - 通 Flash - 断开</returns>
        public bool Q_Read(int addr)
        {
            return protocol.Read(addr, 1, IOType.Q, deviceAddr)[0] == 1;
        }

        /// <summary>
        /// 写开关量输出状态
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <param name="state">True - 通 Flash - 断开</param>
        public void Q_Write(int addr, bool state)
        {
            protocol.Write(addr, new byte[] { (byte)(state ? 1 : 0) }, 0, 1, IOType.Q, deviceAddr);
        }

        /// <summary>
        /// 模拟量读取
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <returns>模拟量的值</returns>
        public float A_Read(int addr)
        {
            byte[] buffer = protocol.Read(addr, 4, IOType.A, deviceAddr);
            return BitConverter.ToSingle(buffer, 0);
        }

        /// <summary>
        /// 模拟量写
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <param name="state">模拟量的值</param>
        public void O_Write(int addr, float state)
        {
            byte[] buffer = BitConverter.GetBytes(state);
            protocol.Write(addr, buffer, 0, 4, IOType.O, deviceAddr);
        }

        /// <summary>
        /// 数据块读取
        /// </summary>
        /// <param name="addr">地址 0,1,2...</param>
        /// <param name="count">数据个数</param>
        /// <returns>数据内容</returns>
        public byte[] Block_Read(int addr, int count)
        {
            return protocol.Read(addr, count, IOType.Block, deviceAddr);
        }

        /// <summary>
        /// 数据块写
        /// </summary>
        /// <param name="addr"></param>
        /// <param name="buffer"></param>
        /// <param name="offset"></param>
        /// <param name="count"></param>
        public void Block_Write(int addr, byte[] buffer, int offset, int count)
        {
            protocol.Write(addr, buffer, offset, count, IOType.Block, deviceAddr);
        }
    }
}

```

The code below shows the UWP code getting the command update from the IoT Hub:

```csharp

 async Task ReceiveCommands(DeviceClient deviceClient)
        {
            Message receivedMessage = null; ;
            string messageData;
            JObject jsonMmessageData = null;
            string msgString = null;
            Message msg2 = null;

            while (true)
            {
                try
                {
                    receivedMessage = await deviceClient.ReceiveAsync();
                }
                catch (Exception e) 
                { 
                   LoggerHelper.Metric($"{source}-Error", 1);
                   LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
                }
                if (receivedMessage != null)
                {
                    try
                    {
                        await deviceClient.CompleteAsync(receivedMessage);
                    }
                    catch (Exception e) 
                    { 
                       LoggerHelper.Metric($"{source}-Error", 1);
                       LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
                    }

                    messageData = Encoding.ASCII.GetString(receivedMessage.GetBytes());
                    Debug.WriteLine("\t{0}> Received message: {1}", DateTime.Now.ToLocalTime(), messageData);

                    try
                    {
                        jsonMmessageData = JObject.Parse(messageData);

                        if (null == jsonMmessageData) continue;

                        if ("GetAllState".Equals((string)jsonMmessageData["Cmd"]))
                        {
                            if (YFSensorModelRTU != null)
                            {
                                msgString = JsonConvert.SerializeObject(YFSensorModelRTU);
                                msg2 = new Message(Encoding.ASCII.GetBytes(msgString));
                                try
                                {
                                    await _deviceClient.SendEventAsync(msg2);
                                }
                                catch (Exception e) 
                                { 
                                      LoggerHelper.Metric($"{source}-Error", 1);
                                      LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
                                }
                            }

                            if (SDT870RTU != null)
                            {
                                msgString = JsonConvert.SerializeObject(SDT870RTU);
                                msg2 = new Message(Encoding.ASCII.GetBytes(msgString));
                                await _deviceClient.SendEventAsync(msg2);
                            }

                            if (LXSGZ20 != null)
                            {
                                msgString = JsonConvert.SerializeObject(LXSGZ20);
                                msg2 = new Message(Encoding.ASCII.GetBytes(msgString));
                                await _deviceClient.SendEventAsync(msg2);
                            }

                            if (yfDI8Model != null)
                            {
                                msgString = JsonConvert.SerializeObject(yfDI8Model);
                                msg2 = new Message(Encoding.ASCII.GetBytes(msgString));
                                await _deviceClient.SendEventAsync(msg2);
                            }

                            if (yfDQ8Model != null)
                            {
                                msgString = JsonConvert.SerializeObject(yfDQ8Model);
                                msg2 = new Message(Encoding.ASCII.GetBytes(msgString));
                                await _deviceClient.SendEventAsync(msg2);
                            }
                        }
                        else if ("SetRly".Equals((string)jsonMmessageData["Cmd"]) && (yfDQ8Model != null))
                        {
                            JToken rlyState = null;
                            string strRlyState = null;

                            for (int i = 1; i < 8; i++)
                            {
                                rlyState = jsonMmessageData["DQ8Q" + i];
                                if (rlyState != null)
                                {
                                    strRlyState = rlyState.ToString();

                                    if (strRlyState.Equals("1"))
                                    {
                                        yfDQ8Model.Q_Write(i - 1, true);
                                        yfDQ8Model.DeviceValue = (byte)(yfDQ8Model.DeviceValue | (0x1 << (i - 1)));

                                    }
                                    else if (strRlyState.Equals("0"))
                                    {
                                        yfDQ8Model.Q_Write(i - 1, false);
                                        yfDQ8Model.DeviceValue = (byte)(yfDQ8Model.DeviceValue & (~(0x1 << (i - 1))));

                                    }

                                }
                            }
                        }
                    }
                    catch (Exception e) 
                    { 
                       LoggerHelper.Metric($"{source}-Error", 1);
                       LoggerHelper.Exception($"{source}-Error", e, new Dictionary<string, string>{{"Detail",logText }});
                    }
                }

                await Task.Delay(TimeSpan.FromMilliseconds(1000));
            }
        }


    }

```


### Cloud part ###

Several WebJobs are created for specific tasks, as shown below:

_IoTHubToYFIOQueue_: This WebJob will get the messages from the IoT Hub and cache them into the Service Bus queue. The primary reason for this cache is because we find that if we directly forward the message to the remote API gateway within the same WebJob where it is received, we might lose some incoming data due to the latency from the remote API gateway. Below is the event processor that will be kicked off when the WebJob gets started.

```csharp
 public class IOTHubToYFIOQueueProcessor : IEventProcessor
    {
        private const int MAX_BLOCK_SIZE = 4 * 1024;// * 1024;
        public static string StorageConnectionString;
        public static string ServiceBusConnectionString;
        private static string CheckPointContainer = "yfiocheckpoints";
        private CloudBlobClient blobClient;
        private CloudBlobContainer blobContainer;
        //private QueueClient queueClient;
        private AzureQueueClient azureQueueClient = null;
        private long currentBlockInitOffset;
        static string source = "IoTHubToQueue";

        public IOTHubToYFIOQueueProcessor()
        {
            azureQueueClient = new AzureQueueClient(Constains.QUEUE_MessagesToYFIO);
            Log($"Queue:{Microsoft.Azure.CloudConfigurationManager.GetSetting("MessageToYFIO")}");
            Log($"SB:{ServiceBusConnectionString}");
        }

        Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
        {
            LoggerHelper.Event(source, "Info", "Connection Closed");
            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
            return Task.FromResult<object>(null);
        }

        Task IEventProcessor.OpenAsync(PartitionContext context)
        {
            Console.WriteLine("StoreEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);

            LoggerHelper.Event(source, "Info","Connection Opened");

            if (!long.TryParse(context.Lease.Offset, out currentBlockInitOffset))
            {
                currentBlockInitOffset = 0;
            }


            return Task.FromResult<object>(null);
        }
        private static void Log(string msg)
        {
            var log = $"{DateTime.UtcNow.ToString()} {msg}";
            Console.Out.WriteLine(log);
            Console.WriteLine(log);
        }
        async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
        {
            LoggerHelper.Metric(source, messages.Count());
            Log("ProcessEvent called");
            foreach (EventData eventData in messages.OrderBy(m => m.EnqueuedTimeUtc))
            {
                try
                {
                    byte[] data = eventData.GetBytes();

                    var text = Encoding.UTF8.GetString(data);
                    TelemetryData obj = JsonConvert.DeserializeObject<TelemetryData>(text);
 
                    var changed = obj.FindChanged();
                    var connectionDeviceId = eventData.SystemProperties["iothub-connection-device-id"].ToString();
                    changed.DeviceId = connectionDeviceId;
                    var content = JsonConvert.SerializeObject(changed);
                    Log($"Content:{content}");

                    await azureQueueClient.SendAsync(content);
                    await AppendAndCheckpoint(context);

                    WriteHighlightedMessage(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
                      context.Lease.PartitionId, Encoding.UTF8.GetString(data)));
                }
                catch(Exception exp)
                {
                    LoggerHelper.Exception(source, exp);
                    LoggerHelper.Metric($"{source}-Exception", 1);
                }
                finally
                {

                }
            }
        }

```

_FetchYFIOStatusTimely_: This WebJob will regularly query the remote API to see if the user has updated the device status through the mobile app. If so, then the Service Bus queue will be updated correspondingly. Another WebJob—CloudToDeviceCommands—will query the Service Bus queue for this update and send it to the IoT Hub. This function is shown below. 

```csharp 
 public class FetchYFIOStatusTimelyFunctions
    {
        static string source = "FetchYFIOStatus";
        [NoAutomaticTrigger]
        public static void InvokeYFIOGetIOValues()
        {
            int sleep = int.Parse(CloudConfigurationManager.GetSetting("Interval")) * 1000;
            AzureQueueClient queueClient = new AzureQueueClient(Constains.QUEUE_MessagesFromYFIO);
            try
            {
                while (true)
                {
                    try
                    {
                        LoggerHelper.Info("FetchYFIOStatusTimely Triggered");
                        foreach (var project in YFIOHelper2.FindProjects())
                        {
                            var resp = YFIOHelper2.Instance(project).GetIOValues(null);
                            dynamic msg = JsonConvert.DeserializeObject(resp);

                            LoggerHelper.Info($"Project:{project} - {msg}");
                            if (msg.info.status == 1)
                            {
                                msg.body.ProjectID = project;
                                var body = JsonConvert.SerializeObject(msg.body);
                                queueClient.SendAsync(body).Wait();
                            }
                            else
                            {
                                LoggerHelper.Event(source, LoggerHelper.SEVERITY_INFO,
                                                    "YFIOT API returns error",
                                                    new Dictionary<string, string>
                                                    {
                                                        { "ProjectId", project},
                                                        {"Message", resp}
                                                    });
                                LoggerHelper.Metric($"{source}-Error", 1);
                            }
                        }
                    }
                    catch (Exception exp)
                    {
                        LoggerHelper.Exception(source, exp);
                    }
                    finally
                    {
                        Thread.Sleep(sleep);
                    }
                }
            }
            catch(Exception exp)
            {
                LoggerHelper.Error($"Exception :{exp.Message}");
                LoggerHelper.Error($"Exception :{exp.StackTrace}");
            }
            finally {
                queueClient.CloseAsync().Wait();
            }
        }
    }
}

```
_CloudToDeviceCommands_: This WebJob will fetch the update command from the Service Bus queue and leverage the IoT Hub SDK to push the command back to the device side.

```csharp
public class CloudToDeviceCommandsFunctions
    {
        static string source = "CloudToDeviceCommands";
        private static void Log(string message)
        {
            Console.WriteLine($"[{DateTime.UtcNow}]{message}");
            LoggerHelper.Event(source, "Info", message);
        }
        public static async Task ProcessQueueMessage([ServiceBusTrigger("%CommandsToDevice%")] BrokeredMessage message,
                TextWriter logger)
        {
            Log($"{message} received at {DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss")}");
            ChangedData data = null;
            data = AzureQueueClient.GetMessageContent<ChangedData>(message);
            if (data.Names == null || data.Names.Count == 0)
            {
                return;
            }
            var dataText = JsonConvert.SerializeObject(data);
            Log($"{dataText} received");
            LoggerHelper.Metric(source,1);
            // Send to device
            //20161018, per talk, a gateway backed for multiple devices. if no deviceid specified
            //iterate all devices under a given project
            var sc = ServiceClient.CreateFromConnectionString(
                            CloudConfigurationManager.GetSetting("iothubMgmt"), Microsoft.Azure.Devices.TransportType.Amqp);
            //message.RenewLock();
            try
            {
                await sc.OpenAsync();
                if (string.IsNullOrEmpty(data.DeviceId))
                {
                    RegistryManager rm = RegistryManager.CreateFromConnectionString(CloudConfigurationManager.GetSetting("iothubMgmt"));
                    var devices = await rm.GetDevicesAsync(100);
                    foreach (var deviceId in devices.Select(d => d.Id).ToArray())
                    {
                        data.DeviceId = deviceId;
                        var cmd = DeviceCommandHelper.CreateDeviceSetRlyCommand(data);
                       
                        Log($"Sending {cmd}");
                        await sc.SendAsync(data.DeviceId, new Message(Encoding.UTF8.GetBytes(cmd)));
                    }
                }
                else
                {
                    var cmd = DeviceCommandHelper.CreateDeviceSetRlyCommand(data);
                    Log($"Sending {cmd}");
                    await sc.SendAsync(data.DeviceId, new Message(Encoding.UTF8.GetBytes(cmd)));
                }
            }
            catch(Exception exp)
            {
                LoggerHelper.Metric($"{source}-Error", 1, exp);
                LoggerHelper.Exception(source, exp,
                    new Dictionary<string, string>
                    {
                        {"DeviceId",data.DeviceId },
                        { "Detail" ,dataText}
                    });
                throw;
            }
            finally
            {
                await sc.CloseAsync();
            }
            // Remove message from queue.
            message.Complete();
        }
    }
}
```

For the Azure Stream Analytics part, there are two major uses in the first stage:

1. Archiving all the data into Blob storage. The Stream Analytics query is shown below: 
        
        SELECT * INTO BlobArchiveOutput FROM eventInput TIMESTAMP BY Time 

2. Tracking the event when both the CO2 reading and PM25 reading are above a certain threshold, and archiving these events for later tracking.
       
          SELECT  DeviceId,ProjectId, EventEnqueuedUtcTime, YFSECO2 YFSEPM25 INTO BlobAlarmOutput 
          Where YFSECO2 > 1000 and YFSEPM25 > 150 
          FROM eventInput 

#### Below is the screenshot from the Azure portal about the consumption status ####

*IoT Hub usage*

![NewHopeLiuhe Smart Farming IoT Hub usage]({{ site.baseurl }}/images/NewhopeLiuhe/IoTHubUsage.png)


*Service Bus queue usage*

![NewHopeLiuhe Smart Farming Service Bus usage]({{ site.baseurl }}/images/NewhopeLiuhe/ServiceBusQueueUsage.png)


*App Service WebJob usage*

![NewHopeLiuhe Smart Farming Service Bus usage]({{ site.baseurl }}/images/NewhopeLiuhe/WebJobUsage.png)


### Mobile end ###

Users can use the mobile app or the H5 chart embedded in the WeChat app to check the data and control the remote device. Some screenshots from the mobile part are shown below.

![NewHopeLiuhe Smart Farming MobileApp]({{ site.baseurl }}/images/NewhopeLiuhe/MobileApp.png)


![NewHopeLiuhe Smart Farming wechat1]({{ site.baseurl }}/images/NewhopeLiuhe/wechat1.png)


![NewHopeLiuhe Smart Farming wechat2]({{ site.baseurl }}/images/NewhopeLiuhe/wechat2.png)


## Project moment ##

Below are some photos from the hackfest at the Microsoft office and the deployment site.

*Coding and debugging the device at the Microsoft office*

![NewHopeLiuhe Smart Farming hackfest1]({{ site.baseurl }}/images/NewhopeLiuhe/hackfest1.png)


![NewHopeLiuhe Smart Farming hackfest2]({{ site.baseurl }}/images/NewhopeLiuhe/hackfest2.png)


*Deploying the solution at a chicken farm*

![NewHopeLiuhe Smart Farming deployment1]({{ site.baseurl }}/images/NewhopeLiuhe/deployment1.png)


![NewHopeLiuhe Smart Farming deployment2]({{ site.baseurl }}/images/NewhopeLiuhe/deployment2.png)


![NewHopeLiuhe Smart Farming deployment3]({{ site.baseurl }}/images/NewhopeLiuhe/deployment3.png)


![NewHopeLiuhe Smart Farming deployment4]({{ site.baseurl }}/images/NewhopeLiuhe/deployment4.png)


## Issues experienced ##

The biggest issue is that the MQTT protocol is not supported on a UWP app. This is a known issue: [https://github.com/Azure/azure-iot-sdks/issues/881](https://github.com/Azure/azure-iot-sdks/issues/881). HTTP is used in the first version. The solution will be updated to adopt AMPQ after the stability test.

## Opportunities going forward ##

- Adding support for Power BI Embedded into the mobile app.
- Scaling the solution to more sites. The current deployment needs to run for at least 60 days to validate the stability of the whole system. After this phase, we will discuss scaling to additional sites with New Hope Liuhe.

## Partner and customer feedback ##

> "By leveraging the great technology from Microsoft, we hope to reinforce the leadership of New Hope Liuhe in the farming industry, and also incubate and develop the new generation of IoT solutions for this industry to embrace the digital transformation."

> —Xia ZanHui, GM of New Hope Liuhe

> "Microsoft provides a comprehensive features set which covers both the device and cloud part. We could pick the ones that best fit our scenario to greatly fasten our solution development."

> —Liu Hongfeng, CEO of Ye Fan
