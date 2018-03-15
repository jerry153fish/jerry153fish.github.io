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


### Encapsulation

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


### Inheritance

Build new class definitions based on existing class definitions

> Although constructors are typically defined as public, a derived class never inherits the constructors of a parent class. Constructors are used to construct only the class that they are defined within, although they can be called by a derived class through constructor chaining. [^1]

- sealed

The compiler will not allow you to derive from this type eg utility class

### Polymorphism

Treat related objects in a similar manner

- virtual and override vs Member Shadowing

#### Abstract class

When a class has been defined as an abstract base class (via the abstract keyword), it may define any number of abstract members. Abstract members can be used whenever you want to define a member that does not supply a default implementation but must be accounted for by each derived class.

Class Casting

```cs

class Person
{}

class Teacher: Person
{}

class Student: Person
{}

// implicit casting
Person t1 = new Teacher();
Person s1 = new Student();

// explicit casting
// is evaluated at runtime, not compile time
Teacher t2 = (Teacher)t1;

// as keyword

t1 as Teacher; // return Teacher reference type
t1 as Student; // return null

t1 is Teacher t2; // if is assign to t2

t1 is Student t2; // if not return false

```
> System.Object

In the .NET universe, every type ultimately derives from a base class named System.Object, which can be represented by the C# object keyword.

Instance Method of Object Class | Meaning in Life [^1]
---|---
Equals() | By default, this method returns true only if the items being compared refer to the same item in memory. Thus, Equals() is used to compare object references, not the state of the object. Typically, this method is overridden to return true only if the objects being compared have the same internal state values (that is, value-based semantics). Be aware that if you override Equals(), you should also override GetHashCode(), as these methods are used internally by Hashtable types to retrieve subobjects from the container. Moreover the ValueType class overrides this method for all structures, so they work with value-based comparisons. 
Finalize() | This method (when overridden) is called to free any allocated resources before the object is destroyed. 
GetHashCode() | This method returns an int that identifies a specific object instance. By default, System.Object.GetHashCode() uses your object’s current location in memory to yield the hash value.
ToString() | This method returns a string representation of this object, using the <namespace>.<type name> format (termed the fully qualified name). This method will often be overridden by a subclass to return a tokenized string of name/value pairs that represent the object’s internal state, rather than its fully qualified name.
GetType() | This method returns a Type object that fully describes the object you are currently referencing. In short, this is a Runtime Type Identification (RTTI) method available to all objects.
MemberwiseClone() | This method exists to return a member-by-member copy of the current object, which is often used when cloning an object.

> System.Exception

```cs
public class Exception : ISerializable, _Exception
{
    // Public constructors
    public Exception(string message, Exception innerException);
    public Exception(string message);
    public Exception();
    // ....
    // Methods
    public virtual Exception GetBaseException();
    public virtual void GetObjectData(SerializationInfo info, StreamingContext context);
    public virtual IDictionary Data { get; }
    public virtual string HelpLink { get; set; }
    public Exception InnerException { get; }
    public virtual string Message { get; }
    public virtual string Source { get; set; }
    public virtual string StackTrace { get; }
    public MethodBase TargetSite { get; }
    //....
}
```

The Exception interface allows a .net exception to be processed by an unmanaged codebase (such as a CoM application), while the ISerializable interface allows an exception object to be persisted across boundaries (such as a machine boundary).


System.Exception Property | Meaning in Life
--- | ---
Data | This read-only property retrieves a collection of key-value pairs (represented by an object implementing IDictionary) that provide additional, programmer-defined information about the exception. By default, this collection is empty.
HelpLink | This property gets or sets a URL to a help file or web site describing the error in full detail.
InnerException | This read-only property can be used to obtain information about the previous exceptions that caused the current exception to occur. The previous exceptions are recorded by passing them into the constructor of the most current exception.
Message | This read-only property returns the textual description of a given error. The error message itself is set as a constructor parameter.
Source | This property gets or sets the name of the assembly, or the object, that threw the current exception.
StackTrace | This read-only property contains a string that identifies the sequence of calls that triggered the exception. As you might guess, this property is useful during debugging or if you want to dump the error to an external error log.
TargetSite | This read-only property returns a MethodBase object, which describes numerous details about the method that threw the exception (invoking ToString() will identify the method by name).

#### interface

An interface is nothing more than a named set of abstract members. 

- no data fields
- no constructors
- no implementation
- can have property, event, indexer
- as parameters / return values


### IEnumerable and IEnumerator Interfaces

```cs
public interface IEnumerable
{
   IEnumerator GetEnumerator();
}

public interface IEnumerator
{
    bool MoveNext ();
    object Current { get;}
    void Reset ();
}

// eg

public People : IEnumerable
{
    private Person[] pArray = new Person[2];
    public People()
    {
        pArray[0] = new Person{ Age = 3, Name = "jerry" };
        pArray[1] = new Person{ Age = 4, Name = "tom" };
    }
    public IEnumerator GetEnumerator()
    {
        // Return the array object's IEnumerator.
        return pArray.GetEnumerator();
    }
    // or by yield
    public IEnumerator GetEnumerator()
    {
        foreach (Person p in pArray)
        {
            yield return p;
        }
    }
}
```

### The ICloneable Interface

```cs
public interface ICloneable
{
   object Clone();
}

public class Person : ICloneable
{
    public int Age { set; get; }
    public string Name { set; get; }
    public Person(int age, string name)
    {
        Age = age;
        Name = name;
    }
    public Person() {}
    public object Clone() => new Person(this.Age, this.Name);
    // or use MemberwiseClone from object need to implement deep clone
    public object Clone() => this.MemberwiseClone();
}
```

### The IComparable Interface

Allows an object to be sorted based on some specified key

```cs
public interface IComparable
{
   int CompareTo(object o);
}

// eg

public class Person : IComparable
{
    public int Age { set; get; }
    public string Name { set; get; }
    // CompareTo Implementation
    int IComparable.CompareTo(object obj)
    {
        Person tempP = obj as Person;
        if ( temP == null)
            throw new ArgumentException("Parameter is not a Car!");
        if (this.Age > temP.Age)
            // comes before temP
            return 1;
        else if (this.Age < temP.Age)
            // comes after temP
            return -1;
        else
            return 0;
        // or steamline the previous CompareTo in this case Int
        if (temp != null)
            return this.Age.CompareTo(temp.CarID);
    }
}

// usage
Array.sort(people);
```

### IComparer Interface

IComparer is typically not implemented on the type you of sorting but on any number of helper classes.


```cs
interface IComparer
{
   int Compare(object o1, object o2);
}
// person name Comparer
public PersonNameComparer : IComparer
{
    int IComparer.Compare(object o1, object o2)
    {
        Person p1 = o1 as Person;
        Person p2 = o2 as Person;
        if(p1 == null || p2 == null)
            throw new ArgumentException("Parameter is not a Person!");
        return String.Compare(p1.Name, p2.Name);
    }
}

// usage
Array.Sort(people, new PersonNameComparer());
```
 
### Reference

[^1]: Christian N 2018, **Professional C# 7 and . NET Core 2. 0**, https://www.amazon.com/Pro-NET-Core-Andrew-Troelsen/dp/1484230175

