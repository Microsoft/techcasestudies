---
layout: post
title:  "Building white-labeled LiveArena video-streaming apps for mobile and Xbox"
author: "Tess Ferrandez"
author-link: "https://twitter.com/TessFerrandez"
#author-image: "{{ site.baseurl }}/images/authors/photo.jpg"
date:   2017-03-28
categories: [Mobile Application Development with Xamarin, Mobile DevOps]
color: "blue"
#image: "{{ site.baseurl }}/images/imagename.png" #should be ~350px tall
excerpt: LiveArena delivers a video recording product that makes it easy, efficient, and affordable for companies to produce their own TV channels and provide on-demand videos. Microsoft partnered with LiveArena and Triona to write mobile front ends and Xbox apps using Xamarin.  
language: [English]
verticals: [Communications/Media]
---

LiveArena delivers a video-recording product that makes it incredibly easy and affordable for a company, sports club, or hotel to produce their own TV channels and provide an elegant way to deliver on-demand videos. Microsoft teamed up with LiveArena and Triona to write mobile front ends to consume client-generated video content.

## Key technologies ##

 - **Xamarin** – For iOS/Android apps.
 - **Universal Windows Platform (UWP)** – Written in C#, sharing code with iOS and Android.
 - **Xbox One** – The UWP app is customized for Xbox.
 - **HockeyApp** – Integrated into our mobile apps to get crash reports and usage analytics. It's also used to deliver test versions to stakeholders and beta testers.
 - **Xamarin Test Cloud** – For UI tests to verify that the app reaches the needed quality.
 - **Visual Studio Team Services (VSTS)** – To enable continuous integration and continuous deployment we check-in in the Git repository, build, test UI with Xamarin Test Cloud, and deliver the generated package with HockeyApp.

## Core team ##

 - Per-Erik Bemer – Head of Products, Live Arena
 - Jonas Ernestes – Senior Developer and Team Leader, Triona
 - Nico Rosenquist – Project Leader, Triona
 - Kenneth Swärd – Developer, Triona
 - Pelle Engstrand – Developer, Triona
 - [Tess Ferrandez](https://twitter.com/TessFerrandez) – Principal Technical Evangelist, Microsoft
 - [Peter Bryntesson](https://twitter.com/petbry57) – Principal Technical Evangelist, Microsoft

## Customer profile ##

**[LiveArena](http://www.livearena.com)** is a Swedish company that develops a solution that lets anyone produce traditional TV channels with live and on-demand content—streamed to anyone, on any device, at any time. Its solution is sold globally with sales representatives in Europe and North America.

Its solution consists of three main parts:

- A complete hardware kit provided to customers for recording, producing, and streaming video; there are solutions for a portable one-camera production and full studio with three cameras, slide synchronization, Skype integration, and more. 
- A comprehensive content management system where customers create TV channels, schedule live and on-demand events, and manage all the metadata for the events. The platform also allows for a mix of free content, pay-per-view content, and subscription-only content.
- A website and mobile app for consuming the content. This is where the mobile app that we developed fits in. Both the website and the mobile app are branded and white-labeled to give the end user the best possible experience.

Here is an overview of the architecture:

![Live Arena Architecture]({{ site.baseurl }}/images/LiveArena/1-Architecture.png)


Per-Erik of LiveArena describes their ambition as “LiveArena Broadcast—Change the Way You Engage.” They are democratizing TV.

**[Triona](http://www.triona.eu)** is a Swedish IT consulting firm with nine offices in Sweden and Norway. It is LiveArena's preferred consulting partner and it does much of LiveArena's back-end development as well. Its main expertise is web development using ASP.NET.

## Problem statement ##

When Microsoft met with the customer, mobile development already had started. They were using Xamarin and C# to develop iOS and Android versions. They shared code via a portable class library. They had not started development on the UWP version, and they didn’t have a DevOps strategy yet. One requirement they had was that the published apps should be *white-labeled/branded*—that is, tailored for the customer to consume just this brand's content. They also wanted to publish the app in the brand's name.

During the initial hackfest, we noticed that no architectural pattern was applied to the code. In our experience, using a pattern like **Model-View-ViewModel (MVVM)** is beneficial to structure and to share as much code as possible for the UI. MVVM is the preferred pattern for UWP apps, and since they wanted to develop a UWP version as well, we suggested that they restructure the app using the MVVM pattern. 

We also performed a value stream mapping (VSM) exercise for how they should develop their white-labeled apps and devised a DevOps strategy to automate the build, test, and distribution. We decided to:

- Use VSTS for continuous integration (CI). 
- Use HockeyApp for continuous delivery (CD).
- Use Xamarin Test Cloud for continuous testing (CT). 
 
We also helped in setting up a branching strategy so that they could develop and test internally and then automatically start the DevOps pipeline for all the white-labeled apps. In comparison with the manual way they were delivering their apps to their customers, this was a major improvement, both in time and in quality.

## Solution overview ##

Triona developers had written the API and websites and they were the main developers for the apps. We teamed up with them to help with the mobile apps and the DevOps process.

To make the development process as quick as possible, they decided to use Xamarin. The aim was to share as much code as possible. This would mean the apps would also be easier to maintain.

To generate the white-labeled apps on demand, we set up the DevOps process as follows, using:

- **Git** for source control and to create a new branch for each white-labeled app.
- **Visual Studio Team Services** to build the apps.
- **Xamarin Test Cloud** for automated UI tests.
- **HockeyApp** to package and distribute the app to testers.

For the walkthroughs, we would be looking at the LIF-TV app (Leksands IF hockey team).

### Application walkthrough iOS and Android ### 

When you open the app, you see an area for showing videos, a list of upcoming live shows, and a list of on-demand shows. What is initially shown in the video area is the current content for the channel. The channel is like a TV channel with scheduled live and on-demand content. For LIF-TV, watching the channel and some of the other content requires a subscription. By tapping the "hamburger" menu (the menu icon) and providing the login credentials, we can log in.

![Mobile screens]({{ site.baseurl }}/images/LiveArena/2-mobile1.png)


Once we are logged in, the channel shows either the currently broadcasting program or a program listing, if no show is currently scheduled. If you want to watch any of the shows listed, you simply tap the show. In this case, we tapped a live show that is scheduled in the future, so we saw a nice countdown timer instead of the video. You can also search for live and on-demand content by tapping the hamburger menu and entering your search phrase in the search box.

![Mobile screens]({{ site.baseurl }}/images/LiveArena/3-mobile2.png)


By tapping the TV Guide button, you get a multiday program listing where you can look for the show you are interested in. Tap the TV Guide button again to hide the program listing. If playing a video, it runs by default in the video section, but you also can choose a full-screen video.

![Mobile screens]({{ site.baseurl }}/images/LiveArena/4-mobile3.png)


The actual look of the app will vary slightly between different versions of the white-labeled apps. Accent color, if there is a channel, and of course the actual content are some of the things that will change.

### Application walkthrough UWP ###

When you open the app, you will see a listing with live and on-demand videos. If the content provider has chosen to have channels, you can find them in the live area. Channels are a way to schedule multiple shows one after another, like TV channels. 

![UWP screens]({{ site.baseurl }}/images/LiveArena/5-UWPMainView.png)


If you select a show, it will start playing in the top section so that you can preview it while continuing to browse. You also can choose to display the video in full screen.

You can view shows by category:

![UWP screens]({{ site.baseurl }}/images/LiveArena/6-UWPCategoryView.png)


Or you can search for shows by name:

![UWP screens]({{ site.baseurl }}/images/LiveArena/7-UWPSearchView.png)


### Application walkthrough Xbox ###

Halfway through the project, a new idea emerged. LiveArena and the consumer group at Microsoft came up with a new business opportunity. The idea was to bundle LiveArena subscriptions with Xbox. Leksands IF (LIF) in this case could offer its customers an Xbox to rent together with a LIF-TV subscription.

With this deal in place, the app must work great on the Xbox as well. 

Since we built a UWP, it will work on the Xbox out of the box, but that is not enough. While most of the controls work great on the Xbox, the navigation and interaction patterns are quite different.  

UWP apps will automatically work on Xbox since Xbox is running Windows 10 and the UWP controls have built-in support for the Xbox controller. On an Xbox, the main input method is a controller. The Xbox screen (TV) is often placed more than 10 feet away and the room is often dark when the Xbox is in use.

These are all factors to consider and make adjustments for so the app feels as if it is built for Xbox.  

![XBox screens]({{ site.baseurl }}/images/LiveArena/8-XboxMainView.png)


Here are a few examples of things we changed when the app runs on Xbox:

- The background is black because the room is often dark when you watch TV. 
- Everything is bigger because the content is farther away.
- The "player" on top is now a preview image. Instead, you immediately go to full screen when you select a show. 

## Solution, steps, and delivery ##

### Basic architecture and code reuse ###

The finished solution consists of seven projects:

- **LiveArena.Core** – Portable Class Library (PCL) containing all services, encryption, and player-specific code.
- **LiveArena.Providers** – Interfaces for device-specific code such as INetworkStatusProvider or INavigationStackProvider.
- **LiveArena.ViewModels** – Interfaces and implementations of the view models.
- **LiveArena.UWP** – UWP head.
- **LiveArena.iOS** – iOS head.
- **LiveArena.Droid** – Android head. 
- **LiveArena.UITests** – UI tests for iOS/Android; for running in Xamarin Test Cloud.

We used native UIs for the different platforms. By using the MVVM pattern, we could minimize the code needed for each platform project.

There is code for each platform—mainly around the media player. Playing media is the main feature of this product. All videos are hosted in Azure Media Services and protected with AES encryption. Finding a solution for playing encrypted media on the UWP platform was pretty straightforward, but doing it on iOS and Android was challenging. It proved to be the biggest technical hurdle in the project.

### MVVM structure and support for offline caching ###

We used the MVVM (Model-View-ViewModel) pattern to structure the code for the application. In this case, we based the structure on **MVVMLight** because we need it to work cross-platform.

The main goals we tried to accomplish were: 

- Focus on the design and user experience rather than the API capabilities when building the views.
- Clean separation of views and view models so we can move as much code as possible to the shared library.
- Provide easy-to-use offline caching.

**Design-first approach**

We used a very strict design-first approach for two reasons: 

- It ensures that we focus on the customer experience rather than the API capabilities when we build the views.
- It allows us to implement the design and the view model independent of one another.

Most users are already familiar with the website and how to use it. We decided to take advantage of this and reuse a lot of the design so the users would feel comfortable when using the app.

*Search on the website*

![LIF search web view]({{ site.baseurl }}/images/LiveArena/9-LIFWebView.png)


The first step was to sketch all the views. We used the sketches to discuss the interaction and details with the designers at LiveArena.

*SearchView sketch*

![UX sketch]({{ site.baseurl }}/images/LiveArena/10-SearchViewSketch.jpg)


When the sketch was complete, we started identifying all the data we needed to build the view. Anything that was visible on the screen at any point became a property in the view model.  

As far as it was possible, we tried to make all view model properties strings. For example, if we are to display a time, we store the actual displayed text in the property, rather than a DateTime. This way we can avoid using converters that have to recalculate the strings on the UI thread.

We also identified any actionable elements and added the commands to the view model. This way we could bind to it and avoid adding code to the code-behind. 

Given the sketch above, the *ISearchViewModel* then looks like the following:

    public interface ISearchViewModel
    {
        string SearchString { get; set; }
        ObservableCollection<EventItemViewModel> AllResults { get; }
        ObservableCollection<EventItemViewModel> LiveNowResults { get; }
        ObservableCollection<EventItemViewModel> UpcomingResults { get; }
        ObservableCollection<EventItemViewModel> VODResults { get; }
        string NoResultsString { get; }
        string NoLiveResultsString { get; }
        string NoUpcomingResultsString { get; }
        string NoVODResultsString { get; }
        RelayCommand ToEventCommand { get; }
        RelayCommand DoSearchCommand { get; }
        Task<bool> DoSearch();
        event SearchComplete SearchComplete;
    }

The *ISearchViewModel* will have two implementations: 

- *DesignTimeSearchViewModel* with mock data that we can use at design time.
- *SearchViewModel* populated with real data from the server.

Since we have separate design-time and runtime data, we can now work on design and data gathering in parallel.

The other benefit of implementing a design-time view model is that we can design the application in the designer. We don’t even have to compile to see the changes.

When it comes time to develop the runtime view model, we save time since we know exactly what we need to implement. Otherwise, it is easy to start implementing nice-to-haves that are never used.

*SearchView – final look and feel*

![Search view final]({{ site.baseurl }}/images/LiveArena/11-SearchViewFinal.png)


**Clean separation of views and view models**

To share as much code as possible, we placed the view models in the portable class library. That way we write and maintain them in one single place. In code-behind, we allow only very view-specific code, such as custom list scrolling.

A few things must happen for this to be possible:

- We replace event handlers with **RelayCommands** and bind the controls events to the commands.
- To navigate to different views in the commands, we define an *INavigationProvider* interface in the shared project. The *NavigationProvider* is then implemented in each of the three apps. (Same for *DispatcherProvider*, *NetworkStatusProvider*, and so on.)
- In the UWP project we create a *PageViewBase*—deriving from *Page*—that passes page-parameters over to the view model.

The base view models have many nice helpers such as:

- **IsBusy** – Used to display spinners when the view is busy gathering data. Our implementation allows for multiple concurrent requests with AddBusy/ReleaseBusy.
- **IsOnline** – Toggles as NetworkStatus changes.
- **IsActive** – Used to determine if the view is visible.
- **UpdateCollection\<T>(oldCollection, newList)** – Updates an observable collection by only modifying updated items. This is to avoid flicker in lists or grids. 
- **SaveAsync\<T>(string name, T data)** – Stores data in cache.
- **\<T>LoadAsync\<T>(string name)** – Loads data from cache.

A ViewModel for a view (for example *SearchViewModel*) can derive from one of two classes: 

- *AppViewModelBaseWithParameter\<T>*
- *AppViewModelBaseWithoutParameter*

It depends on whether the view has parameters or not.

View models for subitems, such as the *EventItemViewModel*, can derive from *AppViewModel* base class. This is more lightweight and doesn't include any of the navigation and populate methods discussed in the offline caching section.

For a reference implementation of this architecture, browse the **[MyWeather](http://Github.com/TessFerrandez/MyWeather)** sample.

**Offline caching**

While you can't stream videos in this app when the app is offline, it is still nice to have offline caching. Caching results offline means the app looks and feels much faster since we populate the views very quickly.

The base view model has two methods that the view models can implement to enable offline caching:  

- **Populate(populatereason, parameter)** – Load data from the cache into the view.
- **OnRequestAsync(parameter)** – Call the service to get data, repopulate the view, and store the data in the cache.

When a user navigates to a view, the following will happen:

1.	Populate is called to populate the view from cache if the cache contains data.
2.	OnRequestAsync is called to get data from the server. If successful, it then stores the data in the cache for future calls to Populate. 
3.	Populate is called again with PopulateReason.DataLoaded in case we want to do any post-processing.

**Use of MVVM in iOS/Android**

Using the MVVM pattern is very natural in UWP and Xamarin.Forms since both these platforms support data binding by default. For iOS/Android, we have to set up data binding manually using **MVVMLight**. To set this up, we:

- Implement a static view model locator class so that we can find a concrete implementation of a view model and other platform-specific implementations.
- Manually add code to bind the UI. 

To illustrate how data binding works in Android code, here is a sample taken from the **[MyWeather](http://Github.com/TessFerrandez/MyWeather)** project (iOS data binding looks similar). 

In the App.cs file, we added the following:

    namespace MyWeather.Droid
    {
        public static class App
        {
            private static ViewModelLocator _locator;
            public static ViewModelLocator Locator
            {
                get
                {
                    if (_locator == null)
                    {
                        // first time initialization
                        var nav = new NavigationService();
                        nav.Configure(ViewModelLocator.WeatherPageKey, typeof(WeatherActivity));
                        SimpleIoc.Default.Register<INavigationService>(() => nav);

                        // add other providers here
                        SimpleIoc.Default.Register<IDispatcherProvider, DispatcherProvider>();
                        SimpleIoc.Default.Register<INetworkStatusProvider, NetworkStatusProvider>();

                        _locator = new ViewModelLocator();
                    }
                    return _locator;
                }
            }
        }
    }

This code creates the needed singleton classes by registering them with our IOC container. It also registers the view model+activity connection with the navigation service so that we can navigate to the right page by referring to the view model. 

Here is the MainActivity.cs file:

    using Android.App;
    using Android.OS;
    using MyWeather.Core.ViewModels;
    using GalaSoft.MvvmLight.Helpers;
    using System.Collections.Generic;

    namespace MyWeather.Droid
    {
        [Activity(Label = "Weather", MainLauncher = true, Icon = "@drawable/icon")]
        public partial class MainActivity
        {
            //trick the linker into believing we use events
            // See https://developer.xamarin.com/guides/android/advanced_topics/linking/
            private static bool _falseFlag = false;
            //save the bindings to avoid garbage collection
            private readonly List<Binding> _bindings = new List<Binding>();
            public IHomeViewModel Vm { get { return App.Locator.HomeViewModel; } }
            protected override void OnCreate(Bundle bundle)
            {
                base.OnCreate(bundle);
                SetContentView(Resource.Layout.Main);

                GetWeatherButton.SetCommand(Vm.ToWeatherCommand);

                //save the binding to avoid garbage collection
                _bindings.Add(this.SetBinding(
                    () => Vm.SearchLocation, () => LocationEditText.Text, BindingMode.TwoWay));

                if (_falseFlag)
                {
                    GetWeatherButton.Click += (s, e) => { };
                }
            }
        }
    }

In this file, we note a couple of things. The *_falseFlag* is a documented workaround to get this to work. We need to trick the compiler to believe that we are using events. [It’s documented here.](https://developer.xamarin.com/guides/android/advanced_topics/linking/) 

In the *OnCreate* method, we map the weather button to a *RelayCommand* property in our view model. Whenever a user taps that button, this command is executed instead of the *Click* event. 

Via the *SetBinding* method, a specific property in the view model is bound to a specific property on a UI element. This means this control’s property is populated from the data in the view model. In this case the binding is two-way, which means that changes in the control are reflected back to the view model.

### Support for Xbox ###

Xbox One runs Windows 10. Any UWP apps built using the Anniversary Update SDK (or later) will work on the Xbox as is. To take the experience from “it works” to "great," we needed to optimize for the gamepad and rework the layout and navigation to work better in a living room. 

These are the steps we followed to customize the app for the Xbox:

1.	Set up the development environment.
2.	Optimize for the gamepad.
3.	Rework the layout and navigation.
4.	Implement TV-safe colors and TV-safe areas. 

**Set up the development environment**

An effective way to get started is by watching the videos from [the App Dev on Xbox event](https://developer.microsoft.com/en-us/windows/projects/campaigns/app-dev-on-xbox-event). These tell you everything you need to know about setting up your environment and your Xbox One and designing for Xbox.

While testing on the Xbox is optimal, you can do a lot of the development and testing on the PC. It is, after all, just a UWP app.  

You can technically test your app using the keyboard because the keyboard actions are very similar to the gamepad. The best way to test, however, is to attach a USB cable to your regular Xbox One gamepad and connect it to the computer.

We used a lot of concepts from [the presidents demo](https://Github.com/Microsoft/uwp-experiences). It shows nicely how you can set up your app on the PC to emulate being on the Xbox. 

When you click the Xbox button, it will:

-	Go to full screen and set the size to 960x540.
-	Set the theme to dark.
-	Turn on sound for control actions.
-	Change to a style sheet with TV-safe colors.

It also contains a nice, handy feature that will show the TV-safe areas when you press F1 or Gamepad.X so that you can make sure you are within the bounds of the TV-safe area.

**Optimize for the gamepad**

The first time you deploy your app to the Xbox One you might be a bit surprised. You will see a mouse cursor on the screen, and to navigate your app you need to move the mouse cursor. This is called mouse mode, and it is the default experience for UWP apps on Xbox. The reason why this is default will probably become obvious later. Most apps have some areas/controls that are unreachable with directional navigation.

To switch to directional navigation (what most users are expecting on Xbox), you just have to add one line of code to the app constructor:

    App.Current.RequiresPointerMode = ApplicationRequiresPointerMode.WhenRequested; 

Controls such as TextBoxes, Buttons, ListViews, GridViews, and Pivots are already built to handle gamepads, so if you use the built-in controls you are off to a great start.

When you work with mouse or touch, it is very easy to see where you are pointing, or what you are interacting with. When you use a gamepad (or keyboard), however, it is very important to show where the focus is. The built-in controls have focus visuals as well as sound to let the users know what is going on.

![Xbox Focus]({{ site.baseurl }}/images/LiveArena/12-XboxFocus.gif)


Sometimes, though, you will notice that you completely lost track of where you are. When this happens, it is more than likely because you are focusing on something that is not visible on screen.  

In cases like that, this piece of code is very helpful to debug focus and identify items that should be collapsed when off-screen. Or to debug buggy tab-orders. 

    #if DEBUG
                //Debugging helper for visual focus
                GotFocus += (object sender, RoutedEventArgs e) =>
                {
                    FrameworkElement focus = FocusManager.GetFocusedElement() as FrameworkElement;
                    if (focus != null)
                        Debug.WriteLine("got focus: " + focus.Name + " (" + focus.GetType().ToString() + ")");
                };
    #endif

**Rework the layout and navigation**

We had already given the UWP app a nice, responsive design that worked all the way from phone to high-res desktops, so a lot of the work with the layout was already done.

The effective working size for apps on the Xbox is 960x540 (1920x1080 with 200% scaling applied). When you first launch your app on the Xbox, you will notice that everything looks really, really big. This is, of course, because you normally look at apps on the Xbox at TV distance. Keep that in mind while you work with the layout and design, so that you continuously test the app from 10 feet away.

***Size, content and layout***

The first thing we had to modify for Xbox viewing was the amount of data to be displayed. We removed anything nonessential, such as timestamps on videos. (Whenever you design for TV, whether it be a PowerPoint presentation or an Xbox app, you need to think like the renowned fashion designer Coco Chanel, who advised that when going out, remove everything you don’t need, and then remove one thing more.) The cleaner and more to the point you can be, the better the viewing experience.

***Navigation***

Next, we had to modify the navigation. When you work with mouse or touch, you can scatter the controls on the page and you can still reach each control with just one tap. When you work with gamepads (or keyboards) that just allow left/right, up/down, you need to make sure that all your controls are reachable.

![The button in the middle is inaccessible]({{ site.baseurl }}/images/LiveArena/13-InaccessibleUI.png)


You also need to make sure you can reach the controls easily and quickly. For example, having to scroll through a list of 500 items to reach a button at the bottom of the screen becomes tedious fast.  

Another problem is focus trapping. If you place a button after a slider, you can’t access the second button without sliding the slider to its maximum value. The same thing applies with maps and other similar controls.

![You can't reach the button on the right]({{ site.baseurl }}/images/LiveArena/14-FocusTrapping.png)


For more on this, read [Designing for TV](https://docs.microsoft.com/sv-se/windows/uwp/input-and-devices/designing-for-tv).

In our case, we have a video player at the top of the screen. The video will autostart when you select a video in a list.  

There were a few issues with this:

-	When you move right from the menu, you will immediately move into the small controls on the player and have to move by them to get to the list of videos.
-	If you select a video and then want to pause the video or maximize it, this becomes impossible. Whenever you move right/left or up, you will end up selecting another video.
-	If you scroll through the list of videos quickly, you will start streaming a lot of videos and the app will feel like it is lagging.

We resolved this by changing the navigation pattern on Xbox. Rather than showing a video in the video player, we just showed a preview image. When you click A on the gamepad, you will then navigate to a full-screen version of the video. This navigation is commonly used for TV apps on Xbox and other TV app providers, so it feels very natural to users.

We also worked a lot with minimizing the amount of “clicks.” Wherever we had more than one actionable item on the screen (buttons, textboxes, and so on) we tried to see which ones we could remove and then group the others together. 

For each screen, we also carefully picked the tab order and made sure to focus on the most common actions when moving right from the menu bar.

**Implement TV-safe colors and areas**

When you use your Xbox or TV, the room will often be dark. If your app has a white background, the background will stand out more than the content. It will also tire your eyes after a while. Therefore, it is recommended to use a dark theme for Xbox apps. We decided to go with black because the app on the phone and desktop is white.

***TV-safe colors***

TVs don’t handle extreme intensities well, such as full white or full black. In general, you should try to use only colors with RGB values between 16-235 (0x10-0xEB) to avoid color weirdness.  

The Presidents app that we referred to earlier contains a style sheet with TV-safe colors for the most common system colors. You need to make any custom colors you use TV-safe as well, including colors on images and other assets.

***TV-safe areas***

When you launch your app on the Xbox the first time, you may notice that it looks very boxed in.

![TV Safe area]({{ site.baseurl }}/images/LiveArena/15-TVSafeArea.png)


Not all TVs show all the content that is at the edges. Because of this, your app will automatically have a TV-safe area added—48 pixels from the left and right edge and 27 pixels from the top and bottom edge. While this is nice, it looks a bit weird when you scroll through a list or grid, and the content is abruptly cut off.  

To use the whole screen, you can add this code to *OnLaunched* in *app.xaml.cs*:

    ApplicationView.GetForCurrentView().SetDesiredBoundsMode(ApplicationViewBoundsMode.UseCoreWindow);

You still need to respect the TV-safe area, though. Anything that users need to see or interact with should be inside of the TV-safe area. However, GridViews and ListViews can extend beyond so they scroll on the whole screen.  

We added a Footer template with a 27-pixel margin to all our lists to make sure that the bottom item was also completely visible when scrolling to the end of the list.

If you use the Presidents sample, you can toggle the TV-safe area on and off to make sure that your content fits well within the area.

*Many of the images in this section were taken from the [Designing for Xbox and TV](https://docs.microsoft.com/sv-se/windows/uwp/input-and-devices/designing-for-tv) article.*

### DevOps strategy ### 

LiveArena needs to produce white-labeled apps for all the platforms. Today, all this work is done manually and locally on a developer's computer. This is cumbersome and very error-prone. To enhance this process, we performed a value stream mapping exercise and concluded that:

-	We need a separate branch for each white-labeled app. Naming convention will be release-\<brand>. (More on branching strategy below.)
-	We identified all the files that we had to change to build each white-labeled app. This list was different for each platform.
-	We integrate with HockeyApp to automatically provide apps to stakeholders and beta testers. This will require us to have three HockeyApp apps per white-labeled app.
-	We use Xamarin Test Cloud to get basic testing done after each build. Xamarin Test Cloud supports only iOS and Android at this time.
-	We use VSTS to build a CI flow from check-in in Git. Here we also need to add three build definitions, one per platform.

One issue we discussed a lot was whether we should write a tool to configure all of the above programmatically. This tool would probably be a PowerShell script to perform the steps above. Ultimately, time constraints forced us to postpone that feature. However, by documenting what steps we need to take to produce the white-labeled apps, we have set the stage for such a tool later.

### Branching strategy ###

The customer has adopted a branching strategy similar to the one proposed by [Vincent Driessen](http://nvie.com/posts/a-successful-git-branching-model/). They work in the developer branch and when they are ready to release they create a release branch with the version number. 

We knew we needed a separate branch for each white-labeled app, but we considered whether that branch should include a version number as well. In the end, we decided that the branch shouldn’t have a version number. We always want the white-labeled app to be on the latest version of the base code. White-labeled customizations are version-agnostic. 

The branching strategy is as follows:

1. Implement new features and/or bug fixes in the developer branch.
2.	Create a release-\<version> branch and start testing against the internal test environment.
3.	Merge the release-\<version> branch into each of the white-labeled app release-\<brand> branches to trigger a continuous build and distribution to external testers and stakeholders.

### Identification of white-label-specific details ###

One important part of a new white-labeled app is to identify everything that must be changed. This analysis must cover all three platforms. We can divide this into three categories:

- App configuration
-	Manifest-related information
-	Image resources

**App configuration**

App configuration is placed in a config.xml file that is embedded in the LiveArena.Core portable class library. In there we specify three settings that are specific for each brand:

-	**BaseURL** – Denotes the URL for this brand’s back-end API. The base URL is in the format `https://xxx.play.livearena.com`.
-	**PrimaryColor** – The accent color used in the UI.
-	**SecondaryColor** – The background color used in the UI.

Because this file is embedded in the portable class library, we only need to change it once.

**Manifest-related information**

All three platforms have the notion of a manifest, which declares information about the app to the system. It includes required permissions, links to resources, and properties such as the publisher and app name. The only thing we need to change for each branded app in the manifests are the app name and possibly the publisher. Even though we need to change image resources, we don’t need to change the manifest itself for that. The following is a detailed list of what needs to be changed for each platform.

***iOS*** 

In *info.plist* we need to change the following values:

-	CFBundleName
-	CFBundleIdentifier

***Android***

In *AndroidManifest.xml* there is a property in the application object that sets the app.name that needs to be changed. The actual string is in *Resources/values/string.xml*. 

***UWP***

Here we need to modify the *Package.appxmanifest* file. There are some properties we need to change:

-	Identity
-	DisplayName
-	PublisherDisplayName

**Image resources**

We need to change three types of image resources: 

-	Splash screen images
-	App icons
-	Logo in login screen

A lot of these files come in different resolutions. Here we have identified an opportunity for improvement. We should find or implement a tool that takes a vector image (or a large bitmap image) and generates all the different resolutions needed. We have such tools for UWP development, but we need a cross-platform version of this. Triona will look into this in the near future.

### HockeyApp integration ###

We decided to integrate HockeyApp into the apps for two reasons: 

-	Get crash reports and analytics.
-	Use HockeyApp to distribute our apps to stakeholders and testers. 

When you add an app to HockeyApp, you need to add one app per platform, so in our case we will get three HockeyApp apps per white-labeled app because we support three platforms: iOS, Android, and UWP (Windows). The workflow for adding an app on the different platforms is very similar. The following describes how it’s done for iOS:

1.	Go to the [HockeyApp web page](http://rink.hockeyapp.net). We log in using LiveArena’s account and click the **New App** button. In the popup dialog, you can start with a built package if you want. We selected the **Create the app manually instead** link. 

    ![New Hockey App]({{ site.baseurl }}/images/LiveArena/16-Hockey1.png)
    
2. Select the iOS platform. The title is the name of the white-labeled app (LIF-TV). For the bundle identifier, we chose to append the brand name after the ordinary bundle name as a pattern to follow. After filling this out, our entry looks like this: 

    ![Create App]({{ site.baseurl }}/images/LiveArena/17-Hockey2.png)

3.	Clicking **Save** shows the final screen contains the _**app id**_ (which is very important). Here you can manage your HockeyApp. We’ll discuss shortly exactly how you integrate HockeyApp with your Xamarin project.

    ![Options and instructions]({{ site.baseurl }}/images/LiveArena/18-Hockey3.png)
    

4. In Visual Studio, we reference the NuGet package _**HockeySDK.Xamarin**_. 

    ![HockeySDK.Xamarin]({{ site.baseurl }}/images/LiveArena/19-VSNuget.png)
    

5. In the *AppDelegate.cs* file, add a reference to the correct namespace and add the following code to the *FinishedLaunching* function. 

        using HockeyApp.iOS;

        …
        public override bool FinishedLaunching (UIApplication application, NSDictionary    launchOptions)
        {
                    var manager = BITHockeyManager.SharedHockeyManager;
                    manager.Configure("YOUR_APP_ID");
                    manager.StartManager();
                    manager.Authenticator.AuthenticateInstallation();
                    return true;
        }

    In the call to *manager.Configure*, we supply the app ID we received when creating the HockeyApp iOS app. This code will give us crash dumps and basic analytics. It will also verify that the user has a right to run the app.

    In fact, we caught one such bug when there was a version mismatch of the back end that caused one of the API calls to fail. By analyzing the crash dump from HockeyApp, we understood what the problem was and fixed the back end.

The workflow for Android and UWP are similar. The differences are outlined below. 

For Android, create a new HockeyApp and include the same NuGet package. Instead of modifying the *AppDeletegate.cs* file, we modify the file *MainActivity.cs*.

        using HockeyApp.Android;
        …
        protected override void OnCreate(Bundle savedInstanceState)
        {
        base.OnCreate(savedInstanceState);
                    CrashManager.Register(this, "YOUR_APP_ID");
        …

That's where we supply the app ID for the HockeyApp Android, of course. 

For UWP, create another HockeyApp app (and target Windows). Since it’s Windows (UWP), we shouldn’t target the Xamarin version but instead reference _**HockeySDK.UWP**_ package. Here we modify the *app.xaml.cs* file.

        using Microsoft.HockeyApp;
        …
        public App()
        {
                    this.InitializeComponent();
                    this.Suspending += OnSuspending;
                    Microsoft.HockeyApp.HockeyClient.Current.Configure("YOUR_APP_ID");
        }

After configuring HockeyApp for all three platforms, the dashboard looks like this:

![Hockey Apps for all three platforms]({{ site.baseurl }}/images/LiveArena/20-Hockey4.png)


### Xamarin Test Cloud ###

**[Xamarin Test Cloud](https://testcloud.xamarin.com)** is a service that enables automated UI testing on thousands of different Android and iOS devices.

After we showed the customer what can be done with Xamarin Test Cloud, they were very excited to use it. They wanted it for two reasons:

-	Doing sanity testing on new builds just to make sure this white-labeled app still looks correct.
-	Doing coverage testing on a variety of devices and testing on specific devices when a customer reports a problem.

We decided to do a couple of basic UI tests so that we got a baseline of the quality. A lot of the functionality of the app has to do with playing videos and interacting with them. That type of functionality isn’t easily tested with a UI test, so we covered a more basic set. To author the test, we used the integrated UI test recorder in Visual Studio. It works very well with the Android emulator and produced test code that we edited for extra clarity. The test code for Android looks like this:

        [Test]
        public void LoginTest()
        {
            app.Screenshot("Start");
            app.Tap(x => x.Marked("Opened"));
            app.Screenshot("tray menu");
            app.Tap(x => x.Id("leftdrawer_itemtitle"));
            app.Screenshot("login screen");
            app.Tap(x => x.Id("username"));
            app.EnterText(x => x.Id("username"), "xxxxxxxxxxxx");
            app.Tap(x => x.Id("password"));
            app.EnterText(x => x.Id("password"), "xxxxxxxxx");
            app.Screenshot("before login");
            app.Tap(x => x.Id("loginBtn"));
            app.Screenshot("after login");
        }

        [Test]
        public void ProgramListTest()
        {
            app.Screenshot("Start");
            app.Tap(x => x.Id("programListShowBtn"));
            app.Screenshot("program list");
            app.Tap(x => x.Id("toggleProgramList"));
            app.Screenshot("Start screen");
        }

        [Test]
        public void SearchTest()
        {
            app.Screenshot("Start");
            app.Tap(x => x.Marked("Opened"));
            app.Screenshot("tray menu");
            app.Tap(x => x.Id("search_src_text"));
            app.EnterText(x => x.Id("search_src_text"), "15:00");
            app.Screenshot("search text");
            app.PressEnter();
            app.Screenshot("search result");
            app.Tap(x => x.Marked("Closed"));
        }

Since the integrated UI test recorder didn’t work with iOS projects, we used the standalone UI test recorder on OS/X to generate similar tests. 

When running Xamarin Test Cloud from the continuous integration pipeline, we tested with a few devices. For manual coverage tests, the customer will do it from the Xamarin Test Cloud portal.

Here is an example of how it looks in Xamarin Test Cloud when a test has been run:

![Xamarin Test Cloud Run]({{ site.baseurl }}/images/LiveArena/28-XTC.PNG)


### Continuous integration using VSTS ###

Having a well-defined continuous integration is vital to the customer. They want the process to be as automatic as possible, using check-ins as the trigger point. We discussed whether we should include release management and automatic upload to the stores but decided against this for the moment. We also tried out the new Visual Studio Mobile Center product. It looks really promising and streamlined for our type of continuous integration. As of this writing, Visual Studio Mobile Center does not yet support UWP. Additionally, Visual Studio Mobile Center is still in preview so we decided against using it. We do believe it will be the best option in the future.

To set up continuous integration for a white-labeled app, we performed the following steps:

1.	Create a new Visual Studio Team Services project called ‘LIF-TV’. Click the **Builds** tab and start adding our build definitions. We start with the Android version and chose the Xamarin-Android template.

    ![Visual Studio Team Service project]({{ site.baseurl }}/images/LiveArena/21-CI1.png)


    ![Visual Studio Team Service project]({{ site.baseurl }}/images/LiveArena/22-CI2.png)


2.	We removed the Xamarin Component Restore task and customized the rest of the settings, so we got a setup like this:

    ![CI Worksflow]({{ site.baseurl }}/images/LiveArena/23-CI3.png)


One thing that caused us some problems is that the JDK version by default is not JDK 8, which is the version we need. Remember to change this if you get version errors on the Java compiler. The setting is near the bottom and looks like this:

![JDK Options]({{ site.baseurl }}/images/LiveArena/24-JDK.png)


If you get a Java compiler out of memory error, you need to increase the Java Max Heap Size setting to 1 GB by changing the value in the properties windows for the LiveArena.Droid project. When done correctly it looks like this:

![Java Max Heap Size]({{ site.baseurl }}/images/LiveArena/25-VS.png)


The continuous integration process:

-	Gets the latest source code from the release-LIF-TV branch in GitHub.
-	Restores needed NuGet packages.
-	Builds the Android project.
-	Builds the UI test project with our UI tests.
-	Runs the tests in Xamarin Test Cloud.
-	Signs the .apk file and publishes the generated files to the staging directory.
-	Distributes the build to testers via HockeyApp.

The setup for iOS/UWP looks similar. For UWP we omitted the UI test and Xamarin Test Cloud tasks since Xamarin Test Cloud supports only iOS/Android at this time. The complete builds list then became:

![Build Definitions]({{ site.baseurl }}/images/LiveArena/26-CI4.png)


It’s tricky to get all the parameters correct in all build definitions. We found [the Visual Studio guidance for building Xamarin apps](https://www.visualstudio.com/sv-se/docs/build/apps/mobile/xamarin) very helpful.

To build the iOS apps, we needed a Mac. Because the customer wanted this deployed entirely in the cloud, we decided to use [MacinCloud](http://macincloud.com) to rent a Mac. They have a special [VSTS Build Agent Plan](https://www.macincloud.com/pricing/build-agent-plans/vso-build-agent-plan) that suits us perfectly.

We set all the build definitions to trigger on check-ins in the repository. The customer is very pleased that no manual intervention is necessary. This will be even more important when the number of white-labeled apps grows.

## Conclusion ##

The purpose of this project was to build a mobile app for the LiveArena video streaming platform. The company's two main goals were to share as much code as possible between the platforms and have an efficient way to develop white-labeled apps for their customers. During development, a business opportunity with an app for the Xbox One came up.

We delivered on all of this. We developed the iOS/Android mobile apps using Xamarin, C#, and .NET. We used MVVM as a pattern so that we could share as much code as possible. We developed the UWP version with responsive design and did special work for it to look good on an Xbox One.

We also set up their DevOps pipeline so they could handle their white-labeled apps as efficiently and automatically as possible. 

- We took advantage of VSTS to get continuous integration (CI) from check-ins in Git. 
- We used HockeyApp to get continuous distribution (CD) to stakeholders and beta testers.
- We used Xamarin Test Cloud to get continuous testing (CT). 

This DevOps pipeline has been a great improvement. Not only do they save a lot of time by not having to do these tasks manually, but the quality of the builds has improved, both by not doing this by hand and by using Xamarin Test Cloud to do UI tests of the app before sending it to the customer. 

## General lessons ##

We think this project shows that Xamarin is a very effective way to develop cross-platform mobile apps, especially when using a pattern such as MVVM to minimize the code we need to write per platform.

We have also shown that writing for UWP with responsive design does not require much additional work if you consider it from the beginning. Here MVVM helps as well. Our decision to use design-time data to help in designing the UX accelerated this process.

While UWP apps work out of the box for Xbox, some thought must be put into the design. Most of the work, however, is pretty standard, like adding safe areas and changing the colors. The difficulties lie around customizing the layout, navigation, and commanding. 

The DevOps pipeline we produced turned out great and is essential for this project. There are lots of moving parts here and we learned much by setting this up. We think the upcoming Visual Studio Mobile Center will simplify this greatly. [MacinCloud](http://macincloud.com) solved the problem of producing automated DevOps builds for iOS at an attractive price.

## Opportunities going forward ##

We wrote this version of the mobile apps with native UI on the different platforms. The customer wants to move to Xamarin.Forms; we have not yet evaluated how well this works for Xbox. The team is also considering writing a tool to automate generating the projects for a new customer. They will also investigate time in Xamarin Test Cloud to do more UI tests. Finally, the team will closely monitor the progress for Visual Studio Mobile Center. When it’s ready, they will probably shift to using that.
