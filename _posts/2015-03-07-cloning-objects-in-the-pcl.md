---
layout: single
title: "C# cloning objects in a PCL"
title: C# cloning objects in a PCL
date: 2015-03-07
tags: ["Xamarin", "Mobile", "Windows", "Windows Phone"]
slug: "cloning-objects-in-the-pcl"
---
[![cloning_by_silverstormblackfang-d5bkptt]({{ site.url }}{{ site.baseurl }}/images/aca302d7-f2e1-4b0c-81f5-0cb964c604d2.jpg "cloning_by_silverstormblackfang-d5bkptt")]({{ site.url }}{{ site.baseurl }}/images/6f1baa16-7684-4798-b3e8-760f3d5c4438.jpg)   

Usually when cloning an object in .Net I just use the Binary stream, this will automatically perform a deep copy of the object. But when writing code on the PCL we do not have the Binarystram and therefore have to look for new options. I'm not a big fan of doing this manually by implementing copy constructors on each an every object as it is an error prone task as it leaves it up to the devs to implement it on new or changes to existing classes.

So the next best option to hand is using another stream writer, I started to use [JSON.Net](http://www.newtonsoft.com/json) for the task as it is available for all the major mobile platforms and on the server objects. So we can rest assured for now that it will run on all the platforms we target. Another goody I like to include implementing it as an extension method on object. So lets have a look at how this would be done:




    public static T Clone<T>(this T source){    if (Object.ReferenceEquals(source, null))    {        return default(T);    }    // In the PCL we do not have the BinaryFormatter    return JsonConvert.DeserializeObject<T>(JsonConvert.SerializeObject(source));}




So from now on when ever we need to clone/deep copy an object we can simply invoke it by calling the clone method on the required object.




    var original = new NestedObject();var clone = original.Clone();






HTH
