---
layout: single
title: "PowerShell for the C# developer–Part 2"
title: PowerShell for the C# developer–Part 2
date: 2017-02-08
tags: ["PowerShell", "General"]
slug: "powershell-for-the-c-developer–part-2"
---

![](https://upload.wikimedia.org/wikipedia/commons/2/2f/PowerShell_5.0_icon.png)

In the first part of the series we covered the [development environment setup](https://mallibone.com/post/powershell-for-the-c-developer) on how to get started with PowerShell. Now lets dive into some code. PowerShell can be used as a dynamic language. For a C# developer this can be one of the most frustrating points. In this post we will look at the following points:

- Variables
- If/else
- Loops and Piping
- Methods
- File handling


So let’s get going ![Smile](https://mallibone.com/posts/files/8bb3ab55-a4fe-4350-a71f-1a4fa0879d3f.png)

# Variables

When we look at a simple program of C# it might look something like this.
<script src="https://gist.github.com/mallibone/1ad910cda776aac441ea20c5ff163afb.js"></script><link rel="stylesheet" href="https://assets-cdn.github.com/assets/gist-embed-5743a64247069ce0c6698fa0dea4de1ba28bdba6ec23513a5aa8ce38828a8124.css">







|  | using System; |
| --- | --- |
|  |  |
|  | namespace ConsoleApplication |
|  | { |
|  | public class Program |
|  |     { |
|  | public static void Main(string[] args) |
|  |         { |
|  | string name = "Harvey Specter"; |
|  | int number = 42; |
|  |             Console.WriteLine($"Hello {name}, your number is {number}"); |
|  |         } |
|  |     } |
|  | } |






[view raw](https://gist.github.com/mallibone/1ad910cda776aac441ea20c5ff163afb/raw/17d16da9fb2fd93a26896985620cbb043cf16147/Program.cs)[Program.cs](https://gist.github.com/mallibone/1ad910cda776aac441ea20c5ff163afb#file-program-cs)<br>        hosted with ❤ by [GitHub](https://github.com)



Now in comparison here is the equivalent PowerShell code.
<script src="https://gist.github.com/mallibone/6e4ea5972010416d84706d0ab4115071.js"></script>
Note that we do not need any Class or Method to get started. Simply start writing your script. Now variables are interesting under PowerShell. Lets add some more info to our PowerShell code to retrieve the type of the variables. The variable is assigned a type when a value is assigned. Since PowerShell is not compiled there are some potential pot holes a typical C# developer might into. For starters this is a totally valid statement.
<script src="https://gist.github.com/mallibone/212b5f20a11d89e0044ea28c5a9e9e72.js"></script>
Resulting in the following output:

[![Variable is first of Type Int32 after assignment of a string has the type String.](https://mallibone.com/posts/files/c236ed19-a1c8-499d-ae27-0a7c43e593f6.png "Variable is first of Type Int32 after assignment of a string has the type String.")](https://mallibone.com/posts/files/c639864b-59d0-49e1-9c24-9ed9826f8b50.png)

We can be more strict in PowerShell by defining the type of the variable which will make the second assignment illegal. But this requires some additional effort on your end.
<script src="https://gist.github.com/mallibone/50cffc143d6ced8680f362aa1d3bd36a.js"></script>
If we would run the strict assignment we would be greeted by an error message which is more of what a C# developer would be used to.

And one more thing. Even though the variable <font face="Consolas">$neverDefined</font> never got defined. Well we can still access it’s value without an exception or error being raised.
<script src="https://gist.github.com/mallibone/676f9af42f8850f61137c35bca618d34.js"></script>
[![PowerShell output showing that the variable $neverDefined simply shows an empty string](https://mallibone.com/posts/files/f487fdf0-67ed-419c-bb82-9fe953e7d86d.png "PowerShell output showing that the variable $neverDefined simply shows an empty string")](https://mallibone.com/posts/files/c949c9eb-241c-4d2d-8686-16429fd58c1a.png)

Keep this in mind while developing since they might just come around and bight you in the foot later on.

# Conditional Operators

When writing conditional code in C#, the standard choice is using if and else or for multiple options a switch/case. So a possible option would be to use them as follows:
<script src="https://gist.github.com/mallibone/6de3744a2b0811dc882f933d21238e59.js"></script>
Apologizing to all the readers who have to work shifts ![Winking smile](https://mallibone.com/posts/files/bbe945ac-d417-406c-ae30-b4aa13440726.png) Lets look at how the same code would be implement in PowerShell:
<script src="https://gist.github.com/mallibone/ff1423c0ff626237cb879b05c4fb6086.js"></script>
No huge changes or surprises So no surprises here. The major difference is the equality sign in the if check. Here is a small translation table of the equality signs you find in C# and PowerShell:


| **Purpose** | **C#** | **PowerShell** |
| --- | --- | --- |
| Equal | == | -eq |
| Not Equal | != | -ne |
| Greater Then | &gt; | -gt |
| Less Then | &lt; | -lt |
| Greater or Equal | &gt;= | -ge |
| Less or Eual | &lt;= | -le |


# Loops and Piping

There are many different constructs for looping in C#: for, while, do while and <font face="Consolas">ForEach</font>. So if we look at all the different types of loops in C#:
<script src="https://gist.github.com/mallibone/6d5dfe1f888aa08e75dd937db5ce4ce0.js"></script>
In PowerShell the equivalent can be  implemented like so:
<script src="https://gist.github.com/mallibone/463ed7c506c81825f49321414454d505.js"></script>
Now the <font face="Consolas">ForEach</font> loop is really great when having to iterate over a list of items. This is often what we end up doing e.g. “Iterating over a list of people to get the count by city” or a bit more PowerShelly “Iterate over number of host information to ensure that everything is okay and no action is needed”. While we could write that with the above for loop, there is a PowerShell called piping. Piping allows us to Take a collection and forward it to the next operation. We could rewrite the <font face="Consolas">ForEach</font> sample as follows:
<script src="https://gist.github.com/mallibone/909cb3762bdeac415fe576168d6048da.js"></script>
Pretty cool no? ![Smile](https://mallibone.com/posts/files/8bb3ab55-a4fe-4350-a71f-1a4fa0879d3f.png) We can even use the <font face="Consolas">ForEach</font> construct to loop over every item being forwarded:
<script src="https://gist.github.com/mallibone/17587abda84f01c0cc4927612244563c.js"></script>
Note that $\_ is always the current item we are going over in the <font face="Consolas">ForEach</font> Loop. The equal construct in C# is achieved with Extension Methods. Piping can be great to enhance readability since one can see the flow. But it also can make the code harder to debug, so be sure to keep the balance here.


> **Note:** Even though <font face="Consolas">foreach</font> is an alias for <font face="Consolas">ForEach-Object</font> they behave differently. When using <font face="Consolas">foreach</font> the operation is paused until all elements are to be processed are present, in case of <font face="Consolas">ForEach-Object</font> it will process the items as they come in. This may lead to some unexpected side effects…


# Summary

In this Blogpost we saw the basic programming structures in PowerShell compared to how they would be implemented in C#. Keep in mind that PowerShell is more dynamic and forgiving at runtime than C# which might lead to some unwanted side effects. In the next Post we will look at how we can implement Methods and work with Parameters which are not only handy for methods but also for Command Line Interface parameters.

# References

There is more in this blog post series:

  


- [<u><font color="#0066cc">PowerShell setup</font></u>](https://mallibone.com/post/powershell-for-the-c-developer)
- [<u><font color="#0066cc">PowerShell basics</font></u>](https://mallibone.com/post/powershell-for-the-c-developer–part-2)

