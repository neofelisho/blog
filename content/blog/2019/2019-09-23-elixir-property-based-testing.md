---
title: "Property Based Testing in Elixir"
date: 2019-09-23T15:13:34+07:00
tags: ["elixir", "property", "testing", "streamdata", "functional"]
categories: ["programming"]
author: "Neo Ho"
description: "Basic of property based testing and how to do it in Elixir by StreamData"
featured: "serval.jpg"
featuredalt: "serval"
featurepath: "date"
type: "post"
---

**Brief: This is a story about a rookie alchemist to learn property testing in Elixir. If you are also a alchemist and want to know what's property testing, welcome to share my experience. If you are not, in the first half of this story, there are some basic knowledges about property testing for your.**

> Maybe we should consider about property based testing.

In a retrospective meeting, a colleagues said this. So the story began...I knew nothing about it at that time.

![Leptailurus serval with many questions](/img/2019/09/serval.jpg)

> What's `property based testing`? 

First of all I need more information about it. Is it a cool stuff? Or just another buzzword?

This is my first impression after gathering information: it's a kind of random generative testing. It confuse me, what's the difference with what we already have?

Try to get a deeper understanding, there is a new term appear: `shrink`. It means that `property based testing` can tell us what's the minimal case to reproduce the error. Magic! Is there a rubber duck inside it?

![Rubber duck can debug code](/img/2019/09/rubber_duck.jpg)

> Maybe I know why I should use `property based testing`.

Sounds great. Before that, we need to write so many test cases, and consider about corner cases. Now what we need is write a property test, and ~~rubber duck will do all things for us~~ the test framework can generate many cases including corner cases for us. If there is anythink wrong, it will tell us what's the minimal case to reproduce error.

Recently I implemented a simple queue by Elixir. I think it should be a good sample to test by property.

{{< gist neofelisho aeaa9e475776fb0be2d2e08144a811b5>}}

![first property testing](/img/2019/09/property_001.png)

> Heaven is near...?

Everything is perfect. What I need in the future is 5~10 line of codes of testing, and ~~ruber duck will save the world~~ property testing will be responsible for everything else.

But, you know, as an engineer, it's too smooth to believe. In order not to be as a fool, I add a test for [87](https://www.urbandictionary.com/define.php?term=87) and it should be failed...because I am not an 87.

{{< gist neofelisho 32c9b8c08da715175ca58afeed55a364>}}

![fool testing](/img/2019/09/property_002.png)

Okay...it's a cruel world, and I am an 87 /sad.

![Cruel world](/img/2019/09/cruel_world.jpg)

I repeated the test over 10 times and it always passed. I finally got the test failure after I restrict the integer range to 86~88.

![fool testing again](/img/2019/09/property_003.png)

> So...is property testing useful?

Now we know that property testing is a kind of random generative testing with shrinking ability. It generates many random test cases including common edge cases such as nil value, empty list and so on. If it find anything wrong during the testing, it will try to shrink the erros, to find the minimal set which cause the problem. The property testing can test more and more different cases over time, maybe it could find something unexpected. But, if we already know you edge case, and the case is not common one (ex. the case of 87), we should add unit test for that case along with property testing. Then, the world will be beautiful.

![Beautiful world](/img/2019/09/beautiful_world.jpg)

> How to do porperty testing?

1. We need a property testing framework. In Elixir, we have [StreamData](https://hexdocs.pm/stream_data/StreamData.html).
2. Suitable scenario. Key idea is about `property` or `behavior`. If there are some functions with specific property, it's time to use `property based testing`.
3. Using patterns.

In Elixir, it's quite simple and easy to understand how to write a basic property testing like the example below.
```elixir
property "for what purpose" do
  check all item <- term() do
    # test here by the `item` taken from generator
    # ...
    assert actual == expected
  end
end
```

First line `property` means we want to use property testing (its `test` in unit testing). Focus on the second line. The `term()` is a generator provided by property testing framework (here is `StreamData`). StreamData provides many different kinds of generators, and `term()` generate all types of data. If the parameter of our method only accepts integer, we can use `integer()` instead here.

We can regard `check all` as a `for loop`. Literally what it does is like `for all items from the generator`. The we can test our logic by the item taken from the generator.

> What's patterns about property testing?

There are many articles about patterns. Here I only list the most commonly used in my opinion.

1. Round-tripping

    a.k.a. Encoder/Decoder. For example, we have `float to string` and `float from string` two methods. We can generate random floating numbers, convert them to string then convert back, it should keep the same one.

    ``` elixir
    assert term |> encoder() |> decoder() == term
    ```

2. Oracle model

    Here is not that [Oracle](https://www.oracle.com/index.html). It means the older and correctness model we already have. For some reason such as performance we need to implement a new one. They should have the same `properties`.

    ```elixir
    expected = oracle_code(term)
    actual = my_code(term)
    assert expected == actual
    ```

3. Smoke testing

    a.k.a Build verification testing. To ensure the functionality of our method working. For example, our method should always return `{:ok, result}` and `{:error, reason}`, then we can have a property testing like this:

    ```elixir
    {status, _result} = my_code(term)
    assert status in [:ok, :error]
    ```

    If it passed, it means our method works (no smoke or on-fired).

4. Stateful testing

    If we have some stateful functions in our system, such as `Memcached`, `Redis`, `ets` or `GenServer` in Elixir. We can use property testing on it like below example:

    {{<gist neofelisho c6c30cc3036f71bcd16108c13431eeaf>}}

> References

These links help me a lot. I hope they also help you.

1. [Introduction to Property Based Testing](https://medium.com/criteo-labs/introduction-to-property-based-testing-f5236229d237)
2. [Property-based or generative testing – insights from European Testing Conference](https://lisacrispin.com/2019/04/20/property-based-or-generative-testing-insights-from-european-testing-conference/)
3. [Property-based Testing is a Mindset - Andrea Leopardi - ElixirConf EU 2018](https://www.youtube.com/watch?v=p84DMv8TQuo)
4. [Property-based Testing Patterns](https://blog.ssanj.net/posts/2016-06-26-property-based-testing-patterns.html)