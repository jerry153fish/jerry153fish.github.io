---
layout: post
title: Job Seek Development Journal - 2
key: 20180128
tags: C# EF-Core Agile DDD Journal TDD
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

```cs
// Keyword.cs
namespace DotNetJobSeek.Domain
{
    public class Keyword
    {
        public int Id { get; set; }
        public string Name { get; set; }
        [NotMapped]
        public int Weight { get; set; }


        public virtual ICollection<TagKeyword> TagKeywords { get; } = new List<TagKeyword>();

        // [NotMapped]
        // public IEnumerable<Tag> Tags => TagKeywords.Select(e => e.Tag);
    }
}

// Tag.cs
namespace DotNetJobSeek.Domain
{
    public class Tag
    {
       public int Id { get; set; }
       public string Name { get; set; }
       // Version for future AI usage
       public int Version { get; set; }

       public virtual ICollection<TagKeyword> TagKeywords { get; } = new List<TagKeyword>();

    //    [NotMapped]
    //    public IEnumerable<Keyword> Keywords => TagKeywords.Select(e => new Keyword {Id = e.Keyword.Id, Name = e.Keyword.Name, Weight = e.Weight });
    }
}

// TagKeyword.cs
namespace DotNetJobSeek.Domain
{
    public class TagKeyword
    {
        public int TagId { get; set; }
        public Tag Tag { get; set; }

        public int KeywordId { get; set; }
        public Keyword Keyword { get; set; }
        // weight for matching algorithm
        public int Weight { get; set; }
    }
}

```

At this stage, I need to test many to many relations, Version / Weight fields for matching algorithm. Thus I need to setup the basic EF Infrastructure services and unit test.

### EF In Infrastructure

EF Dbcontext set up

```sh
# create EF project
dotnet new console -o src/Infrastructure/DotNetJobSeek.Infrastructure.E
cd src/Infrastructure/DotNetJobSeek.Infrastructure.EF/
# inside Domain add database connect for sqlserver or postgresql
# sqlite is essential here for unit test
dotnet add package Microsoft.EntityFrameworkCore.Sqlite
# postgresql 
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL
# sqlserver 
# dotnet add package Microsoft.EntityFrameworkCore.SqlServer # 
# add 
dotnet add package Microsoft.EntityFrameworkCore.Design

```

Then add 

```
<ItemGroup>
  <DotNetCliToolReference Include="Microsoft.EntityFrameworkCore.Tools.DotNet" Version="2.0.0" />
</ItemGroup>
```

to DotNetJobSeek.Infrastructure.EF.csproj

Finally

```sh
dotnet restore

dotnet add reference ../../DotNetJobSeek.Domain/DotNetJobSeek.Domain.csproj
```

#### Define [Mappers](https://jerry153fish.github.io/2018/02/16/DotNet-Study-Note-2-EF.html)

```cs
// KeywordMapper
namespace DotNetJobSeek.Infrastructure.EF
{
    public class KeywordMapper : IEntityTypeConfiguration<Keyword>
    {
        public void Configure(EntityTypeBuilder<Keyword> builder)
        {
            builder.HasIndex(k => k.Name)
            .IsUnique();
        }
    }
}

// TagMapper
namespace DotNetJobSeek.Infrastructure.EF
{
    public class TagMapper : IEntityTypeConfiguration<Tag>
    {
        public void Configure(EntityTypeBuilder<Tag> builder)
        {
            builder.HasIndex(t => t.Name)
            .IsUnique();
            
            builder.HasIndex(t => t.Version);
            builder.Property(t => t.Version).HasDefaultValue(0);
        }
    }
}

// TagKeyword Mapper
namespace DotNetJobSeek.Infrastructure.EF
{
    public class TagKeywordMapper : IEntityTypeConfiguration<TagKeyword>
    {
        public void Configure(EntityTypeBuilder<TagKeyword> builder)
        {
            builder.HasKey(t => new { t.TagId, t.KeywordId });

            builder.HasOne(tk => tk.Tag).WithMany(t => t.TagKeywords).HasForeignKey(tk => tk.TagId);
            builder.HasOne(tk => tk.Keyword).WithMany(k => k.TagKeywords).HasForeignKey(tk => tk.KeywordId);

            builder.HasIndex(t => t.Weight);
            builder.Property(t => t.Weight).HasDefaultValue(0);
        }
    }
}
```

#### Configure EFContext

1. EFContext could load dynamic connection string for test and different data persistance purposes.
2. Dynamic loading mappers

```cs
public class EFContext : DbContext
{

    public EFContext()
    { }
    public EFContext(DbContextOptions<EFContext> options)
        : base(options)
    { }
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        // set default connection string
        if(!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseNpgsql("Username=postgres;Password=hellopassword;Host=localhost;Port=5432;Database=jobseek;Pooling=true;", providerOptions=>providerOptions.CommandTimeout(60))
            .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking);
        }
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.ApplyConfiguration(new TagMapper());
        modelBuilder.ApplyConfiguration(new KeywordMapper());
        modelBuilder.ApplyConfiguration(new TagKeywordMapper());
    }

    public DbSet<Keyword> Keywords { get; set; }
    public DbSet<Tag> Tags { get; set; }
}
```

Then add migrations and init database if using pg or sqlserver

```sh

dotnet ef migrations add init
dotnet ef database update

```

### Unit Test 

TDD is essential for this project as I have to write many test cases to verify the thought and practice new knowledge during development. 


```sh
# add xunit test
dotnet new xunit -o test/DotNetJobSeek.Domain.Test
# add reference to Domain and EFContext
cd test/DotNetJobSeek.Domain.Test
dotnet add reference ../../src/DotNetJobSeek.Domain/DotNetJobSeek.Domain.csproj
dotnet add reference ../../src/Infrastruture/DotNetJobSeek.Infrastructure.EF/DotNetJobSeek.Infrastructure.EF.csproj
```

> According to Microsoft 2016[^1], SQLite has an in-memory mode that allows you to use SQLite to write tests against a relational database, without the overhead of actual database operations.

eg:

```cs
public class TagKeywordMapperTest
{
    Keyword k1, k2, k3;
    Tag t1, t2, t3;
    public TagKeywordMapperTest()
    {
        k1 = new Keyword { Id = 1, Name = "food" };
        k2 = new Keyword { Id = 2, Name = "drink" };
        k3 = new Keyword { Id = 3, Name = "hotel" };
        t1 = new Tag { Id = 1, Name = "bar"};
        t2 = new Tag { Id = 2, Name = "move"};
        t3 = new Tag { Id = 3, Name = "live"};
    }
    [Fact]
    public void TestTagAdd()
    {
        var connection = new SqliteConnection("DataSource=:memory:");
        Tag test;
        connection.Open();
        try
        {
            var options = new DbContextOptionsBuilder<EFContext>()
                .UseSqlite(connection)
                .Options;
            using(var context = new EFContext(options))
            {
                context.Database.EnsureCreated();
            }
            using(var context = new EFContext(options))
            {

                context.Tags.Add(t1);
                try
                {
                    context.SaveChanges();
                }
                catch (System.Exception)
                {
                    throw;
                }
            }
            using(var context = new EFContext(options))
            {
                test = context.Tags.Where(k => k.Id == 1).FirstOrDefault();
            }
            Assert.Equal("bar", test.Name);
            Assert.Equal(0, test.Version);
        }
        finally
        {
            connection.Close();
        }

    }

}
```

### CI / CD

As this project is hosted in gitlab, thus the CI / CD could be enabled by add .gitlab-ci.yml

```yml
image: microsoft/dotnet:latest

stages:
  - test

before_script:
  - "dotnet restore"

test:
  stage: test
  script:
    - "dotnet test test/*"
  only:
    - master
    - develop
```

### Conclusion

In this journey, I spent most of my time in learning EF Core and C# Unit test. In the next journey, I will try to implement entities and repository.

[source code](https://gitlab.com/jerry153fish/dot-net-job-seek/tree/9e4272d378c7894b49931ea4c7ebbe1666657d95)

### Reference

[^1]: Microsoft 2016, **Testing with SQLite**, https://docs.microsoft.com/en-us/ef/core/miscellaneous/testing/sqlite
