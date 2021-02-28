---
layout: single
title: "PowerShell for the C# developer – Part 3"
title: PowerShell for the C# developer – Part 3
date: 2017-02-22
tags: [" PowerShell", "General"]
slug: "powershell-for-the-c-developer-–-part-3"
---

# ![PowerShell Icon](https://upload.wikimedia.org/wikipedia/commons/2/2f/PowerShell_5.0_icon.png "PowerShell Icon")
 
In this Post we will be looking at how to use methods in PowerShell. When thinking about methods parameters are bound to come up. In C# parameters do not get much glory. But in PowerShell parameters do come with some extra features. Which can be reused when invoking a script from the command line.

In the previous posts we saw how to get setup and in Part 2 how to the basic command structures can be implemented. In this post I’ll assume you are familiar with the topics of those posts. So if you are new to PowerShell you might want to read the posts to get familiar with some basic concepts.

# Methods and Parameters
 
Though PowerShell does not require any methods from the get go. It does provide the structure to make code reusable by putting it into a function.
<script src="https://gist.github.com/mallibone/aa3e51fc608123aba1cfaa24ee8bdab3.js"></script> 
The C# function above could be translated to a PowerShell as follows.
<script src="https://gist.github.com/mallibone/593b2bdbbcc72c75a60cb1b826fe4c87.js"></script> 
Most functions take parameters, so let’s do that, again leading of with the C# function:
<script src="https://gist.github.com/mallibone/20940ff7d5f4bdcc78e5264ca670f546.js"></script> 
And the corresponding PowerShell code:
<script src="https://gist.github.com/mallibone/f7ff318025629e96d1fdd56e8926f94f.js"></script> 
As you can see the PowerShell function uses a special keyword.  The parameter name starts with a dollar sign. Type Information is also added to the parameter. It is optional to define the type and if not wanted can be simply left out. Invoking the function does not require any braces for the parameter and they will default to the order they are defined in the method. Now let's look at the following code:
<script src="https://gist.github.com/mallibone/d05ed9a851113e8a3874b4ec64eead3d.js"></script> 
Parameters in PowerShell provide some interesting ways. The previous samples shows how we can write a method with multiple parameters. And also how we can set a parameter by it's name. But we are only scratching the surface of what is possible. The common scenario to start a PowerShell script is from  the Command Line Interface (CLI). To keep a script flexible it makes sense to start it with parameters e.g. IP or URI of a server. When defining the same construct as in the method we can have a script requiring parameters.
<script src="https://gist.github.com/mallibone/b039ac701ac7cafee6be7ac3e9c19d97.js"></script> 
The script can then be invoked further as follows:
<script src="https://gist.github.com/mallibone/4b4f3cc98da9c07ae2774c1659d6335b.js"></script> 
Note that the name of the parameter is reused for the command line parameter. But we can do even more, lets consider the following method description:
<script src="https://gist.github.com/mallibone/819116817213684531a17fbdcaf8a9dc.js"></script> 
We now have three parameters which are assigned default values similar to the way they are done in C#. We can further set the order of the parameters, make a parameter mandatory and even checking a parameter as follows.
 
Summing it you can see that PowerShell provides a  way to  work with methods and provides a rich way on interacting with  parameters. The parameters can also be used when writing a script. Simply place the parameter code at the top of the script and the parameters can be set when invoking the script.
 
# File handling
 
No introduction on PowerShell would not be complete if we wouldn’t have a small section regarding file handling. Writing to a File is as easy as writing the following line:
<script src="https://gist.github.com/mallibone/d1df679449f701b0d1168d46a64b4b7f.js"></script> 
This way we can easily persist information for later usage or inspection of a state during runtime. If the goal is to persist the state of a PowerShell object model. Serialization will serve as a better alternative. Serializing an object provides the option of restoring it’s state at a later time. Allowing to inspect the object in the REPL of PowerShell and analyze why a certain condition may have lead to a failure. Serializing an object in PowerShell can be realized with the following command:
<script src="https://gist.github.com/mallibone/99076c00e79387d15464583c07034fb0.js"></script> 
Deserializing an object will be performed with the following line:
<script src="https://gist.github.com/mallibone/9bf3f127a9a0fe79b028b553c536ae22.js"></script> 
Storing state by serializing objects into an XML file can provide as an elegant solution for transferring state between machines or rolling back to a previous state if an error occurs during the execution of a script. One could also use the serialized objects for automated tests without having to touch the actual system. Let this be my next blog post relating to PowerShell.
 
# Summary
 
In this post we saw how basic control structures can be created with PowerShell such as:
 
- Methods and Parameters
- File handling
- Serializing and deserializing objects

 
With these basic control structures you are able to create simple scripts and reuse code by defining methods. As PowerShell scripts start to grow, testing them gets more and more difficult. This is why in the next post we will be looking into automated testing with PowerShell.
