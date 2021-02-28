---
layout: single
title: "Keeping your git log meaningful"
title: Keeping your git log meaningful
date: 2015-08-02
tags: ["General", "git"]
slug: "keeping-your-git-log-meaningful"
---

[![git-logo_thumb2]({{ site.url }}{{ site.baseurl }}/images/84bb742f-8265-4ce1-9e6b-3f2522d6afdf.jpg "git-logo_thumb2")]({{ site.url }}{{ site.baseurl }}/images/85b628c5-e7ef-4c4d-9bc0-cc567f483da4.jpg)
 
In this blog post we will look at how we can merge multiple commits into a single one with the goal of reducing the chatter in a git log. This technique is generally known as *squashing*.
 
# When to use it
 
When working on a new feature (User Story) it is best practice to often commit. This gives you the option of always reversing back to a safe spot without loosing a great amount of work. But once you commit the changes to a remote repository it would be nice to get the quintessence of your work within one/a few meaningful commit(s). Often all those small safety hooks no longer are helpful on the server repository and may even harm the flow of reading through the log and getting the big picture.
 
# How is it done
 
You can either rebase based on the commit count you want to squash or the commit id:


    git rebase -i 003ee6c6f34516ad325a72d430e5a269e470b4d3


Alternatively you can set the count of commits you want to rebase:


    git rebase -i HEAD~4


Here we are telling git to rebase the last four commits. After executing the command your editor of choice for commit messages will popup:


    pick 5629556 Adds second paragraph
    pick 56ea060 Adds third paragraph
    pick 892c090 Adds final paragraph
    
    # Rebase 003ee6c..892c090 onto 003ee6c
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out


Simply change the commands to **squash** i.e. **s** and your commits will be squashed into one single commit. Note you want to pick the starting point of your squashing:


    pick 5629556 Adds second paragraph
    squash 56ea060 Adds third paragraph
    squash 892c090 Adds final paragraph
    
    # Rebase 003ee6c..892c090 onto 003ee6c
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out


Which merge your commit messages i.e. lets you edit the message that will be associated with the commit:


    Some very meaningful merge commit message comes here.
    
    # This is a combination of 3 commits.
    # The first commit's message is:
    Adds second paragraph
    
    # This is the 2nd commit message:
    
    Adds third paragraph
    
    # This is the 3rd commit message:
    
    Adds final paragraph
    
    # Please enter the commit message for your changes. Lines starting
    # with '#' will be ignored, and an empty message aborts the commit.
    # rebase in progress; onto 003ee6c
    # You are currently editing a commit while rebasing branch 'master' on '003ee6c'.
    #
    # Changes to be committed:
    #    modified:   someFile.txt
    #


# When to leave your fingers from this technique


> Do not rebase commits that exist outside your repository.


With rebase you are changing the history of the git repository. If you do this on a history that you have published and that other people are using.. Let’s just say you will not be rewarded by many friendly words. As you will mangle the history which they are using to work upon leading into a big mess. So **only apply rebasing to local repositories** which you haven’t published yet. An exception is if you are the only committer to a repository (and do not have any other copies with work lying around..).

# Conclusion

Using **git rebase** enables you to modify the commit history and make it more readable and less clunky. This helps you and your co-committers to have a nice log history with meaningful commits. Further it allows developers while coding to commit often to the local repository and easily go back in time if an approach has proven futile. All without littering the log history of the project and ending up with a ton of non-meaningful log entries.

Do however never rebase on public history as this might lead to tears and intense discussions.

## References:

[https://help.github.com/articles/about-git-rebase/](https://help.github.com/articles/about-git-rebase/ "https://help.github.com/articles/about-git-rebase/")

[http://www.git-scm.com/book/en/v2/Git-Branching-Rebasing](http://www.git-scm.com/book/en/v2/Git-Branching-Rebasing "http://www.git-scm.com/book/en/v2/Git-Branching-Rebasing")
