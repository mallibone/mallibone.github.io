---
layout: single
title: "Developing your .NET app using virtual serial ports on macOS"
date: 2025-04-04 00:03:00
tags: [".NET", "macOS", "socat", "dev life", "IoT"]
slug: "dotnet-on-macos"
---

It's the year 2025, and I'm writing this blog post about serial ports—also known as RS232. While this might feel like a blast from the past for many, if you're working in IoT or IIoT, you might find yourself writing a .NET app that communicates over a serial port.

![A low-poly digital illustration of a programmer at a desk working on an IoT application, with a laptop displaying a .NET interface. The scene includes stylized geometric elements like circuit boards, microcontrollers, and serial port connectors, set in a cool-toned color palette of blues, greens, and purples, evoking a modern and technical atmosphere.]({{ site.url }}{{ site.baseurl }}/assets/images/2025-04-04-dotnet_serial.png "A low-poly digital illustration of a programmer at a desk working on an IoT application, with a laptop displaying a .NET interface. The scene includes stylized geometric elements like circuit boards, microcontrollers, and serial port connectors, set in a cool-toned color palette of blues, greens, and purples, evoking a modern and technical atmosphere.")

Using virtual serial ports allows you to test your app without needing any hardware connected to your development machine. So let’s go over the steps you need to take and a simple setup to verify that everything works as expected.

<!-- expand -->

Since we're on a Mac (I'm using a MacBook Pro with an M2 Max), the first step is to install `socat` via the command line. Open your terminal and enter the following command:

```bash
brew install socat
```

This installs `socat`—a very handy tool for creating virtual serial (RS232) ports. In fact, `socat` does much more: it can relay all kinds of communication protocols such as TCP, UDP, and Unix sockets. So if you're looking for a powerful routing tool on a Unix-like system, this might be your [tool of choice](https://linux.die.net/man/1/socat).

But back to our serial ports. With the following command, we’ll set up two virtual serial ports, where the TX of the first port is "wired" to the RX of the second one - and vice versa:

```bash
socat -d -d pty,raw,echo=0 pty,raw,echo=0
```

After running this command, you should see something along the lines of:

```bash
2025/04/03 14:08:15 socat[68927] N PTY is /dev/ttys000
2025/04/03 14:08:15 socat[68927] N PTY is /dev/ttys001
2025/04/03 14:08:15 socat[68927] N starting data transfer loop with FDs [5,5] and [7,7]
```

On my machine, the serial sockets were created under `/dev/ttys000` and `/dev/ttys001`—but this may differ on your system, so make sure to check your terminal output rather than relying on the example above.

With the sockets and wiring in place, let’s write a basic .NET app to verify that everything works. .NET doesn’t include built-in support for RS232, so you’ll need to install the `System.IO.Ports` [NuGet package](https://www.nuget.org/packages/System.IO.Ports#readme-body-tab).

```bash
dotnet add package System.IO.Ports
```

Now, let’s take a look at a simple echo console app:

```c#
using System.IO.Ports;

var portName = "/dev/ttys001"; // change as needed
using var serialPort = new SerialPort(portName, 9600);

serialPort.Open();
Console.WriteLine($"Echo server on {portName}");

serialPort.DataReceived += (s, e) =>
{
    var sp = (SerialPort)s;
    string incoming = sp.ReadExisting();
    Console.WriteLine("Received: " + incoming);
    sp.Write(incoming); // Echo back
    Console.WriteLine("Echo");
};

Console.ReadLine();
```



Note the `portName` and the baud rate (set to 9600)—these might be different for your setup. Other than that, this service will simply echo back whatever you send to it. If you like, you could extend it to simulate responses for specific commands, giving you a basic simulation environment for a device. But I’ll leave that up to you, in case you’re ready to go down that rabbit hole. 😉

We can now start the echo service, and it should simply connect and wait to receive data. For demonstration purposes, let’s write a small app that lets us enter messages to be sent to `ttys000` - and we should receive them back immediately.

```c#
using System.IO.Ports;
using System.Text;

Console.WriteLine("RS232 Terminal Application");

// Setup variables
var serialPort = new SerialPort("/dev/ttys000", 9600);
var running = true;

try
{
    // Open port and handle exit
    serialPort.Open();
    // ..
    
    // Start read task
    _ = Task.Run(() => {
        var buffer = new byte[1024];
        while (running)
        {
            // ...
                    var bytesRead = serialPort.Read(buffer, 0, Math.Min(buffer.Length, serialPort.BytesToRead));
                    Console.Write($"Received: {Encoding.ASCII.GetString(buffer, 0, bytesRead)}");
           // ...
        }
    });
    
    // Main send loop
    Console.WriteLine("Type messages (Ctrl+C to exit):");
    while (running)
    {
        var message = Console.ReadLine();
        if (!string.IsNullOrEmpty(message) && running)
        {
            try { serialPort.WriteLine(message); } 
            catch (Exception ex) { Console.WriteLine($"Send error: {ex.Message}"); }
        }
    }
}
// ...

```



With this app, you can now send messages and see them echoed back—just like that, you’ve got an easy way to develop and test COM port functionality without needing to attach any physical devices.

To clean up the virtual ports created by `socat`, simply stop the process using *Ctrl+C*. The virtual ports will then be removed automatically.

You can find the complete sample application from this blog post on [GitHub](https://github.com/mallibone/DotNetSerialEcho).

HTH


