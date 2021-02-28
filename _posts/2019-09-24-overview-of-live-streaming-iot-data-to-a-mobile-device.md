---
layout: single
title: "Live streaming your IoT data to a mobile device"
title: Live streaming your IoT data to a mobile device
date: 2019-09-24
tags: ["Azure", "Xamarin", "Azure IoT"]
slug: "overview-of-live-streaming-iot-data-to-a-mobile-device"
---

Ever since I have received my [Azure IoT Devkit](https://microsoft.github.io/azure-iot-developer-kit/), I wanted to create a small app with it. The app would let me play around with streaming data from the device and view it on my phone. Also sending data to the device and many other exciting aspects of creating an IoT application. So what better way than to create a system that would stream the sensor data from my devkit to my mobile phone. Once that is implemented, how about creating an alerting system. Should a value increase over a certain threshold, the system will notify me. Many exciting ideas and paired with the capabilities of Azure all in my grasp. So before implementing, let me show you my high-level system design:

[![System Overview showing the IoT Kit, the different Azure services and the Xamarin mobile app.]({{ site.url }}{{ site.baseurl }}/images/e12e8da8-3a10-4b22-ae01-8b182cec6f48.png "System Overview showing the IoT Kit, the different Azure services and the Xamarin mobile app.")]({{ site.url }}{{ site.baseurl }}/images/f79a61a7-48c2-46d0-a1c2-cf9a696ea0eb.png)

However, first we will need to create an app on the device or emulate a device that sends up a stream of data to the Azure cloud over the [IoT Hub](https://azure.microsoft.com/en-us/services/iot-hub/). Then we will probably want to stream the data to the mobile client using [SignalR](https://azure.microsoft.com/en-us/services/signalr-service/). And finally visualise all of the data using some nice [Xamarin Forms](https://dotnet.microsoft.com/apps/xamarin/xamarin-forms) code - perhaps even a fancy charting library.

Once the data is getting streamed to the client let's have a look at alarming scenarios. What if the value e.g. the humidity rises quickly over a short timeframe. We should be able to detect such an event using [Stream Analytics](https://azure.microsoft.com/en-us/services/stream-analytics/) and then send a [push notification](https://azure.microsoft.com/en-us/services/notification-hubs/) to a client.

## SignalR vs IoT Hub

Before diving into the implementation I showed this overview to a friend of mine - to make sure I am not overdoing it with the Azure-candy store. And promptly I got the question: "Hey Mark, so what you are basically doing is a Publish and Subscribe system - why do you need SignalR and IoT Hub. Wouldn't IoT Hub or SignalR offer all you need to send messages at scale to clients?". What a great question - and simplification of the problem. While it is right that what we are doing is nothing else then publishing data and subscribing to it. They are two worlds we are combining here. On the one side we have the IoT world. Here we the creator/company/enterprise produce the software and often also distribute the hardware. So we **own** the device and software. On the other hand when we release an app we often do not own the device on which the app is running on. So while we generally tend to trust our IoT devices because we create them, harden them and equip them with certificates for communication with our backend - in our case Azure IoT Hub. We generally do not have the same trust in mobile apps. Mobile apps are often deployed to the mobile store and therefore have no guarantee that the app does not end up on a rooted Android or Jailbreaked iOS phone. Once on such a device is often just a cat and mouse game to make it as difficult as possible for an attacker to access the secrets that got installed with the app. In short IoT apps and mobile apps come from a different starting position and therefore we tend to use different approaches for implementing security. And that is why we use two pub/sub systems, because both of them were designed with the two different aspects in mind.

In the next blog post we will get started with writing the IoT client and also see how we can write an IoT client without requiring a devkit.
