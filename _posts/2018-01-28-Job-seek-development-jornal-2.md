---
layout: post
title: Job Seek Development Journal - 2
key: 20180128
tags: c# enity rabbitmq redis sharpnltk agile DDD
---

Last week, I tried to fit this small job seek in DDD pattern. And this week, I am going to learn how to implement it.


### Project Setup

```cs
// create DotNetJobSeek solution
dotnet new sln -o DotNetJobSeek
// src: source codes folder
// test: test codes folder
// documents: documents foler
mkdir src test documents
// README file
touch README.md
// add Domain projects and corresponding test
dotnet new console -o src/DotNetJobSeek.Domain
dotnet new xunit -o test/DotNetJobSeek.Domain.Tests

```

```
├── DotNetJobSeek.sln
├── DotNetJobSeek.userprefs
├── README.md
├── documents
│   ├── Job\ seek.xmind
│   ├── job\ seek.vpp
├── src
│   ├── DotNetJobSeek.Domain
└── test
    └── DotNetJobSeek.Domain.Test

```

### Add Constrains

The first need to be done is to add constrains for assets in domain to distinguish Entity, ValueObject, AggregateRoot and Repository.

```cs
// IEntity interface contains a object key
namespace DotNetJobSeek.Domain
{
    public interface IEntity
    {
        // object key for string / int / Guid
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

// IRepository
namespace DotNetJobSeek.Domain
{
    public interface IRepository<T> where T : IAggregateRoot
    {

    } 
}

```

### ValueObjects

Any DAO without constrains in this project is valueObjects. 






3. Entities



1. EF Core set up

```sh
# create domain project
dotnet new console -o DotNetJobSeek.Domain

# inside Domain add database connect for sqlserver or postgresql
# sqlite dotnet add package Microsoft.EntityFrameworkCore.Sqlite
# postgresql dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
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