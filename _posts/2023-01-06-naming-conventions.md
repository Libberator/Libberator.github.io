---
title: Naming Conventions
date: 2023-01-06 12:00:00 -0700
categories: [Blogging]
tags: [programming,opinion,C#]     # TAG names should always be lowercase
image: https://imgs.xkcd.com/comics/code_quality.png
---

> <center>"Any fool can write code that a computer can understand.<br>Good programmers write code that humans can understand."<br>— Martin Fowler</center>

## Listen Up

*We need to talk.*

No, I don't mean that in the "you're in trouble" sense, but more literally: we *need* to be able to communicate more clearly.
When something is written in text, it can be hard to determine the *\*intent\** behind the words or *where they're coming from*.
Words and phrases can have many different meanings to different people, often requiring surrounding context to decipher.
But what if you aren't given enough context or can't infer intent? (see: <a href="https://www.youtube.com/watch?v=sngRrkQayDA">Key & Peele - "When a Text Conversation Goes Very Wrong"</a>)

That's where **coding/naming conventions** come in. :heart_eyes: They're like the emojis of modern communication, able to pack a lot of info in such a small space. :100:

In the dev community, this can be a very sensitive topic - people have **strong** opinions about this.
But that's what this is: an opinion. A personal preference.
The most important thing is: whatever you choose, **be consistent** within your code base.

*Note: This article will mainly focus on the formatting or syntax aspect of C# programming, not so much on the "how to pick a name for a variable" part.*

## Why Should You Care?

1. Avoid conflicts. Did you forgot to specify `this.foo` and accidentally used `foo`, a different variable? 
With good naming conventions, you won't ever run into this mistake. You also won't ever need to type `this`, except for extension methods or passing an object as an argument.

2. Speed up development. If you're reading code, it will let you immediately know the scope or source of a variable.
If you're writing code and are blanking on what you called a certain private field, you can just type `_` and your IDE will list them all for you.
Reduces times spent scrolling.

3. Working with others (including future self). It's easier to parse a co-worker's code, and vice-versa.

4. Some ways of writing are just easier to read than oThErwaYSOFwRiTiNG.
The reader's focus should be on the code content rather than any distracting inconsistent formatting.

## Basic Naming Conventions

At a *minimum*, you should have 3 distinct ways to immediately identify whether a variable is:
- `public`
- `private`
- local (method parameter or created inside a method)

Microsoft has a detailed article covering this topic:<br>
<a href="https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions">https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions</a>

I follow the majority of them, with some differences.
Here is a non-exhaustive summary of my preferred coding conventions:

```csharp
namespace Libberator.Conventions; // "File-Scoped Namespaces" is a C# 10 feature

// Classes - always PascalCase
public class ClassName : ISomeInterface // interfaces start with an "I"
{
    // Fields
    private int _privateField; // private fields camelCase with leading underscore
    protected bool _protectedField; // protected fields camelCase with leading underscore
    public string PublicField; // public is PascalCase. When possible, use properties instead of public fields

    public const float PI = 3.14159f; // const is always SCREAMING_CAPS
    private const string MY_GREETING = "Hello"; // use $"{string} {interpolation}" when possible

    // Properties - always PascalCase
    public int RegularProperty { get => _privateField; protected set => _privateField = value; }
    public string AutoProperty { get; set; } // automatically creates its own backing field
    public bool GetOnlyProperty => _protectedField; // expression bodies to reduce number of lines

    // Methods - always PascalCase. Prefer being explicit with access modifiers like "private"
    private void PrivateMethod(int someParameter) // parameters always camelCase, no underscore
    {
        if (someParameter < _privateField) return; // use guard clauses to reduce indentations
        var someLocalVariable = 100; // local variables always camelCase, no underscore
    }
}
```

### Why I Format `protected` More Like `private` Than `public`

Microsoft's article states [emphasis mine]:
> In the following examples, any of the guidance pertaining to elements marked **public** is also applicable when working with **protected** and protected internal elements, *all of which are intended to be visible to external callers*.

The `protected` access modifier is intended to be used for inheritance. If you never inherit from the class that has `protected` stuff inside, they're essentially the same as just being `private`.
Other non-inherited classes don't have access to `protected` things. If it were `public`-facing, I'd give it PascalCase. They wrote "...visible to external callers", but a class inheriting from another is not an "external caller" in my opinion.
Just like real-world inheritance, the genes get passed down - they become *a part* of you.

Working with `protected` variables inside a class is very similar to working with `private` variables so that's why I make them camelCase with a leading underscore.
Is it important to know whether the variable originated in your class or in one of its parents? Not particularly. It's more important for me to know if any external class has the ability to access it directly.

### To `var` Or Not To `var`

I use `var` all the time. As much as possible. Most of the time, its type can be easily inferred by what's on the right-hand-side of the assignment operator.
If there's any chance of confusion for what type something is, the variable name should be descriptive enough to clear it up.
To be clear, this does **not** mean to ever use <a href="https://en.wikipedia.org/wiki/Hungarian_notation">Hungarian Notation</a>.

Similar to how I'm explicit with `private`, I also prefer to be explicit with how I write numbers. While `float num = 42;` is the same as `float num = 42f;` due to implicit conversion, `var num = 42;` is **not** the same as `var num = 42f;` —
one is an `int`, the other is a `float`. Therefore, it's a good habit to always write the `f` after your number when you're working with floats.
The main mistake I see new programmers make is when they use UnityEngine's `Random.Range(lowerBound, upperBound);`. The `int` version of that method has the upper bound as *exclusive*,
whereas the `float` version's upper bound is *inclusive*. Common source of off-by-1 errors.

Programming becomes simpler when you can just focus on the logic implementation and not be distracted by the types making it slower to write and harder to read.
Now don't get me wrong, I love statically-typed languages. Knowing what type a variable is ***is*** important,
but writing your code in a way that reads closer to plain English or pseudocode is much easier to follow along.
I don't think there's much to *gain* by purposefully avoiding `var`. It's a feature of the language for a reason.

### Bracket Indentation

There are two leading styles when it comes to where the brackets get placed.<br>
<a href="https://en.wikipedia.org/wiki/Indentation_style#Allman_style">Allman</a>:
```csharp
public void BracketOnNewLine()
{
    while (true)
    {
        // code
    }
}
```
<a href="https://en.wikipedia.org/wiki/Indentation_style#K&R_style">K&R</a>:
```csharp
public void BracketOnSameLine() {
    while (true) {
        // code
    }
}
```

This shouldn't be much of a surprise if you looked at the previous block of code, but I prefer **Allman** style.
I like seeing where the scope of one section of code begins and ends. Arguments against would say, "But that's so much vertical space!" and that's valid;
I, too, enjoy reading succint code and not having to scroll as much.

That being said, most methods shouldn't be so large to begin with that it would require scrolling.
Take advantage of ternary operators. If that's not easy to parse, extract it as an expression-bodied property with a human-readable name.
If there's only a single thing inside the brackets, omit the brackets entirely.
Whether the follow-up logic is on the same line as the condition or the next depends on the length or complexity of the line (see previous guard clause).

## General Philosophy & Tips

- Your code should read like prose; it should flow as easily as reading any sentence. Some might say "like poetry" because there is an art to it, but I've seen some cryptic and fragmented poetry in my day.

- Linq is great for readability and making your code succinct, but it has some performance considerations. Try to avoid it for things in the hot path.

- Let your IDE help you.
    - **Refactoring names**. Attention to detail is important. ~~Tpyo's happpen.~~ ~~Alot.~~ Know your IDE's shortcut for renaming. If you're not sure if you've spelled something correctly, look it up.
    - **Method Extraction (for self-documenting code)**. Reduce comment usage. If you have a comment explaining what something is doing (e.g. `// this gets all the active Players`),
you can instead *extract* that portion of code into its own clearly-named method or property (e.g. `GetActivePlayers()` or `ActivePlayers`).
This helps break down larger code sections into more digestible chunks. It will read more like pseudocode which in turn makes it easier to reason about the logic or for fixing bugs.
    - **Linter**. This is a tool that identifies and, if configured, automatically formats your code to adhere to coding standards and stylistic conventions.

Bonus (this one might just be a pet peeve): Wherever you share your code seeking feedback (StackOverflow, Reddit, Discord, etc.), use code block Markdown.
Add syntax highlighting if it's available and remove any unnecessary indentation. You're much more likely to get others to read your code if it's legible.

## Final Remarks

Every choice of character should be **purposeful**, reducing or entirely removing ambiguity.
It should serve as a way to densely pack and relay information. As succinct as an emoji. :wink:
(*Disclaimer: please don't code with emojis*)

I've discussed some of the basics on how to format or style your code for consistency and legibility.
I hope you are using a coding convention and are consistent with it.
And I hope your chosen convention at least differentiates between `public`, `private`, and local.
You'll thank your past self and others will appreciate you for using a coding convention.

> <center>"There are only three hard things when it comes to programming: naming things and off-by-1 errors."</center>

There is so much more on the topic of how to *choose* a name for a variable or method it deserves an entire post of its own.

Thanks for listening.
