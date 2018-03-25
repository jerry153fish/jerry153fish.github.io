---
layout: post
title: DotNet Study Note 3 - C# Collection Generic
key: 20180306
tags: C# DotNet Note Collection Generic
---

This is forth part of DotNet study notes -- quick review for C# language - Collection Generic


### Collection

Collection classes are built to dynamically resize themselves on the fly as you insert or remove items.

> it is actually possible to change the size of an array using the generic Resize()<T> method.however, this will result in a copy of the data into a new array object and could be inefficient. [^1]

### Nongeneric collections (primarily found in the System.Collections namespace)

System.Collections Class | Meaning in Life | Key Implemented Interfaces
--- | --- | ---
ArrayList | Represents a dynamically sized collection of objects listed in sequential order | IList, ICollection, IEnumerable, and ICloneable
BitArray | Manages a compact array of bit values, which are represented as Booleans, where true indicates that the bit is on (1) and false indicates the bit is off (0) | ICollection, IEnumerable, and ICloneable
Hashtable | Represents a collection of key-value pairs that are organized based on the hash code of the key | IDictionary, ICollection, IEnumerable, and ICloneable
Queue | Represents a standard first-in, first-out (FIFO) collection of objects | ICollection, IEnumerable, and ICloneable
SortedList | Represents a collection of key-value pairs that are sorted by the keys and are accessible by key and by index | IDictionary, ICollection, IEnumerable, and ICloneable
Stack | A last-in, first-out (LIFO) stack providing push and pop (and peek) functionality | ICollection, IEnumerable, and ICloneable


>Interface description

System.Collections Interface | Meaning in Life
ICollection | Defines general characteristics (e.g., size, enumeration, and thread safety) for all nongeneric collection types
ICloneable | Allows the implementing object to return a copy of itself to the caller
IDictionary | Allows a nongeneric collection object to represent its contents using key-value pairs
IEnumerable | Returns an object implementing the IEnumerator interface (see next table entry)
IEnumerator | Enables foreach-style iteration of collection items
IList | Provides behavior to add, remove, and index items in a sequential list of objects


System.Collections.Specialized Type  | Meaning in Life
HybridDictionary | This class implements IDictionary by using a ListDictionary while the collection is small and then switching to a Hashtable when the collection gets large.
ListDictionary | This class is useful when you need to manage a small number of items (ten or so) that can change over time. This class makes use of a singly linked list to maintain its data.
StringCollection | This class provides an optimal way to manage large collections of string data.
BitVector32 | This class provides a simple structure that stores Boolean values and small integers in 32 bits of memory.

#### Issues of Nongeneric collections

- poorly performing code, especially when you are manipulating numerical data (e.g., value types). As you’ll see momentarily, the CLR must perform a number of memory transfer operations when you store structures in any nongeneric collection class prototyped to operate on System.Objects, which can hurt runtime execution speed.

- Not type-safe because (again) they were developed to operate on System.Objects, and they could therefore contain anything at all.


### Generic collections (primarily found in the System.Collections.Generic namespace)

- Generics provide better performance because they do not result in boxing or unboxing penalties when storing value types.
- Generics are type safe because they can contain only the type of type you specify.
- Generics greatly reduce the need to build custom collection types because can be specified the “type of type” when creating the generic container.

> only classes, structures, interfaces, and delegates can be written generically; enum types cannot.

Generic Class | Supported Key Interfaces | Meaning in Life
--- | --- | ---
Dictionary<TKey, TValue> | ICollection<T>, IDictionary<TKey, TValue>, IEnumerable<T>|This represents a generic collection of keys and values.
LinkedList<T> | ICollection<T>, IEnumerable<T> |This represents a doubly linked list.
List<T> | ICollection<T>, IEnumerable<T>, IList<T>|This is a dynamically resizable sequential list of items.
Queue<T> | ICollection (Not a typo! This is the nongeneric collection interface), IEnumerable<T>|This is a generic implementation of a first-in, first-out list.
SortedDictionary<TKey, TValue> | ICollection<T>, IDictionary<TKey, TValue>, IEnumerable<T>|This is a generic implementation of a sorted set of key-value pairs.
SortedSet<T> |ICollection<T>, IEnumerable<T>, ISet<T> |This represents a collection of objects that is maintained in sorted order with no duplication.
Stack<T> | ICollection (Not a typo! This is the nongeneric collection interface), IEnumerable<T> |This is a generic implementation of a last-in, first-out list.


> Generic Interfaces


System.Collections.Generic | Meaning in Life
--- | --- 
Interface ICollection<T> | Defines general characteristics (e.g., size, enumeration, and thread safety) for all generic collection types
IComparer<T> | Defines a way to compare to objects
IDictionary<TKey, TValue> | Allows a generic collection object to represent its contents using key-value pairs
IEnumerable<T> | Returns the IEnumerator<T> interface for a given object 
IEnumerator<T> | Enables foreach-style iteration over a generic collection
IList<T> | Provides behavior to add, remove, and index items in a sequential list of objects
ISet<T> | Provides the base interface for the abstraction of sets


> System.Collections.ObjectModel


System.Collections.ObjectModel Type | Meaning in Life
--- | --- 
ObservableCollection<T> | Represents a dynamic data collection that provides notifications when items get added, when items get removed, or when the whole list is refreshed
ReadOnlyObservableCollection<T> | Represents a read-only version of ObservableCollection<T>


### Generic

- default in Generic
    - Numeric values have a default value of 0.
    - Reference types have a default value of null.
    - Fields of a structure are set to 0 (for value types) or null (for reference types).


-
```cs

public class Point<T>
{
    public T Lat { set; get; }
    public T Lng { set; get; }
    public void Reset()
    {
        Lat = default(Lat);
        Lng = default(Lng);
    }
}
```

- type parameters

.Net platform allow use to use the where keyword to get extremely specific about what a given type parameter must look like.

Generic Constraint | Meaning in Life
--- | ---
where T : struct | The type parameter <T> must have System.ValueType in its chain of inheritance (i.e., <T> must be a structure).
where T : class | The type parameter <T> must not have System.ValueType in its chain of inheritance (i.e., <T> must be a reference type).
where T : new() | The type parameter <T> must have a default constructor. This is helpful if your generic type must create an instance of the type parameter because you cannot assume you know the format of custom constructors. Note that this constraint must be listed last on a multiconstrained type. **must be list at least**
where T : NameOfBaseClass | The type parameter <T> must be derived from the class specified by NameOfBaseClass.
where T : NameOfInterface | The type parameter <T> must implement the interface specified by NameOfInterface. You can separate multiple interfaces as a comma-delimited list.

```cs
// new must be at least
public class Person<T, K> where T : class, ILoveable, new()
    where K: struct, IEatable
{}
```

### Delegate

A delegate is a type-safe object that points to another method (or possibly a list of methods)in the application, which can be invoked at a later time.

- The address of the method on which it makes calls 
- The parameters (if any) of this method
- The return type (if any) of this method

> .net delegates can point to either static or instance methods.

In c

```c
typedef int (*fn)(int *arg); 

```

vs

```cs
// This delegate can point to any method,
// taking one integer and returning an integer.
public delegate int fn(int x);

// will become
sealed class fn : System.MulticastDelegate 
{
    public int Invoke(int x);
    public IAsyncResult BeginInvoke(int x,
    AsyncCallback cb, object state);
    public int EndInvoke(IAsyncResult result);
}
```

> When the C# compiler processes delegate types, it automatically generates a sealed class deriving from System.MulticastDelegate providing the necessary infrastructure for the delegate to hold onto a list of methods to be invoked at a later time.

### System.MulticastDelegate and System.Delegate

```cs
public abstract class MulticastDelegate : Delegate
{
    // Returns the list of methods "pointed to."
    public sealed override Delegate[] GetInvocationList();
    // Overloaded operators.
    public static bool operator ==(MulticastDelegate d1, MulticastDelegate d2);
    public static bool operator !=(MulticastDelegate d1, MulticastDelegate d2);
    // Used internally to manage the list of methods maintained by the delegate.
    private IntPtr _invocationCount;
    private object _invocationList;
}

public abstract class Delegate : ICloneable, ISerializable
{
    // Methods to interact with the list of functions.
    public static Delegate Combine(params Delegate[] delegates);
    public static Delegate Combine(Delegate a, Delegate b);
    public static Delegate Remove(Delegate source, Delegate value);
    public static Delegate RemoveAll(Delegate source, Delegate value);
    // Overloaded operators.
    public static bool operator ==(Delegate d1, Delegate d2);
    public static bool operator !=(Delegate d1, Delegate d2);
    // Properties that expose the delegate target.
    public MethodInfo Method { get; }
    public object Target { get; }
}
```

> never directly derive from these base classes in your code (it is a compiler error to do so but use delegate keyword


#### Select Members of System.MulticastDelegate/System.Delegate

Member |  Meaning in Life
--- | ---
Method | This property returns a System.Reflection.MethodInfo object that represents details of a static method maintained by the delegate.
Target | If the method to be called is defined at the object level (rather than a static method), Target returns an object that represents the method maintained by the delegate. If the value returned from Target equals null, the method to be called is a static member.
Combine() | This static method adds a method to the list maintained by the delegate. In C#, you trigger this method using the overloaded += operator as a shorthand notation.
GetInvocationList() | This method returns an array of System.Delegate objects, each representing a particular method that may be invoked.
Remove() RemoveAll() | These static methods remove a method (or all methods) from the delegate’s invocation list. In C#, the Remove() method can be called indirectly using the overloaded -= operator.

```cs

public class SimpleMath
{
    public static int TwoTimes(int x)
    {
        return x * 2;
    }
    public int ThreeTimes(int x)
    {
        return x * 3;
    }
}
static void Main(string[] args)
{
    Fn twoTimef = new Fn(SimpleMath.TwoTimes);
    DisplayDelegateInfo(twoTimef); // static method have not target information
    SimpleMath s1 = new SimpleMath();
    Fn twoTimef2 = new Fn(s1.ThreeTimes);
    DisplayDelegateInfo(twoTimef2);
    Console.WriteLine("twoTimef(3) = {0}", twoTimef(3));
}

/* output
Method Name: Int32 TwoTimes(Int32)
Type Name:
Method Name: Int32 ThreeTimes(Int32)
Type Name: TestDelegate.Program+SimpleMath
twoTimef(3) = 6
*/
```

### Object State Notifications by Delegates

- Define a new delegate type that will be used to send notifications to the caller.
- Declare a member variable of this delegate in the Car class.
- Create a helper function on the Class that allows the caller to specify the method to call back on.
- Implement the notify method to invoke the delegate’s invocation list under the correct circumstances.

```cs
// delegate with class
public class Person
{
    public int Age { set; get; }
    public string Name { set; get; }
    // 1. define a delegate type
    public delegate void GetOlderHandler(string megGetOlder);
    // 2. define a member of hander
    private GetOlderHandler humanGetOlderHandlers;
    // 3. define registration function for the caller
    public void RegisterWithGetOlder(GetOlderHandler hander)
    {
        humanGetOlderHandlers += hander;
    }
    // 4. function to trigger invoke 
    public void Grow(int sleepYear)
    {
        Age += sleepYear;
        if (Age > 60)
            humanGetOlderHandlers(String.Format("I am {0}", Age));
    }
}

public static void onGetOlder(string msg)
{
    System.Console.WriteLine("oh no {0}, go to Gym!!", msg);
}
public static void onGetOlder2(string msg)
{
    System.Console.WriteLine("oh no {0}, go go to KFC!!", msg);
}
static void Main(string[] args)
{

    Person tom = new Person { Age = 40, Name = "tom"};

    tom.RegisterWithGetOlder(new Person.GetOlderHandler(onGetOlder));
    // as long as the function signature match delegate, C# compiler is still ensuring type safety
    tom.RegisterWithGetOlder(onGetOlder2);

    for(int i = 0; i < 10; i ++)
    {
        tom.Grow(10);
    }
}
```

### Generic Action<> and Func<>

- The generic Action<> delegate is defined in the System namespaces of the mscorlib.dll and System.Core.dll assemblies. You can use this generic delegate to “point to” a method that takes up to 16 arguments (that ought to be enough!) and returns **void**.

```cs

Action<string> actionTest1 = new Action<string>(onGetOlder2);

actionTest1("I am only five"); // oh no I am only five, go go to KFC!!


```

- The generic Func<> delegate can point to methods that (like Action<>) take up to 16 parameters and a custom return value. The final type parameter of Func<> is always the return value of the method.

```cs

static string Sum(int x, int y)
{
    return (x + y).ToString();
}

Func<int, int, string> FuncSum1 = new Func<int, int, string>(Sum);
```

### C# event Keyword

When the compiler processes the event keyword, you are automatically provided with registration and unregistration methods, as well as any necessary member variables for your delegate types. These delegate member variables are always declared private, and, therefore, they are not directly exposed from the object firing the event. To be sure,the event keyword can be used to simplify how a custom class sends out notifications to external objects.

- define a delegate type (or reuse an existing one) that will hold the list of methods to be called when the event is fired. Next, 
- declare an event (using the C# event keyword) in terms of the related delegate type.

```cs

// delegate with class
public class Person
{
    public int Age { set; get; }
    public string Name { set; get; }
    // 1. define a delegate type
    public delegate void GetOlderHandler(string megGetOlder);
    // 3. function to trigger invoke 
    public void Grow(int sleepYear)
    {
        Age += sleepYear;
        if (Age > 60)
        {
            // humanGetOlderHandlers(String.Format("I am {0}", Age));
            GoHigh(String.Format("I am {0}", Age));
        }
        else
        {
            // use null-conditional to check
            GoLow?.invoke(String.Format("I am {0}", Age));
        } 
    }
    // declare events
    public event GetOlderHandler GoHigh;
    public event GetOlderHandler GoLow;
}

public static void onGetOlder(string msg)
{
    System.Console.WriteLine("oh no {0}, go to Gym!!", msg);
}
public static void onGetOlder2(string msg)
{
    System.Console.WriteLine("oh no {0}, go go to KFC!!", msg);
}
static void Main(string[] args)
{

    Person tom = new Person { Age = 20, Name = "tom"};

    tom.GoHigh += new Person.GetOlderHandler(onGetOlder);
    tom.GoLow += onGetOlder2;

    for(int i = 0; i < 10; i ++)
    {
        tom.Grow(10);
    }

}
```

### Custom Event Arguments

- The first parameter of the underlying delegate is a System.Object represents a reference to the object that sent the event (Person above)

- The second parameter is a descendant of System.EventArgs represents information regarding the event at hand. The System.EventArgs base class represents an event that is not sending any custom information.

```cs 
public class EventArgs
{
    public static readonly EventArgs Empty;
    public EventArgs();
}
```




### Reference

[^1]: Christian N 2018, **Professional C# 7 and . NET Core 2. 0**, https://www.amazon.com/Pro-NET-Core-Andrew-Troelsen/dp/1484230175

