---
layout: single
title: "Typora - a WYSIWYG markdown editor"
date: 2021-07-03 10:03:00
tags: ["blogging", "markdown"]
slug: "typora"
---

I have been using [Typora](https://typora.io/) now for a few years to edit all my markdown documents. And since I have now migrated my blog over to [GitHub pages](https://mallibone.com/post/miniblog-to-jekyll), it has become a tool I use even more. There are many favourite markdown editors out there, but this one is mine. So let me show you a few highlights.

<!--more-->

## Installation

You can download and install Typora from [the official website](https://typora.io/). You can get it for Windows, macOS and Linux - so I hope that should have you covered. I did not dig into it, but I am guessing it is an Electron-based app, so you will get the same features and look and feel (for the most part) on each platform. Or at least on Windows and macOS, which are my primary drivers.

## The experience

I have used a few markdown editors in the past. What I like about Typora is how it gets out of your way. There is no in your face menus or buttons, just the clean writing experience. Since it renders the output of the markdown file in real-time, you get a feel of the finished document. Other editors have a side-by-side preview, which is nice, but it does not feel the same and forces you to work in split-screen mode.

Different as in, e.g. Word, the text is formatted using markdown control characters. So you can add a # infront of a text on a new line, and it will become a heading one (h1). This combination makes for a superb typing experience, in my opinion, since my hands never have to leave the keyboard for formatting the text. If I do not know how to insert a specific markdown control, I can find it in a menu.

> Should you prefer to see the markdown's source, you can always switch between the raw markdown mode and the What-You-See-Is-What-You-Get (WYSIWYG) experience.

Adding code to blogposts has always been a somewhat "interesting" task. With markdown, code can be embedded, either directly in the text `someCode` or a code block for multiple lines:

```ocaml
// some comment
let theCode = "be part of markdown."
```

Markdown provides many features out of the box, and Typora will enable you to find them and write good looking markdown documents. For example, when it comes to links, I never remember the order of the brackets.

![Trying to remember how markdown links are written.]({{ site.url }}{{ site.baseurl }}/assets/images/2021-07-06-markdownLink.jpg "Trying to remember how markdown links are written.")

Having a WYSIWYG experience is a great help for knowing how the document will look in the end. Further, it shines when you are writing some complex structures like tables in markdown:

| Titel A          | TitelB                              | TitelC                                       |
| ---------------- | ----------------------------------- | -------------------------------------------- |
| Some text        | Don't know what to write here       | Some comment regarding the cell to the left. |
| Different text   | Still no inspiration                | That cell to the left sure lacks inspiration |
| Yet another text | Running out of nothingness to write | Someone make this stop please...             |

Using Typora, this was relatively straightforward to implement. But if we would have to do this in raw markdown:

```markdown
| Titel A          | TitelB                              | TitelC                                       |
| ---------------- | ----------------------------------- | -------------------------------------------- |
| Some text        | Don't know what to write here       | Some comment regarding the cell to the left. |
| Different text   | Still no inspiration                | That cell to the left sure lacks inspiration |
| Yet another text | Running out of nothingness to write | Someone make this stop please...             |
```

I hope you see why I rarely write markdown tables in raw. 😆

## Playing nice with others

While markdown has a significant advantage, it is only text. Being text-based means a markdown file can be opened and edited in the most basic editor, such as the good old Notepad on Windows, to open and edit the file. Then again, sometimes you want a different file format. Along with the likes of Word, PDF or HTML (the actual list is a lot longer...) and this is something you can do with Typora. You can export the markdown into different formats. You can even import from Word files as markdown, though your mileage may vary on this one.

I especially like the PDF functionality when I have to share a document with a client or manager who might not know what to do with a markdown file.

## Conclusion

I think it comes as no surprise that I am writing these lines using Typora. I wrote this post because I have been using Typora a lot over the past few years, and it has never let me down when I had to write some markdown. Be it for a blog post, a document in a project or editing that readme for a GitHub project. As of writing, Typora is a free product though this might change in the future. Be sure to check out the [official website](https://typora.io/), as there are a few more features, such as math formulas, which I never had the pleasure of using.

Thank you, [Juergen](https://twitter.com/sharpcms), for pointing me to Typora back at the MVP Summit.

HTH