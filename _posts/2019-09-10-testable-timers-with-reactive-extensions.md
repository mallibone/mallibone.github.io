---
layout: single
title: "Testable timers with Reactive Extensions for .Net"
title: Testable timers with Reactive Extensions for .Net
date: 2019-09-10
tags: ["testing", "xunit"]
slug: "testable-timers-with-reactive-extensions"
---

[![A view of Big Ben at night with traffic lights going by.]({{ site.url }}{{ site.baseurl }}/assets/images/c994a4d1-6325-454e-b0e3-dfc61438392e.jpg "A view of Big Ben at night with traffic lights going by.")]({{ site.url }}{{ site.baseurl }}/assets/images/e312fddb-8fa2-4f2b-bc85-a3f1519e19df.jpg)

[Reactive Extensions](http://reactivex.io/) ([Rx.Net](https://github.com/dotnet/reactive)) was released in 2009 and therefore is celebrating it's 10th anniversary this year. Reactive Extensions is well known as a solution approach for event-based systems. However, it also features many other helpful features, such as timers! I guess the title of the blog might have given this one away - nonetheless let me show you why Rx.Net provides you with a great alternative to the standard .Net timer(s).

Usually, timers serve for two scenarios. Delaying the execution of a certain piece of logic until a specified later time or to periodically execute some logic. For example, in network stacks, it is common to send a heartbeat / keep-alive message every n-seconds to let the counterpart know that the communication partner is still alive, and the connection should be kept open. We could implement such a general timer as such:


    public class LiveBit : IDisposable
    {
        private readonly INetworkInterface _networkInterface;
        private readonly Timer _timer;
    
        public LiveBit(INetworkInterface networkInterface)
        {
            _networkInterface = networkInterface;
            _timer = new Timer(1000);
            _timer.Elapsed += SendLiveBit;
            _timer.AutoReset = true;
            _timer.Enabled = true;
        }
    
        public void Dispose()
        {
            _timer.Dispose();
        }
    
        private void SendLiveBit(object sender, ElapsedEventArgs e)
        {
            Console.WriteLine("Send Live Bit");
            var liveBit = new byte[]{0xAA}; // Some imaginative payload
            _networkInterface.Send(liveBit);
        }
    }


The code is implemented using only the .Net library and does not use Rx.Net. So why should we even bother to replace this piece of code? Well, how would one go and test this code? We could write a Test looking something like this:


    [Fact]
    public async Task LiveBit_WhenCreated_TheSendMethodWillBeInvokedAfter1Second()
    {
        // Arrange: Given a mock interface ...
        int counter = 0;
        // Using Moq https://www.nuget.org/packages/Moq/
        var networkInterfaceMock = new Mock<INetworkInterface>();
        // Write a function that counts the number of invocations
        networkInterfaceMock
            .Setup(n => n.Send(It.IsAny<IEnumerable<Byte>>()))
            .Callback(() => counter++);
        // Act: ... start the livebit service and wait for a good second ...
        var liveBit = new LiveBit(networkInterfaceMock.Object);
        await Task.Delay(TimeSpan.FromMilliseconds(1100));
        // Assert: ... the mocked send method has been invoked once.
        Assert.Equal(1, counter);
    }


To be honest whenever I see an `await Task.Delay(someNumber)` it never feels right. Usually, this is always some hack. I am not saying you should never do this, but in 95% of all cases, they are an indication of bad programming. However, what else can we do? "Shift time by one second!" - Sure, but how would you do that? Well, let me show you. First, let's rewrite the timer function to use the Rx.Net approach:


    public class ReactiveLiveBit : IDisposable
    {
        private readonly INetworkInterface _networkInterface;
        private readonly IDisposable _timer;
    
        public ReactiveLiveBit(INetworkInterface networkInterface)
        {
            _networkInterface = networkInterface;
            _timer = Observable
                .Interval(TimeSpan.FromSeconds(1))
                .Subscribe(x => SendLiveBit());
        }
    
        public void Dispose()
        {
            _timer.Dispose();
        }
    
        private void SendLiveBit()
        {
            Console.WriteLine("Send Live Bit");
            var liveBit = new byte[]{0xAA}; // Some imaginative payload
            _networkInterface.Send(liveBit);
        }
    }


We can rerun our test, and it is still passing. However, the delay is still present in our test code. Looking at the definition of Rx.Net `Scheduler`, we see that it takes an `IScheduler` as an argument. This means that we can inject the time (ticker) into our scheduler - in other words, we can play [timelords](https://en.wikipedia.org/wiki/Time_lord). So if we rewrite our constructor a bit:


    public ReactiveLiveBit(INetworkInterface networkInterface, IScheduler scheduler = null)
    {
        var timerScheduler = scheduler ?? Scheduler.Default;
        _networkInterface = networkInterface;
        _timer = Observable
            .Interval(TimeSpan.FromSeconds(1), timerScheduler)
            .Subscribe(x => SendLiveBit());
    }


We now pass in an optional parameter of `IScheduler` which is set to `null` by default. Moreover, if the parameter is not set, we use the `Scheduler.Default`. So if we do not inject a scheduler, the method uses the system clock and a second is a second. To test this, we can rerun our test at this point to find it still snoozing in the green.


> When testing with Rx.Net, there is a dedicated [NuGet package](https://www.nuget.org/packages/Microsoft.Reactive.Testing) `Microsoft.Reactive.Testing` that provides some excellent helpers such as the `TestScheduler`.


Using the `TestScheduler` we can rewrite our test code like this:


    [Fact]
    public void ReactiveLiveBit_WhenCreatedWithATestScheduler_TheSendMethodWillBeInvoked1TestSecond()
    {
        // Arrange: Given a mock interface ...
        int counter = 0;
        // Using Moq https://www.nuget.org/packages/Moq/
        var networkInterfaceMock = new Mock<INetworkInterface>();
        // Write a function that counts the number of invocations
        networkInterfaceMock
            .Setup(n => n.Send(It.IsAny<IEnumerable<Byte>>()))
            .Callback(() => counter++);
        var testScheduler = new TestScheduler();
        // Act: ... start the livebit service and wait for a good second ...
        var liveBit = new ReactiveLiveBit(networkInterfaceMock.Object, testScheduler);
        testScheduler.AdvanceBy(TimeSpan.FromSeconds(1).Ticks);
        // Assert: ... the mocked send method has been invoked once.
        Assert.Equal(1, counter);
    }


Utilising the `TestScheduler` allows us to move forward in time with the `AdvanceBy` method. Forwarding the time will execute the timer code and reduce our test execution time to mere milliseconds.

## Conclusion

So do you need Rx.Net to write timers? No, you do not - then again do you need C# to write applications instead of using assembler? Again no, you don't, but it helps to ship stable and maintainable code. Be sure to check out the official Rx.Net website to find more resources on Rx.Net.

You can find all the code to this small sample on [GitHub](https://github.com/mallibone/TestingReactiveScheduler).

HTH
