---
layout: post
title: Job Seek Development Journal - 2
key: 20180128
tags: c# enity rabbitmq redis sharpnltk agile DDD
---

Last week, I tried to fit this small job seek in DDD pattern. And this week, I am going to learn how to implement it.

### Preparing

To begin with, docker is used to set up external services in this project:

* rabbitmq

This is for message queue and event notice.

```sh
 docker run -d --hostname my-rabbit -p 5672:5672 --name first-rabbit rabbitmq:3
```

* Data persist 

Redis for cache, postgresql for data.  

```sh
docker run -p 6379:6379 --name first-redis -d redis # redis for cache
docker run -p 5432:5432 --name first-postgres -e POSTGRES_PASSWORD=hellopassword -d postgres # posgresql for data
# or sql server 2017
docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=Mypa33w0rd!' -p 1401:1433 --name sql1 -d microsoft/mssql-server-linux:2017-latest
```

* selenium with chrome

Web crawling

```sh
docker run -p 4444:4444 --name first-selenium -d selenium/standalone-chrome
```

### Project file structure

├── DotNetJobSeek.sln
├── DotNetJobSeek.userprefs
├── README.md
├── documents
│   ├── Job\ seek.xmind
│   ├── job\ seek.vpp
├── src
│   ├── DotNetJobSeek.UI
│   └── DotNetJobSeek.Application
│   └── DotNetJobSeek.Domain
│   └── Infrastructure
|        └── DotNetJobSeek.Scrpay
|        └── DotNetJobSeek.Utility 
|        └── DotNetJobSeek.Tag
|        └── DotNetJobSeek.Repository
|        └── DotNetJobSeek.MQ 
└── test
│   ├── UI.Tests
│   └── Application.Tests
│   └── Domain.Tests
│   └── Infrastructure.Tests
|        └── DotNetJobSeek.Scrpay.Tests
|        └── DotNetJobSeek.Utility.Tests
|        └── DotNetJobSeek.Tag.Tests
|        └── DotNetJobSeek.Repository.Tests
|        └── DotNetJobSeek.MQ.Tests

### Domain Implementation

1. EF Core set up

```sh
# create domain project
dotnet new console -o DotNetJobSeek.Domain

# inside Domain add database connect for sqlserver or postgresql
dotnet add package Microsoft.EntityFrameworkCore.SqlServer # 
# add 
dotnet add package Microsoft.EntityFrameworkCore.Design

```

Then add 

```
<ItemGroup>
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
</ItemGroup>
```

to DotNetJobSeek.csproj

```sh
dotnet restore
```

2. Add constrains 

Add constrains for assets in domain and will enrich them during the future development

```cs
// IEntity interface contains a object key
namespace DotNetJobSeek.Domain
{
    public interface IEntity
    {
        object Key { get;}
    } 
}

// IAggretage interface
namespace DotNetJobSeek.Domain
{
    public interface IAggregateRoot : IEntity
    {
        
    }
}

```
