---
layout: single
title: "Accessing a private Azure DevOps NuGet feed when using Docker build containers"
date: 2023-12-31 00:03:00
tags: ["Docker", "DevOps", "NuGet", "Azure DevOps"]
slug: "docker-build-private-nuget-feed"
---

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/20231231_DockerPrivateArtefacts.jpg" alt="Docker private Azure DevOps artefacts feed" style="zoom:50%;" />

Using [Docker](https://www.docker.com/) for your .NET builds gives you a reproducible way to execute your builds on your build server and developers' devices. Setting up Docker to build your .NET applications is a straightforward process, but what if you use a private [Azure DevOps](https://azure.microsoft.com/en-us/products/devops/) NuGet feed that has to be accessed to build your app? Well, let's find out. 🤓

<!-- expand -->

In this example, Docker is used to build an ASP.NET Core application. Let's start by looking at a Dockerfile defining the image we want to create for our build. Microsoft offers a few of the [images](https://hub.docker.com/_/microsoft-dotnet-sdk) we can use for this task. At the time of writing, .NET 8 was the latest image, so we will be using this for our app:

```dockerfile
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build

COPY src/. /source

WORKDIR /source/Hello.AspNet

RUN --mount=type=cache,id=nuget,target=/root/.nuget/packages \
    dotnet publish --use-current-runtime --self-contained false -o /app

# Rest of the file using the build output
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

# Copy everything needed to run the app from the "build" stage.
COPY --from=build /app .

# ... run app
```

> One advantage of using Docker to build your app is that it will create a reproducible build environment. We can ensure that even in a few years when .NET 8 is no longer the current standard, we can use the same image used during the app's creation.

It is not uncommon to extract standard functionality into a library. Many companies use private feeds to post libs for reuse in other closed-source applications. Azure DevOps provides a way to host said libraries. A private feed for NuGet, NPM, et al. can be created and used by publishing to [Artefacts](https://learn.microsoft.com/en-us/azure/devops/artifacts/start-using-azure-artifacts?view=azure-devops&ocid=aisc23_CSC_DT-MVP-5002881&tabs=nuget%2Cnugetserver%2Cnugettfs). But how can we enable our Docker to build a container to access that feed? First, we will have to inform NuGet what the package sources are. For this, we can define a *nuget.config* file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!--To inherit the global NuGet package sources remove the <clear/> line below -->
    <!-- <clear /> -->
    <add key="Private Feed" value="https://pkgs.dev.azure.com/my-org/_packaging/my-private-feed/nuget/v3/index.json" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

You can place the *nuget.conifg* at the root of the project. Alternatively, copy the file before the build starts:

```dockerfile
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build

COPY src/. /source
COPY nuget.config /source/Hello.AspNet

# ... unchanged from the docker file above
```

The `dotnet build` will know where to look but will need authorization access. We can use a *Personal Access Token*, or *PAT* for short. The *PAT* will require read access enabled under the Packaging section. Creating a PAT is a simple process, and I will point you to the [official docs](https://learn.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&wt.mc_id=DT-MVP-5002881&tabs=Windows) to ensure that a future redesign will not be the source of any confusion. 😉  

> If you create a PAT, you can reuse it. However, since the generated code is only shown right after generating it, storing it in your password manager of choice is a good idea if you want to reuse the PAT for other build tasks.

With the PATat hand, we now have different options for using it. The most straightforward way is to put it into the NuGet.config file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!-- The package sources -->
  </packageSources>
  <packageSourceCredentials>
    <keyName>
      <add key="Username" value="Will-be-ignored" />      
      <add key="ClearTextPassword" value="THE-PAT-GENERATED-BY-AZURE-DEVOPS" />
    </keyName>
  </packageSourceCredentials>
</configuration>
```

While adding the PAT credentials directly to the file will fix the authorization issue. It will also mean that your PAT to source control. It could be better, but what are the alternatives? One way would be to set the PAT as an environment variable. So we leave the nuget.config as follows:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!--To inherit the global NuGet package sources remove the <clear/> line below -->
    <!-- <clear /> -->
    <add key="Private Feed" value="https://pkgs.dev.azure.com/my-org/_packaging/my-private-feed/nuget/v3/index.json" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <keyName>
      <add key="Username" value="Will-be-ignored" />      
      <add key="ClearTextPassword" value="%THE_PAT%" />
    </keyName>
  </packageSourceCredentials>
</configuration>
```

And set the environment variable using Docker as follows:

```dockerfile
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0 AS build

COPY src/. /source

WORKDIR /source/Hello.AspNet

ARG FEED_PAT

# ... build and run app
```

If you are triggering the Docker container build via the Azure DevOps build pipeline, you can set variables or give your build run access via [the NuGetAuthenticate build step](https://stackoverflow.com/questions/63158785/azure-devops-nuget-artifact-feed-and-docker). This will allow your build to access the private NuGet feed without the PAT showing up in any of your files, i.e. git repo.

HTH

