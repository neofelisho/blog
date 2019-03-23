+++
author = "Neo Ho"
categories = ["programming"]
tags = ["c#","async","await","thread","task","deadlock"]
date = "2019-03-18"
description = "C# async/await and threads deadlock"
featured = ""
featuredalt = ""
featuredpath = "date"
linktitle = ""
title = "C# async/await and threads deadlock"
type = "post"
+++

## A thread-safe async/await program

First is a very simple program to simulate requesting I/O in the main function.

```csharp
private static async Task Main(string[] args)
{
    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    await GetIoTaskAsync();
    await GetIoTaskAsync();
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}

private static async Task GetIoTaskAsync()
{
    await Task.Run(() =>
    {
        Thread.Sleep(500);
        Console.WriteLine($"Waiting for I/O, {Thread.CurrentThread.ManagedThreadId}");
    });
}
```

Execute this program and we can get the result like:

![001.PNG](/img/2019/03/001.PNG)

## A unsafe program occurs thread deadlock

Sometimes we can see something like below because someone want to invoke an async method in a sync function:

```csharp
private static void Main(string[] args)
{
    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    GetIoTaskAsync().Wait();
    GetIoTaskAsync().Wait();
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}
```

It still works, and have the same result with the previous one.
No...it's a illusion. 

If we restrict the number of max threads, and request more I/O like below.

```csharp
private static async Task Main(string[] args)
{
    // The number of maximum threads must larger than the number of minimum threads (can't equal).
    // So we have to set the minimum first, than set the maximum and make sure max > min.
    ThreadPool.SetMinThreads(1, 0);
    ThreadPool.SetMaxThreads(2, 2);
    ThreadPool.GetMaxThreads(out var workerThreads, out var completionPortThreads);
    Console.WriteLine($"{workerThreads}:{completionPortThreads}");

    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    Parallel.For(0, 10, async _ => { await GetIoTaskAsync(); });
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}
```

Well, we can get an acceptable result like this:

![002.PNG](/img/2019/03/002.PNG)

The main function goes to the `End` but we still have some tasks uncomplete. 
If we don't terminate the main function, the remaining tasks will be executed and completed by default [TaskScheduler](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskscheduler).

Surely, we can use `Task.WhenAll` to wait all jobs done and then show the `End` message.

```csharp
private static async Task Main(string[] args)
{
    // The number of maximum threads must larger than the number of minimum threads (can't equal).
    // So we have to set the minimum first, than set the maximum and make sure max > min.
    ThreadPool.SetMinThreads(1, 0);
    ThreadPool.SetMaxThreads(2, 2);
    ThreadPool.GetMaxThreads(out var workerThreads, out var completionPortThreads);
    Console.WriteLine($"{workerThreads}:{completionPortThreads}");

    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    var tasks = new List<Task>();
    for (var i = 0; i < 10; i++)
    {
        tasks.Add(GetTaskAsync());
    }
    await Task.WhenAll(tasks);
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}
```

Yes, we get a better result than the previous one.

![003.PNG](/img/2019/03/003.PNG)

But now, if someone want to invoke this async I/O method 10 times in a sync function just like before. 

```csharp
private static void Main(string[] args)
{
    // The number of maximum threads must larger than the number of minimum threads (can't equal).
    // So we have to set the minimum first, than set the maximum and make sure max > min.
    ThreadPool.SetMinThreads(1, 0);
    ThreadPool.SetMaxThreads(2, 2);
    ThreadPool.GetMaxThreads(out var workerThreads, out var completionPortThreads);
    Console.WriteLine($"{workerThreads}:{completionPortThreads}");

    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    Parallel.For(0, 10, _ => { GetTaskAsync().Wait(); });
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}
```

What will happen? It occurs thread deadlock!

![004.PNG](/img/2019/03/004.PNG)

We can not complete all tasks by the two worker threads we have.

## How to avoid thread deadlock

An acceptable way is to wrap all tasks in another method, and use `Task.WaitAll` to execute them synchronously.

```csharp
private static void Main(string[] args)
{
    // The number of maximum threads must larger than the number of minimum threads (can't equal).
    // So we have to set the minimum first, than set the maximum and make sure max > min.
    ThreadPool.SetMinThreads(1, 0);
    ThreadPool.SetMaxThreads(2, 2);
    ThreadPool.GetMaxThreads(out var workerThreads, out var completionPortThreads);
    Console.WriteLine($"{workerThreads}:{completionPortThreads}");

    Console.WriteLine($"Start, {Thread.CurrentThread.ManagedThreadId}");
    RunAllTask();
    Console.WriteLine($"End, {Thread.CurrentThread.ManagedThreadId}");
    Console.Read();
}

private static void RunAllTask()
{
    var tasks = new List<Task>();
    for (var i = 0; i < 10; i++)
    {
        tasks.Add(GetTaskAsync());
    }

    Task.WaitAll(tasks.ToArray());
}
```

## TL;DR

The better way is followed these principles:

1. Only invoke async method in async function. Surely, invoke sync method in async function is okay lol.
2. Don't block async method in sync function. It means, don't use `.Result()` or `.Wait()`.
3. Don't use `lock` in async method. If lock is necessary, use `SemephoreSlim` instead of `lock`.

## Reference

1. [Asynchronous programming](https://docs.microsoft.com/en-us/dotnet/csharp/async)
2. [Async in depth](https://docs.microsoft.com/en-us/dotnet/standard/async-in-depth)
3. [ThreadPool.SetMaxThreads(Int32, Int32) Method](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.threadpool.setmaxthreads)
4. [Constraining Concurrent Threads in C#](https://markheath.net/post/constraining-concurrent-threads-csharp)