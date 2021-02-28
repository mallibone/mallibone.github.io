---
layout: single
title: "Pushing a local repository to GitHub"
title: Pushing a local repository to GitHub
date: 2015-05-03
tags: ["General"]
slug: "pushing-a-local-repository-to-github"
---

[![GitToGitHub]({{ site.url }}{{ site.baseurl }}/images/248ce163-960e-4988-86e2-a114b8cd8578.png "GitToGitHub")]({{ site.url }}{{ site.baseurl }}/images/a757994b-b1c4-47a7-b714-fb6caa940cba.png)

I usually end up looking for these steps every time I write a blog post with a code sample that I in the end push up to [GitHub](https://github.com/). So the steps I do are:

1. Creating and committing to a local git repository
2. Create a GitHub repository with License (and readme)
3. Configure the local repository to push it to GitHub


Step one is usually done by creating a new Solution in Visual Studio which automatically adds the .gitignore file according to my C# project which is very helpful to not ending up committing the bin folder and the [NuGet](https://www.nuget.org/) package binaries.

After finishing my solution I go to GitHub and create a new repository, in the setup process you can add a License file and a blank readme. You could also add the .gitignore at this step but as some of my code gets thrown away as it is of no use I tend to first start locally and then push the solution once I feel it has some value to add.

Now once all is set up on GitHub I use the **Git Shell** which you get after installing [GitHub for Windows](https://windows.github.com/) . There navigate to the root of your project and add the repository (you find the URI after creating the repository) with the following line:


    git remote add origin https://github.com/your-account/the-repository-name


Next verify that your branch has been successfully added as a remote branch:


    git remote -v


First we have to get the remote branch to our local repository:


    git fetch


Then set the upstream branch:


    git branch --set-upstream-to=origin/main main


Pull the license and possible readme (et al):


    git pull


Now you can simply push your main branch to GitHub:


    git push


And all is good :-)

Technorati Tags: [Git](http://technorati.com/tags/Git),[GitHub](http://technorati.com/tags/GitHub),[GitHub for Windows](http://technorati.com/tags/GitHub+for+Windows)
