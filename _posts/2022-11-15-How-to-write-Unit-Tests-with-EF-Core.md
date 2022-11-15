---
layout: single
title: "Getting started writing automated tests when using EF Core"
date: 2022-11-15 10:03
tags: ["EF Core", "Automated Testing", "Unit Testing"]
slug: "efcore-unittesting-pattern"

---

After making changes or adding a few lines of code to your app and all tests are green, I have come to greatly appreciate the soothing feeling in my life as a developer. It goes without saying I am a huge fan of writing automated tests to verify that I have not made a mistake or, worse, broken some aspect of an app that was perfectly fine before my change. This holds true when writing code that persists data to a database using [EF Core](https://learn.microsoft.com/en-us/ef/core/?WT.mc_id=AZ-MVP-5003494). In this post, I want to dive into a few aspects that have helped me write unit tests with parts of code that use EF Core along the way.

<!-- more -->

But before we get to writing tests, there are a few bases to cover first.

## Managing the database dependency

Stubbing or mocking a particular dependency is often required when writing automated tests. This allows us to ensure the test will always get or receive the same data and make it easier to test logic within the class under test. To ensure your code is not tightly coupled to an implementation, one often injects the dependency into the class via the constructor. If you are writing ASP.NET Core or .NET MAUI apps, this has become the default if you use the dependency service during the app's startup. The same is true for EF Core dependencies. If we inject them via the constructor, we can control how the database context has been set up in our tests. So the first step to enabling your code to become testable is to inject the EF Core database context:

```c#
[ApiController]
[Route("[controller]")]
public class GnabberController : ControllerBase
{
    private readonly GnabberContext _context;

    public BloggersController(GnabberContext context)
    {
        _context = context;
    }
  
    // ... where the magic happens with the context
}
```



## Speeding up your tests

Another consideration when testing with EF Core is: Do we really want to hit an existing database? While there are arguments for writing tests that invoke an actual database, there is quite an impact on the execution time of your tests. I write almost all of my tests using an in-memory database provider. The [In-Memory database](https://learn.microsoft.com/en-us/ef/core/providers/in-memory/?tabs=dotnet-core-cli&WT.mc_id=AZ-MVP-5003494) provided by the EF Core team offers an easy way to create such an in-memory database for testing.

> ***In-memory vs actual database (MS SQL et al.):*** Generally speaking, it is a fact that Input / Output operations, or IO for short, take more time to access something stored than something that is in the computer's RAM. So if we write our test accessing an actual database, we will have tests that take a lot longer because the data will be served from another service that will probably have to read the information from a drive. While it is true that many databases have excellent caching capabilities, automated tests tend to benefit less from them. One paradigm of writing automated tests is that they should be executed in isolation. This requires resetting the entire database before every test run. As a general rule of thumb: If you are testing something specific to a database implementation or not covered by the in-memory database. Write your test against a real database engine. You should be fine using the in-memory database in all other cases and enjoy the speed bump.
>
> If you are interested in the scales of accessing the different data storage on a computer. Check out the [blog](https://blog.codinghorror.com/the-infinite-space-between-words/) post by [Jeff Atwood, aka Coding Horror](https://blog.codinghorror.com/about-me/), that sums this topic up nicely.

After adding the NuGet package,  a DB context is created as follows:

```c#
var dbContext = new GnabberContext(new DbContextOptionsBuilder<BloggerContext>().UseInMemoryDatabase(dbName)
    // don't raise the error warning us that the in memory db doesn't support transactions
    .ConfigureWarnings(x => x.Ignore(InMemoryEventId.TransactionIgnoredWarning))
    .Options);
return dbContext;
```

Note that given the same name, we can create multiple "fresh" DBContexts. All the Contexts use the same in-memory database. This is an excellent feature, as we will see later. But furthermore, it allows for easily parallelizing tests without having them interfere with each other. For example, if we use a GUID in the test setup, we can ensure that every test run will use its own DB:

```c#
public class GnabberControllerShould
{
    public GnabberControllerShould()
    {
        var dbName = Guid.NewGuid().ToString();
        var testDbContext = CreateInMemoryContext(dbName);
    }
    
    [Fact]
    public async Task DoSomethingMeaningfulAndBeVerifiedInThisTest()
    {
        // the test code
    }
}
```

The code above is built using [XUnit](https://xunit.net/). But this can also be achieved in the setup methods of other unit test frameworks such as MSTest, NUnit, etc.

## Writing tests

With the setup out of the way, let's look at writing actual tests. This little example app allows seeing what blog posts a person has written. A controller is used for retrieving all the people and their blog posts:

```c#
[ApiController]
[Route("[controller]")]
public class BloggersController : ControllerBase
{
    private readonly BloggerContext _context;

    public BloggersController(BloggerContext context)
    {
        _context = context;
    }

    [HttpGet(Name = "GetBloggers")]
    public async Task<IReadOnlyCollection<Person>> Get()
    {
        var bloggers = 
            await _context
            .People
            // .Include(p => p.BlogPosts) // the "bug"
            .ToListAsync();
        return bloggers;
    }
}
```

The commented line inserts a little bug we hope to catch with the automated test. With the dependency injection implemented and equipped with the knowledge of an in-memory database. The only thing left to do is to write an actual automated test. Following the classical Arrange, Act, Assert setup, let's dive into the test:

```c#
public async Task GoodTestReturnAllPeopleAndTheirBlogpostss()
{
    // Arrange
    var testDbName = Guid.NewGuid().ToString();
    var testDbContext = CreateInMemoryContext(testDbName);
    var setupDbContext = CreateInMemoryContext(testDbName);
    
    SeedTestDatabase(setupDbContext); // Insert one person with two blog posts
    var sut = new BloggersController(testDbContext);
    
    // Act
    IReadOnlyCollection<Person> bloggers = await sut.Get();
    // Assert
    Assert.Single(bloggers);
    Assert.Equal(2, bloggers.First().BlogPosts?.Count ?? 0);
}

```

Using a different context for the setup and execution is critical. EF Core will cache information to reduce roundtrips to the database. If the context from the setup is reused for the test, the forgotten line to correctly request the blog information will slip through the test.

## Conclusion

Writing automated tests can be a great help in writing a maintainable code base. Using EF Core should not put a stop to that. By using dependency injection, we make our app code testable. Using an in-memory database can significantly speed up the execution time of tests. However, it may not be suitable if you intend to test aspects unavailable in the in-memory database and require the real database engine used.

Last but not least, it is essential to separate the contexts used for setting up the tests and the actual execution or else the tests will not catch bugs. I hope you can see in this post how you can set up your apps and tests to write successful tests. You can find the entire sample app on [GitHub](https://github.com/mallibone/EfCoreTesting).

HTH

