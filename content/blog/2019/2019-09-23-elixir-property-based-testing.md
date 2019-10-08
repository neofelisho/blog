---
title: "Property Based Testing in Elixir"
date: 2019-09-23T15:13:34+07:00
tags: ["elixir", "property", "testing", "streamdata", "functional"]
categories: ["programming"]
author: "Neo Ho"
description: "StreamData and property based testing from zero"
featured: "rubber_duck.jpg"
featuredalt:
featurepath: "date"
type: "post"
---

**Brief: This is a story about a rookie alchemist to learn property testing in Elixir. If you are also a alchemist and want to know what's property testing, welcome to share my experience. If you are not, in the first half of this story, there are some basic knowledges about property testing for your.**

> Maybe we should consider about property based testing.

In a retrospective meeting, a colleagues said this. So the story began...when I knew nothing about it.

![Leptailurus serval with many questions](https://truth.bahamut.com.tw/s01/201710/19ab338ba72a8c3624b23ffe92cbaf69.JPG)
(/img/2019/09/serval.jpg)

> What's `property based testing`? 

Is it a cool stuff or just another buzzword?

First impression is, it's a kind of random generative testing. It confuse me, what's the difference with what we already have?

Try to get a deeper understanding, there is a new term appear: `shrink`. It means that `property based testing` can tell us what's the minimal case to reproduce the error. Magic! Is there a rubber duck inside it?

![Rubber duck can debug code](https://i.ytimg.com/vi/9cbTzMj3bFM/maxresdefault.jpg)
(/img/2019/09/rubber_duck.jpg)

> Maybe I know why I should use `property based testing`.

Sounds great. Before that, we need to write so many test cases, and consider about corner cases. Now what we need is write a property test, and ~~rubber duck will do all things for us~~ the test framework can generate many cases including corner cases for us. If there is anythink wrong, it will tell us what's the minimal case to reproduce error.

> Heaven is near...?

![Cruel world](http://blog-imgs-74.fc2.com/t/a/r/taruannie/849f021a.jpg)
(/img/2019/09/cruel_world.jpg)

We all know it's a cruel world. But if we know how to use it well, the world will be beautiful.

![Beautiful world](http://phoenix-wind.com/common/img/OGP/word/shingeki_mikasa_03.jpg)
(/img/2019/09/beautiful_world.jpg)

I think the key idea is `property`, or `behavior`. If there are some functions with specific property, it's time to use `property based testing`. It doesn't mean that we don't write unit test anymore. Otherwise, we should combine both advantages.


## `Property Based Testing` in Elixir

We need a good tool to do property based testing. This tool should support `generative` and `shrinkable`.
In Elixir, we have library `StreamData` which supports these functionality.

MyQueue example

property testing using circular code
=> term |> enqueue |> dequeue == term

property testing using oracle model
=> my_queue == :erlang.queue



## Patterns

1. Circular Code
    => term |> encoder |> decoder == term
2. Oracle Model
    => my_code() == oracle_code()
    => new, refactor, improved implementation == order, less performant implementation
3. Smoke Tests
    => API: 200, 201, 400, 404
    => order calculator should get the result in [A, B]

## Use Property Based Testing instead of Unit Testing?

NO. Some already know cornor case we should use unit test to make sure it's solid.
=> Should combine both.
=> Although property testing can generate and shrink, but it doesn't mean that it can always gen corner case.
=> list_a = [1,2,3] list_b = [4,5,6] enqueue a, b will get a list contains a, or b, 
   but it should also contain [3,4] this case

## Stateful Testing

If some function in our system is stateful, ex. Redis or GenServer. It's good to use property testing.
Use a map to keep the expected data (a.k.a. control group), then assert the actual one (a.k.a. test group) should always equal.

## <a name="references" />References

1. [Introduction to Property Based Testing](https://medium.com/criteo-labs/introduction-to-property-based-testing-f5236229d237)
2. [Property-based or generative testing – insights from European Testing Conference](https://lisacrispin.com/2019/04/20/property-based-or-generative-testing-insights-from-european-testing-conference/)
3. [Property-based Testing is a Mindset - Andrea Leopardi - ElixirConf EU 2018](https://www.youtube.com/watch?v=p84DMv8TQuo)