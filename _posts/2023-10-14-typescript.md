---
layout: post
title: "Thinking about Typescript"
date: 2023-10-14 00:00:00 +0000
cuid: clnpj0a7g00000ajvg39d0ytz
slug: thinking-about-typescript
tags: typescript
categories: General
published: true
---

Typescript has been popular in the development world for a few years now, introducing a typing system to Javascript development being the primary objective. In this regard, it has been wildly successful and has made (For some developers) a massive improvement over the Javascript development process. Recently, some debate has arisen in the development communities around Typescript vs Javascript and the benefits it brings to the table. While I am far from an expert on Typescript I do work in it daily and have a few points I would like to express around the debate.

## Compiled vs Interpreted

Compiled languages do bring some benefits to the development lifecycle, even though they also bring drawbacks. The first benefit is that code is syntactically checked at compile time, which catches a lot of common mistakes developers make (Bad assignments, unused or unknown variables, etc). The drawback is compiling does take time and over a day can waste a lot of time while waiting for compiles to complete. Another point is the amount of steps going from writing the code to seeing the results, which in a compiled solution will require additional steps to complete (EG: Writing the code, compiling the solution, and testing the changes as opposed to writing the code and immediately seeing the changes). The final point, which is directly applied to Typescript vs Javascript is that the compiled results are often what you will be debugging in most scenarios and tracing back from the compiled Javascript to the original Typescript requires extra steps to achieve correctly.

## What about Types?

Typescript, unsurprisingly, brings types to the Javascript code. Javascript does not care about types, so Types are only enforced at compile time (unlike strongly typed languages which also fail at runtime if the types are invalid). Take the following C++ code for example.

```cpp
int sum(int a, int b) {
  return a + b;
}

// Fails at compile time
auto c = sum('2', 1);
// Fails at runtime if we do not provide a valid integer
int d;
cin >> d;
auto e = sum(atoi(d), 1);
```

If we try and compile the above code it will fail because the string '2' cannot be used as an integer. If we fix that it will also fail at runtime if we do not provide a valid integer for the variable d when it prompts for one.

Javascript, on the other hand, does not make such strict checks. Types in Javascript are still important, and every variable has an underlying type. It is just that the type is determined when the code is executed. Consider the below example

```javascript
function sum(a, b) {
  return a + b;
}

// This works at runtime
const c = sum('2', 1);
var d = readline();
// This works at runtime
const e = sum(d, 2);
```

Unlike the C++ example, this code will not fail at runtime but might do some unexpected results (For example the sum of '2' and 1 will be "21" since string concatenation takes precedence over arithmetic). The input value of d can be anything (String or number) and the result of e could be a string, number, or something else entirely. In the given code it will always do concatenation since readline always returns a string, even if a number is inputted. This type of mistake is so common in Javascript development since it is entirely up to the author to ensure that the types are what they want them to be.

Typescript at least brings some sanity to the code above by helping determine the "valid" or "sane" use case. The following Typescript code will help prevent unintended usage

```typescript
function sum(a: number, b: number): number {
  return a + b;
}

// This will be flagged at compile time now
const c = sum('2', 1);
var d = readline();
// This will require a cast now, since readline does not by default return a number
const e = sum(parseInt(d), 2); 
```

Suddenly, the code needs to follow the intention, whereas, in the Javascript example, it is entirely up to the author to remember to put in the checks that ensure the correct values are passed in.

## Is it worth it?

This question is the one that I think most developers will ultimately get stuck with. There is also a cost to any choice, and it is up to the team/developer to accept that cost. For example, using Typescript introduces more tooling and compile time costs for potentially a safer code base and clear intentions about what is expected for inputs and outputs. I have seen well-written Javascript that makes the need for Typescript unnecessary. I have also seen Javascript that works by luck and is hostile to everyone except the original author to read or debug.

I think that a mature codebase with little development or changes could transition away from Typescript to pure Javascript. Practically I have not seen many cases that would apply.

I feel personally that the benefits of removing ambiguity from the code and helping avoid simple mistakes before the code goes live is a cost I am willing to accept.