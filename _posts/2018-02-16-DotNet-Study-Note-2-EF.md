---
layout: post
title: C# Study Note 2 - Entity Framework 
key: 20180216
tags: EF-Core-2.0 DotNet C# Note
---

This is second part of DotNet study notes -- EF Core 2.0. 

### Introduction

As ORM framework, EF has three design approaches:

1. Database First -- Database --> EDM
2. Model First -- EDM --> Database
3. Code First -- Code --> Database

I had spent most of my time in code first for the last two weeks. Here are the key notes of my study journey.

### Corner stone

* DbContext which is container in the memory of all entities and states
    * communication to database
    * mapping between entities and database tables
    * tracing states of entities

* DbSet Entities inside DbContext


### Connecting to database

Connect to database is simple just override ```OnConfiguring``` method

```cs
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    // UseSqlServer for sql server
    // UseSqlite for sqlite
    // UseNpgsql for postgresql
}
```

### Mapping

There are two way to indicate mapping model:

1. Data Annotation: which is very convenient but should not be heavily used here for loose coupling purpose
2. Fluent API

As far as I am concerned, it is a good to decouple the class and mappers by using fluent apis. This is because we can focus on the main logic regardless of the data persist and it is easy to maintain and extend.

Fluent API could be achieved by override ```OnModelCreating``` method:

eg:

```cs 
protected override void OnModelCreating(DbModelBuilder modelBuilder)
{
    // configure mapper here
}

```

It is possible OnModelCreating methods would grow to hundreds lines. We could add a mapper layer for an aggregate by implement ```IEntityTypeConfiguration``` interface

```cs
public class SomeAggregateMapper : IEntityTypeConfiguration<SomeAggregate>
{
    public void Configure(EntityTypeBuilder<SomeAggregate> builder)
    {
        // configure aggregate mapper here
    }
}
```

then add mapper to ```Configurations``` in ```modelBuilder```

```cs
modelBuilder.ApplyConfiguration(new SomeAggregateMapper());
```

### One to One

A user has a profile

```cs
// user
public class User 
{
    public int Id { set; get; }
    public string Email { set; get; }
    public int ProfileId { set; get; }
    public Profile Profile { set; get; }
}

// user mapper
public class UserMapper : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.HasKey(c => c.Id);
        // profile not null
        // builder.HasOne(p => p.Profile).WithMany().HasForeignKey(p => p.ProfileId);
        // profile could be null
        builder.HasOne(p => p.Profile).WithMany().HasForeignKey(p => p.ProfileId);
    }
}

```

1. HasRequired / HasOptional as means in its name
2. WithMany() no parameters means no linking back
 

```cs
// profile
public class Profile
{
    public int Id { set; get; }
    public int FirstName { set; get; }
}

// profile mapper

public class UserMapper : IEntityTypeConfiguration<Profile>
{
    public void Configure(EntityTypeBuilder<Profile> builder)
    {
        builder.HasKey(c => c.Id);
    }
}
```


### One to Many

A profile has many projects add ```public virutal ICollection<Project> Projects { get; set; }``` to Profile class


```cs

public class Project
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int ProfileId { get; set; }
    public Profile Profile { get; set; }
}


public class ProjectMapper : IEntityTypeConfiguration<Project>
{
    public void Configure(EntityTypeBuilder<Project> builder)
    {
        builder.HasKey(c => c.Id);

        builder.HasOne(p => p.Profile).WithMany(p => p.Projects).HasForeignKey(p => p.ProfileId);
    }
}

```

### Many to Many

Tags and Keywords

```cs
// tag
public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<TagKeyword> TagKeywords { get; set; }
}

// keyword

public class Keyword
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ICollection<TagKeyword> TagKeywords { get; set; }
}

// TagKeyword

public class TagKeyword
{
    public int TagId { get; set; }
    public Tag Tag { get; set; }

    public int KeywordId { get; set; }
    public Keyword Keyword { get; set; }    
}

public class TagKeywordMapper : IEntityTypeConfiguration<TagKeyword>
{
    public void Configure(EntityTypeBuilder<TagKeyword> builder)
    {
        builder.HasKey(t => new { t.TagId, t.KeywordId });

        builder.HasOne(tk => tk.Tag).WithMany(t => t.TagKeywords).HasForeignKey(tk => tk.TagId);
        builder.HasOne(tk => tk.Keyword).WithMany(k => k.TagKeywords).HasForeignKey(tk => tk.KeywordId);
    }
}

```

### Self-reference 

1. One to many / parent - children 

```cs
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int ParentId { get; set; }
    public Category Parent { get; set; }
    public ICollection<Category> Children { get; set; }
}

public class CategoryMapper : IEntityTypeConfiguration<Category>
{
    public void Configure(EntityTypeBuilder<Category> builder)
    {
        builder.HasOne(tk => tk.Parent).WithMany(t => t.Children).HasForeignKey(tk => tk.ParentId);
    }
}

```

2. Many to many



### Inheritance

When it comes to inheritance in EF Core 2.0, there are three types of relations:

* TPH：Table per inheritance

![tph](/assets/img/tph.png)[^2]



* TPT：Table per type


![tpt](/assets/img/tpt.png)[^2]

* TPC：Table per Concrete Type 


![tpc](/assets/img/tpc.png)[^2]

 ### Conclusion

 This is just a first glance of EF, there are so many things need to be learnt during the development of projects.

### API Reference

For API lookup[^1]

Fluent API Methods |	Usage
--------------------|--------
HasDefaultSchema()	| Specifies the default database schema.
ComplexType()	| Configures the class as complex type.
HasIndex()	| Configures the index property for the entity type.
HasKey()	| Configures the primary key property for the entity type.
HasMany()	| Configures the Many relationship for one-to-many or many-to-many relationships.
HasOptional()	| Configures an optional relationship which will create a nullable foreign key in the database.
HasRequired()	| Configures the required relationship which will create non-nullable foreign key column in the database.
Ignore()	| Configures that the class or property should not be mapped to a table or column.
Map()	| Allows advanced configuration related to how the entity is mapped to the database schema.
MapToStoredProcedures()	| Configures the entity type to use INSERT, UPDATE and DELETE stored procedures.
ToTable()	| Configures the table name for the entity.
HasColumnAnnotation()	| Sets an annotation in the model for the database column used to store the property.
IsRequired()	| Configures the property to be required on SaveChanges().
IsConcurrencyToken()	| Configures the property to be used as an optimistic concurrency token.
IsOptional()	| Configures the property to be optional which will create nullable column in the database.
HasParameterName()	| Configures the name of the parameter used in stored procedure for the property.
HasDatabaseGeneratedOption()	| Configures how the value will be generated for the corresponding column in the database e.g. computed, identity or none.
HasColumnOrder()	| Configures the order of the database column used to store the property.
HasColumnType()	| Configures the data type of the corresponding column in the database for the property.
HasColumnName()	| Configures the corresponding column name in the database for the property.
IsConcurrencyToken()	| Configures the property to be used as an optimistic concurrency token.


**Ps**: The above are the cheat sheet for EF, there are several mutation for EF Core eg: HasOne

### Reference

[^1]: Entity Framework Tutorial 2018, **Entity Framework Tutorial**, https://msdn.microsoft.com/en-us/library/jj554200.aspx
[^2]: Inheritance with EF Code First 2018, **Enterprise .Net**, https://weblogs.asp.net/manavi