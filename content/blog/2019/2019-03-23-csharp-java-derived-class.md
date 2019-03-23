+++
author = "Neo Ho"
categories = ["programming"]
tags = ["oop","c#","java","object","oriented","base","class","derived","sub","super"]
date = "2019-03-23"
description = "Difference between C# and Java about derived class."
featured = "derived-class-01.PNG"
featuredalt = "derived-class"
featuredpath = "date"
linktitle = ""
title = "Difference between C# and Java about derived class."
type = "post"

+++

## A simple derived class example in `Java`

I begin to develop Java recently. I had been asked a question about the behavior super/sub class. And, it's not quite the same as I thought. For example we have this Java program using derived class:

```java
public class Main {

    public static void main(String[] args) {
        One first = new Two();
        Two second = (Two) first;
        One third = (One) second;

        first.method();
        second.method();
        third.method();

        System.out.println(first.value);
        System.out.println(second.value);
        System.out.println(third.value);
        System.out.println(((Two) first).value);
        System.out.println(((One) second).value);
        System.out.println(((Two) third).value);
    }
}

class One {
    int value = 1;

    public void method() {
        System.out.println("one");
    }
}

class Two extends One {
    int value = 2;

    public void method() {
        System.out.println("two");
    }
}
```

### About the value

The simple part is about the `value`. The instance `first` is created from class `Two`. So, there are two `value`s in this instance. One is from class `Two` itself, another is derived from superclass/baseclass `One`.

If we use `first.value`, we will get the value `1` from type `One`. And, if we cast instance `first` to type `Two` then get it's value, we will get the value `2` from type `Two`. This code `System.out.println(((Two) first).value);` does this.

### About the method

Originally I think this code section:

```java
first.method();
second.method();
third.method();
```

Will get this result:

![derived-class-02.PNG](/img/2019/03/derived-class-02.PNG)

But actually the result is:

![derived-class-03.PNG](/img/2019/03/derived-class-03.PNG)

Why is the result different from what I think?

## Derived class in `C#`

In C#, there are three keywords about inheritance and polymorphism. There are `virtual`, `new` and `override`. So, we use two simple examples to demostrate the differences between `new` and `override`:

### Use the keyword `new`

```csharp
class Program
{
    static void Main(string[] args)
    {
        One first = new Two();
        Two second = (Two) first;
        One third = (One) second;

        first.Method();
        second.Method();
        third.Method();
        Console.WriteLine(first.Value);
        Console.WriteLine(second.Value);
        Console.WriteLine(third.Value);
        Console.WriteLine(((Two) first).Value);
        Console.WriteLine(((One) second).Value);
        Console.WriteLine(((Two) third).Value);
    }
}

class One
{
    public int Value = 1;

    public virtual void Method()
    {
        Console.WriteLine("one");
    }
}

class Two : One
{
    public int Value = 2;

    public new void Method()
    {
        Console.WriteLine("two");
    }
}
```

This example use keyword `new` to define the method in the derived class is independent of the method in the base class. Lets execute it, and we can get the result like:

![derived-class-04.PNG](/img/2019/03/derived-class-04.PNG)

Well, the result is the same as I thought.

### Use the keyword `override`:

We modify the code, let it looks like this:

```csharp
class One
{
    public int Value = 1;

    public virtual void Method()
    {
        Console.WriteLine("one");
    }
}

class Two : One
{
    public int Value = 2;

    public override void Method()
    {
        Console.WriteLine("two");
    }
}
```

Execute it and we can get the result:

![derived-class-05.PNG](/img/2019/03/derived-class-05.PNG)

Well, it looks like the result of the Java version.

### Why?

If we remove the C# keywords and let it looks like the Java code:

```csharp
class One
{
    public int Value = 1;

    public void Method()
    {
        Console.WriteLine("one");
    }
}

class Two : One
{
    public int Value = 2;

    public void Method()
    {
        Console.WriteLine("two");
    }
}
```

Well, we get some warnings, but it still can run and get result. It will be the same with the keyword `new` version.

![derived-class-04.PNG](/img/2019/03/derived-class-04.PNG)

In C#, if we don't use any keywords when we using derived class. The default behavior is `new`. It works like the `value` in the Java code. Two versions, from the `base class` and `derived class` will co-exist. It will depends on the `Type` at the time when the property or method be called.

And, Java's default (and only) behavior is `override`. If one member in the derived class is declared by `override`, it means this member will use the verison of derived class instead of the version of base class.

## Reference
1. [Versioning with the Override and New Keywords (C# Programming Guide)](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/classes-and-structs/versioning-with-the-override-and-new-keywords)
2. [Java Polymorphism](https://www.w3schools.com/java/java_polymorphism.asp)
3. [Polymorphism in Java with example](https://beginnersbook.com/2013/03/polymorphism-in-java/)