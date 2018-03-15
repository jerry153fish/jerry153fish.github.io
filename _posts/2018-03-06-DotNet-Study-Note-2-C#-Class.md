---
layout: post
title: DotNet Study Note 3 - C# Class
key: 20180306
tags: C# DotNet Note Class
---

This is third part of DotNet study notes -- quick review for C# language - class.


### Class

A class is a user-defined type that is composed of field data (often called member variables)
and members that operate on this data (such as constructors, properties, methods, events, and so forth). Collectively, the set of field data represents the “state” of a class instance (otherwise known as an object)

All classes are provided with a free default constructor and it will be removed as soon as there is a custom constructor with any number of parameters defined.

This keyword that provides access to the current class instance and for a class using a technique termed constructor chaining.

```cs

class Person
{
    public int Age;
    public string Name;

    public Person()
    : this(10, 'jerry')
    {}

    public Person(int age)
    : this(age, 'jerry')
    {}

    public Person(string name)
    : this(10, name)
    {}

    public Person(int age, string name)
    {
        this.Age = age;
        this.Name = name;
    }
}
```

#### Static

- static member
- static function
- static constructor

### The Pillars of OOP

#### Encapsulation

Hide unnecessary implementation details from the object user.

C# Access Modifier | May Be Applied To | Meaning in Life 
---|---|---
public |Types or type members| Public items have no access restrictions. A public member can be accessed from an object, as well as any derived class. A public type can be accessed from other external assemblies. 
private | Type members or nested types | Private items can be accessed only by the class (or structure) that defines the item.
protected | Type members or nested types | Protected items can be used by the class that defines it and any child class. However, protected items cannot be accessed from the outside world using the C# dot operator.
internal | Types or type members | Internal items are accessible only within the current assembly. Therefore, if you define a set of internal types within a .NET class library, other assemblies are not able to use them.
protected internal | Type members or nested types | When the protected and internal keywords are combined on an item, the item is accessible within the defining assembly, within the defining class, and by derived classes.

> Default Access Modifiers: type members are implicitly private while types are implicitly internal. 

```cs
// same codes

class Person
{
    Person()
    {}
}

internal class Person
{
    private Person()
    {}
}
```

> private, protected, and protected internal access modifiers can be applied to a nested type. non-nested types  can be defined only with the public or internal modifiers

- Automatic Properties

ValueType: the hidden backing fields will be assigned a safe default value, reference type: NULL

```cs

class Person
{
    // default: 0
    public int Age { get; set; }
    // initialize
    public string Name { get; set; } = "jerry"
    
}
```

- object initializer syntax

```cs

Person p1 = new Person { Age = 3; Name = "Tom" };
```

- const vs read-only

Same: can not be cannot be changed after the initial assignment

diff: 

const | read-only
must be known at compile time and implicitly static | can be determined at runtime and not implicitly static

```cs
class Person
{
    // compiled time 
    const int DesiredAge = 18;
    readonly int TargetAge; // as for static read only value if initialize must use static ctor
    public Person(int age)
    {
        // error
        DesiredAge = age;
        // can be assigned in runtime
        TargetAge = age;
    }
}
```


#### Inheritance

Build new class definitions based on existing class definitions

> Although constructors are typically defined as public, a derived class never inherits the constructors of a parent class. Constructors are used to construct only the class that they are defined within, although they can be called by a derived class through constructor chaining. [^1]

- sealed

The compiler will not allow you to derive from this type eg utility class

#### Polymorphism

Treat related objects in a similar manner



<a class="fabox" href="/assets/img/csharp/Csharp-review.png" target="_blank"><img src="/assets/img/csharp/Csharp-review.png" alt=""/></a>

 
### Reference

[^1]: Christian N 2018, **Professional C# 7 and . NET Core 2. 0**, https://www.amazon.com/Pro-NET-Core-Andrew-Troelsen/dp/1484230175

