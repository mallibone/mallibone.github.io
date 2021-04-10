---
layout: single
title: "Migrating mallibone.com from MiniBlog to Jekyll"
date: 2021-04-13 15:03
tags: ["Jekyll", "MiniBlog", "F#"]
slug: "miniblog-to-jekyll"
---
[![Photograph showing boxes]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-14-jekyll-title.jpeg "00_TitlePhoto")]({{ site.url }}{{ site.baseurl }}/assets/images/2021-04-14-jekyll-title.jpeg)

After 8 years of using [MiniBlog](https://github.com/madskristensen/MiniBlog) created by [Mads Kristensen](https://twitter.com/mkristensen), I decided it was time to move to a markdown / statically generated website based on [Jekyll](https://jekyllrb.com/). And since it seems obligatory to tell ones journey when migrating from one blogging platform to another, I will follow the calling. But let\'s start with why? Migrating a blog platform is no small feat, and I have been using my current system since 2013. Well, it was not because I was looking to stay on the latest and greatest. 😉 But there were a few things that finally made up my decision to pack my current XML and image files and move them to the markdown land of statically generated websites.

MiniBlog is really great. I was able to run it on a minimal Azure plan. It required no database and even comes with an online editor that allows you to write blog posts within the browser. But I rarely used the online editor. Mostly I would use the [Open Live Writer](https://openlivewriter.com/) (OLW), which provided a What-You-See-Is-What-You-Get (WYSIWYG) experience similar to Word. While OLW is an excellent tool, it is (at the time of writing) a Windows-only solution, and I am writing these lines on a Mac. So I ended up writing my blog posts in markdown, then exporting the HTML and importing/uploading it using OLW. Not great, but doable. What finally made me jump the ship was the fact that the current implementation still ran on .Net 4.x. Now there is a MiniBlog for .Net Core. But I was already intrigued by statically generated websites since [Gerald](https://blog.verslu.is/), [Steven](https://www.thewissen.io/) and I started our own podcast - [Null Pointers](https://nullpointers.io/) \#BeSureToCheckItOut. 🙂 We choose [GitHub](https://pages.github.com/) Pages. For one, they are free (always a great price to begin with), provide the build toolchain and were just an overall great fit for our needs. GitHub Pages uses [Jekyll](https://jekyllrb.com/) under the hood, which is based on Ruby and has been around for quite some time. There is a wide variety of themes to choose from, and some are even free to use. After getting familiar with the Jekyll system for setting up our podcast site, I was sold on it and started to plan to migrate my blog over to it.

## The migration

Before starting out on my journey, I laid out a carefully (not really) crafted (again\... sort of) \"battleplan\":

-   Get hold of blog posts

-   Migrate from XML to Markdown

-   Migrate images

-   Find a theme

-   Ensure old links still work

-   Fiddle some more with Jekyll\*

> \*) While not on the original plan, this ended up taking quite a bit of time on my end. But it was fun, and I even learnt a bit of CSS 😃

### Getting hold of my blog posts

I had two things going here for me. For one, MiniBlog actually does not require a database. It stores all of the blogposts in XML. The second thing I had going for me is my paranoia - which meant I had enabled automated [backups](https://docs.microsoft.com/en-us/azure/app-service/manage-backup) of my site. So all that I had to do is grab the latest zip-backup of my blog, and I was ready to start migrating it over to markdown.

### From XML to Markdown

Usually, when migrating from one blog platform to another, I recommend using [your favourite search engine](https://duckduckgo.com/). There is an excellent chance when using a popular platform. Someone has already gone down the same path and has written about his experiences. So you might find a blog post or even a complete migration script that you can try on a copy of your blog. But MiniBlog seems to have stayed a secret gem.

Since MiniBlog uses XML files to store blog posts, I thought why not use the F\# [Type Providers](https://docs.microsoft.com/en-us/dotnet/fsharp/tutorials/type-providers/) I have already used for similar tasks. Using the Typeprovider and a reference XML file:

```ocaml
#r  "nuget: FSharp.Data"

open FSharp.Data

// ...

type BlogPost = XmlProvider<"path-to-miniblog-post.xml">
```

I was able to extract the meta information of the blog post like this:

```ocaml
let createHeader (blog:BlogPost.Post) =
    [ "---";
        "layout: single";
        sprintf "title: \"%s\"" blog.Title;
        sprintf "title: %s" (blog.Title.Replace(":", "&#58;"));
        sprintf "date: %s" (blog.PubDate.ToString("yyyy-MM-dd"));
        sprintf "tags: [%s]" (formatTags blog.Categories);
        sprintf "excerpt: '%s'" blog.Excerpt;
        sprintf "slug: \"%s\"" blog.Slug;
        // sprintf "excerpt: \"%s\"" blog.Excerpt;
        "---" ]
    |> String.concat "\n"

let parseBlog outputDirectory blog =
    let header = createHeader blog

    // convert content
    
    // adjust image paths
    
    let post = header + "\n" + content

		// filename
    let postname = $"""{blog.PubDate.ToString("yyyy-MM-dd")}-{blog.Slug.Replace("/", "-").Replace(":", "")}.md"""

    // ensure output directory is created
    Directory.CreateDirectory(outputDirectory) |> ignore

		// Write file
    let file = Path.Combine(outputDirectory, postname)
    File.WriteAllText(file, post)
    file
```

Next would be the content which was stored as HTML in another tag. Quickly playing with the idea of writing my own HTML to Markdown parser, I just as quickly abandoned that idea finding the [ReverseMarkdown NuGet package](https://www.nuget.org/packages/ReverseMarkdown/). With that, I could convert my HTML to markdown in one line:

```ocaml
let rawContent = (Converter()).Convert(blog.Content)
```

I was already quite happy with this. Next would be the images.

### The Images

The images were already optimised by OLW, so all that I had to do is change the URL and copy them into the correct directory. So I extended my F\# script with a few more lines of code for copying the images:

```ocaml
let copyImage destinationDirectory source =
    Directory.CreateDirectory(path=destinationDirectory) |> ignore
    let filename = FileInfo(source).Name.Replace("NoLongerNeededFilenamePart", "")
    let destination = Path.Combine(destinationDirectory, filename)
    File.Copy(source, destination, overwrite=true)
    destination
```

And then used a string replace while generating the markdown:

```ocaml
let content = 
    rawContent.Replace("https://mallibone.com/posts/files/", "{{ site.url }}{{ site.baseurl }}/assets/images/")
```

### Finding the right theme

Jekyll comes out of the box with the minima theme. And I must say, I actually quite liked it. But I also knew of quite a few sites already sporting this design, so I thought, why not quickly invest 15 minutes to see if I find another theme [here](https://jekyllrb.com/docs/themes/) or [here](https://duckduckgo.com/?q=jekyll+theme&ia=web). 4 hours later, I discovered my new design to be. The glorious and [minimal-mistakes](https://github.com/mmistakes/minimal-mistakes) would be the theme of my choosing for my new blog platform.

### Ensure old links still work

To be honest, I checked this out before I started migrating. The thing is that once you have a blog, you generally want to stick to whatever URL format your current platform has. If you change the URL and don\'t want to confuse the Googles, Bings and DuckDuckGos of this world, you would have to hardcode in redirects. And that is about as much fun as it sounds like. Luckily you can configure the path to your posts in the `config.yml` file:

```yaml
#
# So much config stuff
#

defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      # stuff...
      permalink: '/post/:slug'

# More config stuff
```



### The fiddling

And now started the fiddling part. For me, that was adjusting the styling in certain areas, aka adjusting the theme to my liking. I could have probably mitigated this by choosing a different theme or just going with the defaults. But then again, I really like how my site looks now.

Another big fiddle was the images. While most would render nicely, some of the phone screenshots would just blow up. Especially the recorded sequences I put into some blogposts. But with some CSS magic, I finally got to a solution that was satisfying enough for me.

Once everything was running nicely locally, I pushed it up to GitHub. Using the personal repository [https://your-user-name.github.io](https://mallibone.github.io). I use CloudFlare, so I had to swap out the servers it was pointing at. Then a long breath, a push of a button, and up was my blog running on GitHub Pages.

CloudFlare comes with great perks, such as a free SSL certificate and distributed caching for free, was to be able to register redirects. It just so turned out that the RSS feed was one thing I could not redirect easily (read Mark could not figure out how 😇). So I ended up registering a [page rule](https://support.cloudflare.com/hc/en-us/articles/200172286-Configuring-URL-forwarding-or-redirects-with-Cloudflare-Page-Rules) on Cloudflare to make a permanent redirect, which I hope could be removed again in a few months if all RSS readers play nicely - one can dream, right? 😆

## Conclusion

So was it really worth it? In short, yes, for me, it was totally worth the experience. For one, it was fun learning and reading about these static websites, and I really think they have a lot of potential in the future. While my blog is primarily static, I could use this setup and then invoke an Azure Function to trigger some backend logic. So I think this approach is not only limited to blogs.

Further, the cost of running is now 0, which is a nice treat. So happily ever after? Well, there are still a few things I will have to figure out before I will be pleased. One is image resizing, which was done automatically by OWL. Another area is comments which can be done, but I just haven\'t gotten around to yet. If you have a comment feel free to reach out to me on Twitter for now 🙂

So this was my journey from MiniBlog to Jekyll. I hope many more blog posts will follow this one. You can find the complete script I used on [GitHub](https://github.com/mallibone/MiniBlogToMarkdown). Note that you will need to make some changes for it to work due to the hardcoded paths etc.

HTH

Photo by **[Ketut Subiyanto](https://www.pexels.com/@ketut-subiyanto?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)** from **[Pexels](https://www.pexels.com/photo/empty-apartment-with-packed-carton-boxes-before-moving-4246119/?utm_content=attributionCopyText&utm_medium=referral&utm_source=pexels)**