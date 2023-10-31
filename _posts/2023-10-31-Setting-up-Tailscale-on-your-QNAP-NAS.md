---
layout: single
title: "Setting up Tailscale on your QNAP NAS"
date: 2023-10-31 12:03:00
tags: ["VPN", "Home Automation"]
slug: "tailscale-qnap"
---

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/2023-10-31-DALL-E-VPN.png" alt="2023-10-31-DALL-E-VPN" style="zoom:50%;" />

By discovering [Tailscale](https://tailscale.com/), I finally found a solution for my [QNAP NAS](https://tailscale.com/) as a VPN gateway into my home network. I have a couple of devices at home that do not talk to some cloud service but provide services when accessed through my home network. Whenever I am not at home, accessing said services is no longer possible. Let's examine how Tailscale enables accessing devices on the same network as your QNAP NAS.

<!-- expand -->

Tailscale is a VPN (Virtual Private Network) service that enables devices to communicate securely with each other. You must set up an account by following the steps described [here](https://tailscale.com/download). You can install a client on your phone, tablet and/or laptop. Next, be sure to [install](https://tailscale.com/kb/1273/qnap/) Tailscale on your QNAP NAS. You can choose your NAS as your [exit node](https://tailscale.com/kb/1103/exit-nodes/). This is nice if you are abroad and want to access websites/movie services only available in your home country. Open the app on your QNAP and select advertise as exit node to enable this feature.

But there is more. What if you want to access devices only available on your network at home. For this, you have to enable the local bridge mode. This will require you to access your QNAP system via SSH. You will have to ensure that [SSH is enabled](https://www.qnap.com/en/how-to/faq/article/how-do-i-access-my-qnap-nas-using-ssh) on your QNAP NAS. Next, an SSH client is required. If you are on macOS, use Terminal. On Windows, you can use the Windows Terminal to connect to your NAS:

```bash
ssh your-nas-user@your-nas-ip-or-dns
```

After successfully logging in, depending on your QNAP NAS, you must locate where your Tailscale app has been installed. You should find it in either one of these two locations:

1. */share/CACHEDEV1_DATA/.qpkg/*
2. */share/MD0_DATA/.qpkg/*

You can check if the NAS exists using the `ls THE-PATH` command. If a list of directories is returned, you have found the right one. Next, invoke the following command - in my case, it was the second option, adjust to your NAS accordingly:

```bash
/share/MD0_DATA/.qpkg/tailscale/tailscale -socket /tmp/tailscale/tailscaled.sock up --advertise-routes=\192.168.1.0/24 --advertise-exit-node --reset
```

Note that you might have to change the IP according to the IP of your local network. The /24 is for the Subnet and says how many Bits are used to identify the network, e.g. shorthand for `255.255.255.0`. Once this command runs through, open up the Tailscale app. You will be able to enable your NAS as a logical bridge. Once enabled and connected to your Tailscale VPN, you can see all your local network devices.

HTH
