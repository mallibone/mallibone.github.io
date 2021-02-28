---
layout: single
title: "Book review: Building Microservices"
title: Book review&#58; Building Microservices
date: 2015-07-08
tags: ["General", "Book review"]
slug: "book-review:-building-microservices"
---

![](http://samnewman.io/img/book.jpg)
 
During my day job I’m mainly involved in mobile applications which often (like in always…) have to communicate with a backend that provides data and other services to the mobile app. The backend often needs to be more then a mere database gateway and is often extended with additional features which enables a rich mobile app experience and integration to other systems. These services change over time and when the app is successful are required to offer ever more functionality. Microservices is a new buzzword in town that comes with praises which allow a system to grow without slowing down the development and further allows for better fault tolerance due to it’s distributed nature.
 
[Building Microservices](http://samnewman.io/books/building_microservices/) is a book written by [Sam Newman](http://samnewman.io/about/) which covers the ideas and concepts behind microservices and gives an introduction into the challenges of monolithic systems and how microservices try to solve the demands towards todays services.
 
# A general take
 
The author assumes that the reader has a general understanding of application and service development. The author goes through great lengths to ensure the reader gets a general understanding of how services can be created and what the ideas are behind their designs. Thereby he highlights what the benefits and downsides are for each system. The reader is led through many different aspects of designing a microservice from communication protocols, datastorage even company culture which boosts the design of microservices are described in the book and introduce the concepts and patterns on which microservices are built on.
 
While the book contains a lot of conceptual information on microservices and development techniques it does not go into great detail on the topics. However if your interested you will find references to books, libraries, frameworks etc. in the book. So if you are looking for a step-by-step guide on how to create a microservice with language X using framework Y you will not be happy with this book. If however you are looking for an overview and introduction into microservices i.e. on how to leverage microservices in your projects this is the book for you - and if you are looking for a cookbook book, consider reading this book anyway as it will give you some great insight on what to look out for when creating your service ![Winking smile](http://www.mallibone.com/posts/files/3ccf50a8-e686-4920-bd8b-0c5a6f177325.png)
 
# The books content
 
Due to the high level approach the book covers a lot of topics and gives a good insight in what is to be expected, so lets have a glance at what is available in the books 280 pages:
 
1. Microservices
    - This chapter gives a basic introduction into the topic and introduces the idea behind microservices. Also explains why microservices are [no silver bullet](https://en.wikipedia.org/wiki/No_Silver_Bullet).
2. The Evolutionary Architect
    - Explains the role of an architect when having a system created of microservices.
3. How to Model Services
    - Defining boundaries of services and how this effects the data model being passed in between. This chapter takes input from Domain Driven Design and explains how these could be implemented in a service i.e. why they should be considered.
4. Integration
    - Explains how services can be integrated together. How one use different technology stacks for different tasks at hand without creating an error prone system.
5. Splitting the Monolith
    - This chapter shows how existing applications can be split up into microservices. The author generally advises to first start out with the monolith and only when the demand for microservices arises should the system be pushed into that direction.
6. Deployment
    - Having multiple services comes with a larger complexity when it comes down to deployment. In this chapter you see how to automate as much as you can for your services leaning heavily on Continuous Integration and Continuous Deployment methodologies.
7. Testing
    - This chapter shows how to automatically test your services with tests that go beyond unit tests. It introduces the concept of load tests. Further explains why system wide tests are not as useful as one would hope due to their brittleness and how to replace them with more modular tests.
8. Monitoring
    - This chapter shows how to effectively monitor multiple services deployed on multiple instances and what are the key metrics you should implement into your services.
9. Security
    - Perhaps one of the most important chapter shows the challenges faced when developing a service and how to secure the systems. It provides information on what are the current industry standards which you should use and how to manage data. Core take away: ***do not write your own security software*** - unless you are a security expert.
10. Conway’s Law and System Design
    - In this chapter the idea of [Conway’s law](https://en.wikipedia.org/wiki/Conway's_law) is applied to microservices and highlights how team organization will impact the design and architecture of your system for better or worse. These are backed up with some real world examples of companies such as Netflix and Amazon which went with the microservice design approach and built their teams/company culture accordingly.
11. Microservices at Scale
    - This chapter gives you a glance at what it means to have a business running on microservices. Dealing with failing services by degrading functionality, caching, discovering services and scaling (including database scaling) will be looked at in this chapter
12. Bringing it all together
    - The final chapter goes over the principles of microservices and gives a reminder when to **not** use microservices.

 
Generally speaking this book covers a lot of topics within 280 pages and the author does a great job in building up the knowledge of the reader. So much that I every time a new chapter started I found myself wanting to know how this additional puzzle piece fits into the picture. One notices while reading the book that the author has had a lot of real world experience and shares his insights throughout the book and giving hints on where to dig deeper if you ever need or want to do so on a given topic.
 
# Conclusion
 
Building microservices gives a great introduction into the concept behind microservices and shows why the approach is a great solution to certain problems which just happen to be a lot of apps we are writing these days ![Smile](http://www.mallibone.com/posts/files/87514f96-cf95-47eb-a32a-eab420db4ae5.png)
 
This book gives you a conceptual overview and stays clear of diving into technical solutions (though you will find links to notable libraries, frameworks etc. throughout the book) but explaining concepts on how to solve certain problems that occur in highly decoupled and distributed environments. The book therefore is not bound to a certain technology stack or development language.
 
If you are looking for a book that gives you a general overview of the topic or are looking for a book that gives you an introduction into microservices I can only recommend this book to you and hope you will enjoy reading it as much as I did.
 
Small Remark: I read this book on my Kindle and the formatting was flawless.
