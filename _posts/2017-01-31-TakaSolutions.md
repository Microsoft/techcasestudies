---
layout: post
title:  "Using IoT in building management to bring energy savings to Taka Solutions customers"
author: "Mert Unan"
author-link: "# Include twitter link here"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-17
categories: IoT
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: Taka Solutions implemented a building energy-management IoT solution that uses Azure IoT Hub and Stream Analytics to centralize telemetry from multiple buildings for visualization and analytics. Its ultimate goal is to reduce the world's energy consumption.
language: English
verticals: [Energy, Environmental, Smart Cities]
---


Microsoft and Taka Solutions joined forces to implement a proof of concept that integrates Tridium Niagara deployments for building management with Azure IoT services for centralization of the telemetry. This would enable Taka Solutions to do advanced energy consumption analysis, reporting, and visualization.

**Key technologies:**

- [Tridium Niagara](https://www.tridium.com/)
- [oBIX (Open Building Information Xchange)](http://www.obix.org/) 
- [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/)
- [Azure Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/)
- [Azure DocumentDB](https://docs.microsoft.com/en-us/azure/documentdb/)
- [Microsoft Power BI](https://powerbi.microsoft.com/en-us/)
- C#

**The core team:**

- Charles Blaschke – Managing Director, Taka Solutions
- Nathan Jenkins – Data Analytics, Taka Solutions
- Henry Jeha – Energy Engineer, Taka Solutions 
- Mert Unan – DX Technology Evangelist, Microsoft
- Hatim Nagarwala – DX Application Developer, Microsoft

## Customer profile ##

[Taka Solutions](http://takasolutions.com/) is a Dubai-based energy efficiency technology company that focuses on *smart building* implementations. Its business model is to retrofit, or implement during construction, buildings with "smart" equipment to reduce energy loss from air conditioning, heat insulation, electricity, and water. 

The initial retrofitting is done free of charge to the customer, with Taka Solutions taking a fraction of the customer's savings over the energy management period. This paid-through-savings model ensures aligned interests and long-term partnerships with customers.

The smart cities implementation allows Taka Solution's customers to reduce their energy bills while also reducing the facilities’ environmental footprint. The company says its goal is “to reduce world consumption by 20%.” Taka Solutions is reimagining the usage of building IoT telemetry while pushing the boundaries of positive environmental impact.

## Problem statement ##

>“Having this IoT setup will allow us to quickly connect multiple buildings and begin performing prediction, machine learning, and ultimately saving more energy, carbon, and money for our customers to help our goal of reducing the world's energy consumption by 20%.”
>
>– Charles Blaschke, Managing Director, Taka Solutions

Taka Solutions wanted to explore a new way of aggregating/centralizing all the data from the server hardware located in the buildings. Such functionality would enable Taka Solutions to get the data feed from all the buildings in near real-time and compare it to historical data. The idea was to implement a solution where multiple buildings are connected with capabilities such as:

- Archiving data from buildings.
- Historical lookup for certain time periods.
- Remote monitoring of buildings and energy.
- Visualization and dashboards of energy telemetry from buildings.
- Drill-down to “per device controller” level in the buildings to better understand energy usage.
- Predicting values like “enthalpy” or “heat loss” and future consumption.

Initial setup in the buildings used Tridium Niagara building-management software running on Windows-based PCs. The team focused on creating a solution without replacing the current setup, thus avoiding any additional cost. Niagara was responsible for gathering all the data from the controllers and saving it locally. The next step was to come up with a solution that connects this data to the cloud. 

## Solution and steps

The solution will have a Windows service client that will extract the data from Tridium Niagara using the oBIX drivers and be accompanied by a number of Azure services to accommodate data persistence, analytics, and visualization

To address the requirements, the team discussed various options and decided on the following architecture for fulfilling the scenario at hand:

- IoT Hub to ingest telemetry from buildings.
- Stream Analytics to route information flow and enable near real-time streaming of telemetry.
- DocumentDB to store the data and to provide BI with historical building telemetry.
- [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/) to calculate additional custom values from the telemetry.

*Figure 1. Cloud architecture*

![Cloud Architecture]({{ site.baseurl }}/images/2017-01-31-takasolutions/snip_20170131134642.png) 


### Implementation steps ###

The first step was to obtain the data from the Tridium Niagara BMS (building management system) running on the Windows OS with different versions varying from building to building. We chose one of the buildings where the Niagara system was already deployed to start experimenting. The team decided to implement a Windows service that can communicate with Niagara deployed in the same hardware to extract the telemetry and send it to an IoT hub for data ingestion. 

The team used the oBIX protocol to communicate with Niagara. The oBIX driver for Niagara was not installed by default, so Taka Solutions installed it accordingly. The oBIX driver for Niagara enables the Niagara controller to use an endpoint that can be queried over the network to respond with an XML payload of data telemetry. This way, the team could use the oBIX protocol to query Niagara for telemetry from the Windows service implementation.

*Figure 2. Data flow design*

![Data Flow Design]({{ site.baseurl }}/images/2017-01-31-takasolutions/dataflow.JPG) 


The second step involved the data design. Dashboards in Power BI didn’t require all the data stored in Niagara to be displayed; hence, the next challenge was to identify the data points to be sent. 

In step three, the team created and configured the resources: IoT Hub, Stream Analytics, DocumentDB, and Power BI account integration. This was a project milestone to see the data flow from Niagara to Power BI. 

*Figure 3. Stream Analytics job diagram*

![SA job diagram]({{ site.baseurl }}/images/2017-01-31-takasolutions/sajobdiagram.JPG)


Next, the team started working with Power BI Desktop to add custom value columns for additional telemetry that is calculated using the values from the building telemetry. Power BI dashboards, data altering, and filtering are applied to the raw data coming from the stream.

In the final step, the Windows service was packaged, installed, and configured on the building to enable data flow from the site. This generic approach will enable Taka Solutions to extend its sites with minimal effort. Upon installation of the Windows service, the sites will start providing data to the Power BI dashboards. The default frequency of data push is five minutes. This is configurable using the config file provided for the service to the desired frequency. This is in case there might be a need for near real-time deployment in the future.

## Technical delivery

### Field device specs

At given sites, Taka Solutions has different OS versions on Windows-based PCs that run Tridium Niagara. There are multiple JACE 8000 controllers manufactured by Tridium. These controllers use line power from the building mains connection (electrical supply) and support Wi-Fi capability of A, B, G, or N networks. They also have 10/100 MB Ethernet ports and isolated RS485 ports. The preferred connection is Ethernet if the implementation allows. JACE 8000 controllers run Tridium Niagara 4.1 or later. The devices have a removable MicroSD card storage of 4 GB. 

These controller devices are responsible of gathering telemetry from the sensors and feed it to the main PC that uses a C# Windows service to connect to the Azure IoT hub. The device supports expansions that are then used to connect to analog sensors. This solution has multiple types of sensors that measure airflow, temperature, humidity, and more. One example of those sensors is ESF-35-2 from Greystone. This device uses the principle of air cooling effect of the air flow to measure the flow through a given space. It reports an analog current signal of 4-20mA, which reaches the JACE 8000 controller.

### oBIX and Windows service ###

Tridium Niagara was responsible for collecting and storing the building telemetry locally. The team implemented a Windows service to collect that data to be sent to an IoT hub. Niagara uses the protocol oBIX, which serves the data upon a request to a specific oBIX endpoint in XML format. In our case, Niagara and the Windows service work on the same physical hardware. Thus, making a request to the local oBIX endpoint yields the desired outcome. In case the Windows service and the Niagara deployment are located in separate physical hardware, the request needs to be done to the corresponding endpoint.

The solution has two functions that create and send the request to the oBIX endpoint to receive an XML response. These functions are included in a helper class called “Weblayer.” 

```csharp
public static string CreateRequest(string queryString)
	{
		try
		{
		return (ConfigurationManager.AppSettings["ObixUrl"] + queryString);
		}
		catch (Exception ex)
		{
			Library.WriteErrorLog("Create Request Error" + ex.Message);
return "";
		}
		//Creates the Path for making request

		}

	public static XmlDocument MakeRequest(string requestUrl)
	{
		try
		{

			HttpWebRequest request = WebRequest.Create(requestUrl) as HttpWebRequest;
			string authInfo = "Username" + ":" + "Password";
			authInfo = Convert.ToBase64String(Encoding.Default.GetBytes(authInfo));

			request.Headers[HttpRequestHeader.Authorization] = "Basic " + authInfo;
			HttpWebResponse response = request.GetResponse() as HttpWebResponse;

			XmlDocument xmlDoc = new XmlDocument();
			using (var responseStream = response.GetResponseStream())
			{
				xmlDoc.Load(responseStream);
			}

			return (xmlDoc);


		}
		catch (Exception ex)
		{
			//log the exception 

			Library.WriteErrorLog("Make Request Error" + ex.Message);
			return null;
		}
	}
```

Another function in the same class is used to send telemetry to an IoT hub, shown below.

```csharp
public static void Post2IoTHub(string Message)
		{
			try
			{
				HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://”yourIotHubhere”.azure-devices.net/devices/”yourdevicenamehere”/messages/events?api-version=2016-02-03");
				request.Method = "POST";
				request.ContentType = "application/xml";

 					request.Headers.Add(HttpRequestHeader.Authorization, "SharedAccessSignature sr=Yoursharedaccesssigniturehere");

				byte[] bytes = System.Text.Encoding.ASCII.GetBytes(Message);
				request.ContentLength = bytes.Length;
				System.IO.Stream os = request.GetRequestStream();
				os.Write(bytes, 0, bytes.Length); 
				os.Close();
				System.Net.WebResponse resp = request.GetResponse();
			}
			catch (Exception PostException)
			{
				Library.WriteErrorLog("Post Error" + PostException.Message);

			}
		}
```

The Niagara structure for our example installation was as follows: a tree hierarchy consisting of stations, units, and measurements. This hierarchy is traversed in the code in a top-down approach.

- Each building has stations.
- Each station has units.
- Each unit has measurements, which are the actual values from the sensors, such as temperature, humidity, and airflow.

*Figure 4. Control system hierarchy*

![Control System Hierarchy]({{ site.baseurl }}/images/2017-01-31-takasolutions/controllerhierarchy.JPG)


To represent this structure in code, we had three model classes: measurement, station, and unit. The query to the oBIX endpoint responds back with an XML file, which has to be parsed and then converted into JSON values. The URL of the controller that comes with the response is required to construct the next endpoint to be queried.

```csharp
public class Measurement
{
	public string Name { get; set; }
	public string Value { get; set; }
	public string href { get; set; }
}

public class Station
{
	public string Name { get; set; }
	public string href { get; set; }
}

	public class Unit
{
	public string Name { get; set; }
	public string href { get; set; }
}
```

The base oBIX unit is queried to obtain a list of stations. Then each station is queried to obtain a list of units that belong to that particular station. Then each unit is queried to obtain a list of measurement values attached to it.

At the unit level, a Post2IoTHub function is used to post the values to the IoT hub.

```csharp
private void timer1_Tick(object sender, ElapsedEventArgs e)
{
	try
	{
		Library.WriteErrorLog("Timer Started");

		//get the base xml document to start
		//creating with the base URL
		string StationRequest = WebLayer.CreateRequest("");
		XmlDocument StationsXml = WebLayer.MakeRequest(StationRequest);

		ProcessResponse(StationsXml);

	}
	catch (Exception ex)
	{
		Library.WriteErrorLog("Timer ticked at this time - " + DateTime.Now + ex.Message);
	}

}

private void ProcessResponse(XmlDocument stationsXml)
{

	List<Station> _stationsList = ParsingHelpers.ParseNiagraStations(stationsXml);

	foreach (var _station in _stationsList)
	{

		try
		{
		string UnitRequest = WebLayer.CreateRequest(_station.href);
		XmlDocument UnitXml = WebLayer.MakeRequest(UnitRequest);

		List<Unit> _unitsList = ParsingHelpers.ParseUnits(UnitXml);

			foreach (var _unit in _unitsList)
			{
				try
				{
					string MessageToSend = "{";

					MessageToSend += "\"SiteId\":" + "\"" + ConfigurationManager.AppSettings["SiteID"] + "\"" + ",";
					MessageToSend += "\"ControllerID\":" + "\"" + _station.Name + "\"" + ",";
					MessageToSend += "\"UnitID\":" + "\"" + _unit.Name + "\"";



					string MeasureMentRequest = WebLayer.CreateRequest(_station.href + _unit.href);
					XmlDocument MeasurementXml = WebLayer.MakeRequest(MeasureMentRequest);


					List<Measurement> _measurementList = ParsingHelpers.ParseMeasurements(MeasurementXml, _station.href + _unit.href);


					MessageToSend = CreateJson(MessageToSend, _measurementList);
					//to send to IoT Hub, default frequency is 60000ms (10 mins) can be changed thru the config file as desired
					WebLayer.Post2IoTHub(MessageToSend);
					Library.WriteErrorLog("The Code pushed successfully");

				}
				catch (Exception unitException)
				{
					Library.WriteErrorLog("Unit Exception" + unitException.Message);
				}

			}
		}
		catch (Exception stationException)
		{
			//write the logging code

			Library.WriteErrorLog("Station Exception" + stationException.Message);
		}
		//Getting the units for the current Station
 
	}
}
```

This method ensures the tree is traversed and all the values that belong to all the units are sent to the IoT hub. Also, if a new unit or controller is added to a building, it will immediately be included in the search because we request the current list of units each time individually.

### Data design ###

One of the challenges was around the data schema. Every unit provides data that is semi-structured and the measurements change from unit to unit. One of the units might be dedicated to temperature and humidity whereas another unit under the same controller might be measuring something completely different such as AC flow speed and power status. The data schema requirement for both transportation and storage is semi-structured and must be generic so that any newly added sensor would comply without disrupting the system operation. The team decided to use JSON to transport the data and Azure (NoSQL) DocumentDB to store it.

This enabled Power BI to connect to DocumentDB for a cold-path scenario using the integrated connector to access historical data and create custom dashboards. One of the project requirements was to have values calculated based on the parameters coming from the buildings. This was achieved by using custom calculated columns in Power BI Desktop.

Sample JSON document from DocumentDB:

```json
{
	"SiteID":"hidden",
	"ControllerID": "hidden",
	"UnitID": "hidden",
	"Bag_Filter": "20.0",
	"CO2": "417.6",
	"Frequency": "35.09";
	"Humidty": "100.0",
	"In": "500.0",
	"NumericSwitch1": "20.0",
	"Temperature": "26.29",
	"Total_Kwh": "9379.0",
	"Valve": "100.0",
	"Velocity": "86.32089725024065",
	"EventProcessedUtcTime": "2017-01-11T10:23:45.6982534Z",
	"PartitionId": 1,
	"EventEnqueueUtcTime": "2017-01-11T10:23:45.177Z",
	"IoTHub": {
		"MessageId": null,
		"CorrelationId": null,
		"ConnectionDeviceId": "hidden",
		"ConnectionDeviceGenerationId": "636194698879646546",
		"EnqueuedTime": "0001-01-01T00:00:00",
		"StreamId": null
	},
	"id": "d6b6aac3-8fe3*9dad-cb8d-087ce1b2f10d"
}
	
```

The second Stream Analytics output sink was configured to be Power BI, which gave near real-time reporting on building parameters. The power of the Stream Analytics, Power BI, and DocumentDB trio gave the team the opportunity to create dashboards that can pull data from historical archives and near real-time feeds displayed in a single dashboard.

### IoT cloud architecture ###

The cloud architecture was done using the Azure portal. The workloads deployed are:

- IoT Hub
- Stream Analytics
- DocumentDB

IoT Hub is deployed and the Windows service is configured to receive the data coming from the buildings. Stream Analytics has one input sink configured to be the IoT hub from the same Azure subscription and two separate output sinks to Power BI and DocumentDB. Power BI Desktop is used to configure a connector to the deployed DocumentDB. This enables Power BI to automatically receive schemas and data from DocumentDB.

Power BI Desktop offers a way to add custom columns that can contain formulas to calculate new values depending on other columns. This enables calculation of dependent values to be displayed in the dashboards. This eliminates the need to do the calculation on the code side and gives more flexibility in terms of data design. Also, this way Taka Solutions will be able to add/change values in the future just by using Power BI and without having to update the code.

### Windows service ###

We started by testing the Windows service with the example building. The starting state of the Windows service is set to automatic, which starts the service on system boot. This is particularly useful for the service to keep running in the event of an issue such as an unintended reboot of the server, an electricity disruption, or a maintenance restart.

The service implements a task that runs on a scheduled interval. The interval frequency value is stored on the `app.config` file so it can be updated accordingly, catering to the need of the site, the controller, and so on. By default, the frequency is set to 300,000 milliseconds (5 minutes).

```csharp
public partial class Scheduler : ServiceBase
{
	private Timer timer1 = null;

	public Scheduler()
	{
		InitializeComponent();
	}

	protected override void OnStart(string[] args)
	{
		timer1 = new Timer();
		this.timer1.Interval = double.Parse(ConfigurationManager.AppSettings["Frequency"].ToString());//value in configuration file
		this.timer1.Elapsed += new System.Timers.ElapsedEventHandler(this.timer1_Tick);
		timer1.Enabled = true;
		Library.WriteErrorLog("Test window service started Successfully");
   
	}
}
```

The library class implements a StreamWriter to create a local error log in the file system. This enables tracking and debugging of the error in the event of an interruption to the data connection.

```csharp
public static class Library
{
	public static void WriteErrorLog(Exception ex)
	{
		StreamWriter sw = null;
		
		sw = new StreamWriter(AppDomain.CurrentDomain.BaseDirectory + "\\LogFile.txt", true);
		sw.WriteLine(DateTime.Now.ToString() + ": " + ex.Source.ToString().Trim() + "; " + ex.Message.ToString().Trim());
		sw.Flush();
		sw.Close();
		
	}

	public static void WriteErrorLog(string Message)
	{
		StreamWriter sw = null;
				
		sw = new StreamWriter(AppDomain.CurrentDomain.BaseDirectory + "\\LogFile.txt", true);
		sw.WriteLine(DateTime.Now.ToString() + ": " + Message);
		sw.Flush();
		sw.Close();
		
		
	}
}
```

### Security and device registration ###

The communication between the end devices (sensors collecting the values from the building) and the Windows PCs that host Tridium Niagara is through the building network that is physically secured. The PCs are in restricted areas that require security clearance. The software runs on PCs that are used only by authorized administrators. Our Windows service communicates with Tridium Niagara on the same machine locally.

The communication between the Windows service and the IoT hub is HTTPS using a [shared access signature](https://docs.microsoft.com/en-us/azure/storage/storage-dotnet-shared-access-signature-part-1) (SAS). SAS is a powerful way of granting limited access to the calls, made to an endpoint. SAS works together with a policy defined in the Azure workload. The possible permissions for IoT hub are:

- Registry read
- Registry write
- Service connect
- Device connect

In this case, we are using the **device** default policy that contains the **Device connect** permission. Each policy has a generated primary and a secondary key. Both of them can be used on the device side to create what's called a SAS token.

The generated SAS token is passed along with every call to authenticate against the endpoint with the related policy.

When you require per-device security and control, IoT Hub includes a device registry so that each individual device can be registered. There are a couple of ways to do this, such as the command-line tool, via [Device Explorer for IoT Hub](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer), or through the code. For the scope of this project, we decided to do this through code because we prefer the autonomy when different sites are set up for connecting to the solution. Below is a sample code to achieve this.

```
	registryManager = RegistryManager.CreateFromConnectionString(connectionString);
	AddDeviceAsync().Wait();
	
 
static async Task AddDeviceAsync()
{
	Device device1;
	try
	{
		device = await registryManager.AddDeviceAsync(new Device(deviceId));
	}
	catch (DeviceAlreadyExistsException)
	{
		device = await registryManager.GetDeviceAsync(deviceId);
	}
        Console.WriteLine("Generated device key: {0}", device.Authentication.SymmetricKey.PrimaryKey);
}
```

## Conclusion ##

The main purpose of this project was to enable an integration between an on-site building management system, Tridium Niagara, and Azure. After this implementation, Taka Solutions will have:

-  Multiple sites connected to the cloud platform.
-  An archive of the historical telemetry data on a nonstructured database that will not require changes if the data schema changes.
-  Access to the near real-time telemetry from buildings.
-  A platform, Power BI, to modify/add extra calculated values without any changes in software code.
-  Power BI to create custom dashboards for different user profiles from the data.
-  A way to enroll new buildings by installing a Windows service and changing a configuration file on the site.

All the components on this implementation were created as generically as possible, providing flexibility to different deployments, data types, and schemas.

This implementation traverses data points in Tridium Niagara individually. Hence, additional sensors can be added/removed and the rest of the system will adapt.

The Windows service can be installed in different projects as long as the Tridium Niagara hierarchy is the same. This will automatically provide the site server with the ability to traverse and stream the data to an IoT hub endpoint provided in the `app.config` file.

Note that the Windows service requires Windows-based hardware for Windows 8 and up or Windows Server to operate on. Any other hardware that Niagara runs on will not work with this installation.

### Opportunities going forward ###

The team at Taka Solutions plans to deploy the solution to their current building installations. They want to establish a stable flow of data from the sites to the custom-created dashboards.

The next step for this project is to implement a machine-learning web service using Azure Machine Learning Studio. This web service will enable complex predictions of values energy consumption and temperature regression analysis. The idea is to add this web service as a function to Stream Analytics to filter and predict in near real-time before storing in a database.

### Additional resources ###

- [Taka Solutions](http://takasolutions.com/)
- [Tridium home page](https://www.tridium.com/)
- [oBIX (Open Building Information Xchange)](http://www.obix.org/)
- [Azure IoT Hub](https://docs.microsoft.com/en-us/azure/iot-hub/)
- [Azure Stream Analytics](https://docs.microsoft.com/en-us/azure/stream-analytics/)
- [Azure DocumentDB](https://docs.microsoft.com/en-us/azure/documentdb/)
- [Microsoft Power BI](https://powerbi.microsoft.com/en-us/)
- [Microsoft Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/)
