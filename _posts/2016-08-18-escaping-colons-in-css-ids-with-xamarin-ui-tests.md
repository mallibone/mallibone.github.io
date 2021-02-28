---
layout: single
title: "Escaping colons in Xamarin UI Tests"
title: Escaping colons in Xamarin UI Tests
date: 2016-08-18
tags: ["Xamarin Test Cloud"]
slug: "escaping-colons-in-css-ids-with-xamarin-ui-tests"
---

I recently wrote a UI test for an app which displayed a CSS ID which included a colon. The UI tree from the REPL looked like this:
 
<script src="https://gist.github.com/mallibone/9bbbe9b5101f60b7263c5592d4f5c129.js"></script>
 
Googling did bring me closer to a possible [solution](http://stackoverflow.com/questions/122238/handling-a-colon-in-an-element-id-in-a-css-selector/122266#122266 "link to stackoverlfow question which is not the solution to this problem") by suggesting to escape the colon with a backslash, which ended in the following output:
 
<script src="https://gist.github.com/mallibone/a4212ae8843c3dc70d679e6094b9bd13.js"></script>
 
So no luck, before going into any detail here is how to solve the issue:
 
## iOS
 
<script src="https://gist.github.com/mallibone/9ed89f13cdf8cb28d4577b716ce5d557.js"></script>
 
## Android
 
<script src="https://gist.github.com/mallibone/67c609ff1c2fd3ecbbae52121be658e8.js"></script>
 
The reason why the colon has to be escaped that many times is because the string will be passed through multiple runtimes. Each will be interpreting and escaping the string anew. The differing count is due to how Xamarin UI Tests have to handle the platforms.
 
# Reference
 
Many thanks to Tobias Røikjer from the Xamarin UI Testing team for pointing out how to solve this problem.
