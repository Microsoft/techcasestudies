---
layout: post
title:  "How OpportunitySpace is boosting the effort to restore neighborhoods with a Xamarin.Forms mobile app"
author: "Gavin Bauman"
author-link: "#"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-02-09
categories: [Mobile Application Development with Xamarin]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: This article details the implementation of a cross-platform mobile application built in Xamarin that takes vector map data from an API and renders it using Xamarin.Forms.
language: English
verticals: "Retail, Consumer Products & Services"
---



OpportunitySpace is a Boston-based startup focused on providing contextual neighborhood data to code enforcement and government officials. Until now, it had a web application that allowed users to research before heading out of the office. But the company needed a mobile app to provide that data wherever the users may be. Resizing the web application for a phone's mobile browser did not provide the smooth experience that OpportunitySpace wanted, so it started to look at building a native mobile app.  

**Core team:**

- Nicolas Tejera – CTO, OpportunitySpace  
- [Gavin Bauman](https://twitter.com/gavination2) – Technical Evangelist, Microsoft
- [Ryan Lowdermilk](https://twitter.com/RyanLowdermilk) – Technical Evangelist, Microsoft
- [Paul DeCarlo](https://twitter.com/pjdecarlo) – Technical Evangelist, Microsoft
- [Katherine Harris](https://twitter.com/KatVHarris) – Technical Evangelist, Microsoft

## Solution overview ##

To help government officials and social workers prioritize their work, we teamed up with OpportunitySpace to build a client app for their pre-existing service. We created a Xamarin application that provides the information requested from the back-end service onto a cross-platform mobile application that government officials will be able to use on the job.

## Customer profile ##

[OpportunitySpace](https://www.opportunityspace.org/) provides a mapping and analytics application for municipal officials tasked with fighting urban decay. By bringing together a wealth of parcel-specific data into a single dashboard, OpportunitySpace enables cities to deploy data-driven strategies that maximize benefits in disinvested communities. Cities are tracking and creating a wealth of valuable information on property and neighborhood vitality.

OpportunitySpace's goal is to aggregate the data in order to drive actionable insights to promote the repair of damaged neighborhoods and cities. The result is data organized in a beautiful and understandable way at the fingertips of the cities that own it. 

Because the app's government users do much of their work in underserved communities, it would be incredibly beneficial if they could query their application from the field to get a holistic view of the properties they are visiting. A code enforcement official who knows that a property has an extensive criminal history can exercise appropriate caution before paying a visit. Police officers responding to a call for service can do the same if they know the structure they are about to visit has been condemned. And a community development professional who is meeting a family in need can alert them if they are in danger of losing their home through tax foreclosure.

These capabilities will not only improve OpportunitySpace's offering to governments, growing its customer base and increasing its staying power with existing customers, but they will also provide governments with powerful tools to help revitalize the neighborhoods that need it most across the country.

## Problem statement ##

As of now, OpportunitySpace has a web application for different municipalities to register. When this site renders on mobile, however, it does not provide an optimal experience for the end user. The developers seek to bring their platform to mobile devices in the form of a mobile application. This poses issues around the time taken to develop and test an individual app for each platform as well as expertise needed in using the native tooling and platforms for these targets. By leveraging Xamarin.Forms, we were able to build a cross-platform mobile application for the business that provided native performance as well as the functionality needed to act as a proper client. 

## Solutions, steps, and delivery ##

A number of different components are involved in the development of the OpportunitySpace client application. It's a cross-platform mobile application written using Xamarin.Forms as a framework that communicates to a back end using both Azure and AWS as cloud providers. An overview of the application architecture is below:

![ArchitectureDiagram]({{ site.baseurl }}/images/Architecture Diagram - OpportunitySpace Platform.png)


**Building the application UI using Xamarin.Forms**

The decision to go with Xamarin.Forms arose out of a need to develop quickly. One of our initial challenges, however, was using the Mapbox SDK, a core requirement that OpportunitySpace leaned on to deliver the mapping data for their clients. Unfortunately, while there was a Mapbox SDK for Classic Xamarin, there was not one for Xamarin.Forms. 

We experimented with building out the mapping platform using Classic Xamarin, but while effective, the development overhead was too cumbersome, especially for a hackfest. With some ingenuity, we were able to forego the need for the Mapbox by consuming the vector tile data exposed by the API, processing it on the mobile client, and passing that information into the Xamarin.Forms map control. The map control for Xamarin.Forms calls the native mapping applications for each platform (for example, Google Maps for Android, Apple Maps for iOS, and Bing Maps for Windows) and provides the end user with a rich native experience. The screenshots are below:

*Android logon screen*

<img src="{{ site.baseurl }}/images/droid-login.png" width="300">


*Android map screen*

<img src="{{ site.baseurl }}/images/droid-map.png" width="300">


*Android filter screen*

<img src="{{ site.baseurl }}/images/droid-filter.png" width="300">


*iOS logon screen*

<img src="{{ site.baseurl }}/images/ios-login.png" width="300">


*iOS map screen*

<img src="{{ site.baseurl }}/images/ios-map.png" width="300">


*iOS filter screen*

<img src="{{ site.baseurl }}/images/ios-filter.png" width="300">


*Windows logon screen*

<img src="{{ site.baseurl }}/images/UWP.PNG" width="300">


*Windows map screen*

<img src="{{ site.baseurl }}/images/UWP-map.PNG" width="300">



**MVVM: pros and cons**

The team decided to use the Model-View-ViewModel (MVVM) paradigm to build their solution for better testability and code reuse throughout future iterations of the product. However, we did not want to be confined to a third-party MVVM framework, so we used Xamarin.Forms' built-in ViewModel platform for building our application. We did run into some interesting challenges, notably navigation from page to page. There was no easily built-in page navigation feature for Xamarin.Forms, so we bypassed that by creating an invisible toggle control on the page we needed to navigate from that would flip and navigate to the next page.

```xaml
    <Switch x:Name="LoginStatus" Opacity="0" IsToggled="{Binding IsLoggedIn}"/>
```
```csharp
 public LoginPage()
        {
            var vm = new LoginViewModel();
            this.BindingContext = vm;

            InitializeComponent();

            LoginStatus.Toggled += LoginStatus_Toggled;
        }

        private void LoginStatus_Toggled(object sender, ToggledEventArgs e)
        {
            if (LoginStatus.IsToggled)
                Navigation.PushAsync(new FilterMapPage());
        }
```

**Vector tiles with Xamarin.Forms maps**

In the App.cs file we initialized a simple Xamarin.Forms map control and wired up an event handler. When we move the map, tiles are requested.

```csharp

public App()
        {
            var map = new Map(MapSpan.FromCenterAndRadius(
                new Position(38.2527, -85.7585),
                Distance.FromMiles(1.5)));

            var cp = new ContentPage
            {
                Content = map
            };

            MainPage = cp;
            map.PropertyChanged += Map_PropertyChanged;
        }

```

Based on the center of the map, we asked for a center tile and the surrounding 8 tiles. This happens every time we move the map.

```csharp
private async void Map_PropertyChanged(object sender, System.ComponentModel.PropertyChangedEventArgs e)
{
    var m = (Map)sender;

    if (m.VisibleRegion == null)
        return;

    Exception error = null;

    try
    {
        var lat = m.VisibleRegion.Center.Latitude;
        var lng = m.VisibleRegion.Center.Longitude;

        TileData t = WorldToTilePos(lng, lat, m);

        for (int x = -1; x < 2; x++)
        {
            for (int y = -1; y < 2; y++)
            {
                ProcessTile(t, m, x, y);
            }
        }

    }
    catch (Exception ex)
    {
        error = ex;
    }

    if (error != null)
        await Application.Current.MainPage.DisplayAlert("Error!", error.Message, "OK");
}

```

Finally, for each tile, we processed any binary tile data we may have had on our server or, in this case, any local binary vector files.

```csharp
private bool ProcessTile(TileData t, Map m, int xOffset, int yOffset)
{
    int x = t.X + xOffset;
    int y = t.Y + yOffset;

    string key = $"{x}_{y}";

    if (MapStore.XY.ContainsKey(key))
    {
        return false;
    }
    else
    {
        MapStore.XY.Add(key, true);
    }

    Assembly assembly = GetType().GetTypeInfo().Assembly;
    string binaryFile = $"MapFilter.{VectorTileDataFolder}.{t.Z}_{x}_{y}.mvt";

    Debug.WriteLine(binaryFile);

    using (Stream stream = assembly.GetManifestResourceStream(binaryFile))
    {
        if (stream == null)
            return false;

        var layerInfos = VectorTileParser.Parse(stream);

        if (layerInfos.Count == 0)
            return false;

        var fc = layerInfos[0]?.ToGeoJSON(x, y, t.Z);

        foreach (var geo in fc.Features)
        {
            var lng1 = ((GeoJSON.Net.Geometry.GeographicPosition)((GeoJSON.Net.Geometry.Point)geo.Geometry).Coordinates).Longitude;
            var lat1 = ((GeoJSON.Net.Geometry.GeographicPosition)((GeoJSON.Net.Geometry.Point)geo.Geometry).Coordinates).Latitude;
            m.Pins.Add(new Pin() { Position = new Position(lat1, lng1), Label = $"{lng1},{lat1}" });
        }
    }
    return true;
}
```

TileData is a Plain Old CLR Object (POCO) that provides a simple tile with an X, Y, and Z property. This allows our code to be a bit more readable.

```csharp
namespace MapFilter
{
    public class TileData
    {
        public int X { get; set; }
        public int Y { get; set; }
        public int Z { get; set; }
    }
}
```

MapStore is a POCO that we used to cache our tile requests. We wanted to ask for a particular tile once.

```csharp
using System.Collections.Generic;

namespace MapFilter
{
    public static class MapStore
    {
        public static Dictionary<string, bool> XY = new Dictionary<string, bool>();
    }
}
```

**Building the TileSystem**

Based on feedback from the Bing Maps team, we used some geo-industry-standard constants and calculations. We did not use all the methods from the TileSystem. In fact, the method we were most interested in is LatLongToPixelXY, which converts a given point's longitude, latitude, and zoom level to pixel XY coordinates. This was one of the methods we used to place markers onto our map.

```csharp
  /// <summary>
        /// Converts a point from latitude/longitude WGS-84 coordinates (in degrees)
        /// into pixel XY coordinates at a specified level of detail.
        /// </summary>
        /// <param name="latitude">Latitude of the point, in degrees.</param>
        /// <param name="longitude">Longitude of the point, in degrees.</param>
        /// <param name="levelOfDetail">Level of detail, from 1 (lowest detail)
        /// to 23 (highest detail).</param>
        /// <param name="pixelX">Output parameter receiving the X coordinate in pixels.</param>
        /// <param name="pixelY">Output parameter receiving the Y coordinate in pixels.</param>
        public static void LatLongToPixelXY(double latitude, double longitude,
                                            int levelOfDetail, out int pixelX,
                                            out int pixelY)
        {
            latitude = Clip(latitude, MinLatitude, MaxLatitude);
            longitude = Clip(longitude, MinLongitude, MaxLongitude);

            double x = (longitude + 180) / 360;
            double sinLatitude = Math.Sin(latitude * Math.PI / 180);
            double y = 0.5 - Math.Log((1 + sinLatitude) / (1 - sinLatitude)) / (4 * Math.PI);

            uint mapSize = MapSize(levelOfDetail);
            pixelX = (int)Clip(x * mapSize + 0.5, 0, mapSize - 1);
            pixelY = (int)Clip(y * mapSize + 0.5, 0, mapSize - 1);
        }

```

## Lessons and takeaways ##

By the time the hackfest ended, the team had created a capable solution that OpportunitySpace now owns. The user may sign in, view a map of the city they're registered for, and apply filters based on the year the property was built. More importantly, the solution is modular enough to be iterated upon relatively simply for future projects because the API offers many filters that can be added. 

Since Mapbox's lack of compatibility with Xamarin.Forms was a challenge, the data now received from the API has been genericized enough to no longer require it and instead renders using respective native mapping platforms.

Finally, one of the client's primary concerns was performance for this application because they've built a mobile mapping client before with web technologies, only to be plagued with user interface, performance, and speed issues. Building the app using Xamarin allowed for 
us to stay native to the respective platforms while still giving us the speed of development that made this possible during the scope of a hackfest. Additionally, a look at the screenshots shows that the controls used in the app are native to each respective platform. The iOS client looks and feels like an iOS app, and the same can be said for Windows and Android. 
