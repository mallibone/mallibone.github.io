---
layout: single
title: "Extracting data from websites with F#"
title: Extracting data from websites with F#
date: 2019-10-08
tags: ["F#"]
slug: "ral-colour-table-with-fsharp"
---

[![A picture containing colour crayons]({{ site.url }}{{ site.baseurl }}/images/c82746ea-9147-409a-a4f1-01a312c06f36.jpg "A picture containing colour crayons")]({{ site.url }}{{ site.baseurl }}/images/8258d68a-8d7d-491d-bea0-984f02b1c0b8.jpg)

I recently was faced with the task to render [RAL Colours](https://en.wikipedia.org/wiki/RAL_colour_standard) on an app that I was developing. RAL Colours are used mainly in industrial colour appliances - i.e. powder coating. So while well known in the powder coating industry it was quite an exciting read on Wikipedia to find out how RAL colours came to life. The good thing is that there is a finite set of classic RAL colours. Even better there is a table on [Wikipedia](https://en.wikipedia.org/wiki/List_of_RAL_colors) which contains a good enough approximation for the classic RAL colours.

So instead of copy & pasting the table into an editor and slashing away at the data. I was wondering if there would be a better way to extract the information from the website. And store the data in a more handy form such as a JSON file.

In the last couple of months, I have been dabbling with F# in my free time. Data providers are a powerful tool which is available in the F# language. In short, you can point a data provider at a data source, and during compilation, the types used in that source get generated for you. So for your typical JSON response from a website. You can use the JSON type provider to create a type based on that stream which you then can use throughout your F# program. Now, this is a more than your "create a C# POCO from JSON" Visual Studio feature. You also get methods to slice and dice through your data. In other words, it is an excellent tool for exploring new data sources or just parsing new data sources and processing that data.

As with LINQ extensions it is is possible to write your type providers. But for most general use cases the type provider already exists and can be added to your project as a NuGet package. The [NuGet](https://www.nuget.org/packages/FSharp.Data/) package we will be using is the `FSharp.Data` which provides type providers for JSON, XML, CSV and HTML (plus the World Bank ‍🤷‍♂️).

Using an F# script, we will first have to reference the type provider:


    #I "./packages"
    #r "FSharp.Data/lib/netstandard2.0/FSharp.Data.dll"
    
    open FSharp.Data





> Side note: I am using [paket](https://fsprojects.github.io/Paket/index.html) for my dependency management because it installs the package right into the project folder. You do not need to use paket, but you will have to make sure that the `#r ...` line points to the dll.


Now type providers create a type based on a data source. In our case I can point it at the Wikipedia website listing all the RAL colours:


    type wikipedia = HtmlProvider<"https://en.wikipedia.org/wiki/List_of_RAL_colors">


We could have also provided a local file:


    type wikipedia = HtmlProvider<"list_of_ral_colors.html">


The local file is excellent if you do not always want to hit the remote site. But you run the risk of having an older version locally than on the server which can lead to ugly problems. As far as we are concerned for the script. I will point it at Wikipedia and be sure to make my again annual donation at the end of year

In the JSON file, we will want to store the RAL, RGB and colour name. So let's create a record type for that quickly:


    type ralColor = { ral: string; hex: string; name: string}


Now that we have our types, we are all set to extract that data. By looking at the website, we can see in which section the table is located:

[![Picture showing part of the Wikipedia RalColor List]({{ site.url }}{{ site.baseurl }}/images/3de4d662-5203-4b5c-b3ea-19c34e65d7d4.png "Picture showing part of the Wikipedia RalColor List")]({{ site.url }}{{ site.baseurl }}/images/1d87ff95-20d6-4a8e-8b10-6fd79d8c3fce.png)

Knowing this location, we can scan the site and hone in on the data we are looking for:


    let ralColorSection = wikipedia.Load("https://en.wikipedia.org/wiki/List_of_RAL_colors")
                                    .Tables
                                    .``All RAL Colours in a single listing``


If you have never used type providers, you will be probably reading the lines above and go: "Okay... I guess..." - so let's just quickly look at what happened underneath those lines of code. As the name suggests, type providers provide a type based on a data source. This is where we nod. So what do we get with the lines above? We get a type which represents the table of RAL colours. We can access all of the rows via `ralColorSection.Rows`. When iterating over each row we can read the value in a column by using its name. So we could print out all colour names as follows:


    ralColorSection.Rows |> Seq.iter (fun r -> printfn "%s" r.``Colour name``)


I know this is freaking cool right! So if we wanted to extract the RAL, RGB and name from the table we could use our previously defined type and the values as follows:


    let ralColors = ralColorSection.Rows
                        |> Seq.map((fun r -> 
                            {ral = r.``RAL Number``; 
                                hex = r.``HEX Triplet``; 
                                name = r.``Colour name``}))


Note the two ticks are how F# variables with spaces in them can be accessed. Now we have all the data we wanted. So now all that is left to do is the boring bit of storing it into a JSON file:


    #I "./packages"
    #r "Newtonsoft.Json/lib/netstandard2.0/Newtonsoft.Json.dll"
    
    // ...
    
    open Newtonsoft.Json
    
    // ...
    
    let writeToJsonFile ralColors =
        let filePath = Path.Combine(__SOURCE_DIRECTORY__, "ral_colour_map.json")
        let jsonString = JsonConvert.SerializeObject(ralColors)
        File.WriteAllText(filePath, jsonString, Encoding.UTF8) |> ignore


And that wraps up this blog post. I hope you have seen that F# 's type providers can be a great way to scan through data sources and extract the information you need. One thing to be aware of when using type providers: You can't directly share the generated type with other .Net languages such as C#. You would have to wrap the data in a record type - by the way: the type we created to hold the subset of data is a record type. So while there might be some additional effort up ahead when writing fully-fledged enterprise applications, they are a no brainer for scripting. And will provide you with a significant productivity boost when exploring new datasets.

Be sure to check out the official [documentation](https://fsharp.github.io/FSharp.Data/library/HtmlProvider.html) on the HTML provider used in this post. As always you can find the entire code on [GitHub](https://github.com/mallibone/RalColorTable).

HTH
