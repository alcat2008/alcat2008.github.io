---
layout: post
title: React 设计思想
date: '2016-07-14 16:00:00 +0800'
tags: [React]
categories: react
---

# React 设计思想

## Transformation

The core premise for React is that UIs are simply a projection of data into a different form of data. The same input gives the same output. A simple `pure function`.

## Abstraction

You can't fit a complex UI in a single function though. It is important that UIs can be abstracted into `reusable pieces that don't leak their implementation details`. Such as calling one function from another.

## Composition

You also need to be able to build abstractions from the containers that `compose` other abstractions.

## State

There is actually a lot of state that is specific to an exact projection and not others.

We tend to prefer our data model to be immutable. We thread functions through that can update state as a single atom at the top.

## Memoization

Calling the same function over and over again is wasteful if we know that the function is pure. We can create a memoized version of a function that keeps track of the last argument and last result.

## Lists

To manage the state for each item in a list we can create a Map that holds the state for a particular item.

## Continuations

Unfortunately, since there are so many lists of lists all over the place in UIs, it becomes quite a lot of boilerplate to manage that explicitly.

We can move some of this boilerplate out of our critical business logic by deferring execution of a function. For example, by using "currying" (bind in JavaScript).

## State Map

We can move the logic of extracting and passing state to a low-level function that we reuse a lot.

## Memoization Map

Once we want to memoize multiple items in a list memoization becomes much harder. You have to figure out some complex caching algorithm that balances memory usage with frequency.

Luckily, UIs tend to be fairly stable in the same position. The same position in the tree gets the same value every time. This tree turns out to be a really useful strategy for memoization.

## Algebraic Effects
