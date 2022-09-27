---
layout: post
title: "Self, self, .self. what the self? in Swift"
date: 2022-09-27 4:48:49 +0900
categories: study
---

## Intro

OK. I've been programming `Swift` for quite some time now about 4 years and didn't really bother too much about `self`. Most of the my everyday coding life revolves around using `self.<some variable>`. Maybe a few `.self` when registering a cell class in `UICollectionView` for in `UITableView` like this:

```swift
collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "Cell")
```

But I don't yet 100% understand what are `.self`, `Self`, `.Self`, or even `self`. This post is about these `selves` in swift.

## self with lowercase `s`

This is a common lower-case `self` we often use inside `struct` or `class`. It means the variable is defined inside the `struct` or `class` at an outermost layer. Let's see an example:

```swift
class Fruit {
    let name: String   // #1
    let color: String
    init(name: String, color: String) {
        self.name = name // #2
        self.color = color
    }
}
```

1. variable `name` is defined at the outermost layer of the class `Fruit`
2. inside `init(name: String, color: String)` function, `self.name` on the left side represents `name` defined in `#1` and `name` on the right side represents one of the argument variables in `init` function.

> In Swift, the **self keyword refers to the current object inside the type that implements the object.** [Definition Reference](https://www.codingem.com/self-in-swift/)

## Self with capital `S`

> The **Self keyword is used in protocols to represent the type that is going to conform to the protocol.** [Definition Reference](https://www.codingem.com/self-in-swift/)

Let's contemplate on this with an example:

```swift
protocol Callable {
    func greet(_ other: Self)
}

struct Person : Callable {
    func greet(_ other: Person) {
        print("Hi, I am  \(other)")
    }
}
```

In swift, we define `protocol` so that other `types` can conform to it. For instance, if we have a `Person` struct that conforms to `Callable` protocol, the `Person` struct must implement `greet(_ other: Self)` function inside its body. That's what it means to _conform to a protocol_.

So, when we conform to a protocol in a newly defined type, in this case `Person` struct, and if we want to reference the conforming type inside the function call `greet` we have to use `Self` keyword.

Let's look at it in another angle one more time.

When we define a `protocol` in `Swift`, we do not care about the types, like `classes` or `structs`, that will conform to this protocol. We shouldn't. That's the whole point of having a `protocol`, **a blueprint of methods, properties, and other requirements that suit a particular task or piece of functionality.**. We try to define generic functionalities in the protocol so that we can generalize and categorize what it means to use this specific type of protocol. Then when it is conformed by a class or a struct, detailed business logics are implemented inside the conforming class or struct.

In this case, the protocol `Callable` wants its conforming type to implement `func greet()`. And inside the `greet` function, it wants the conforming type to use itself as an argument, shown as `(_ other: Self)`. Like I mentioned above, protocol doesn't give a shit about its conforming type. So it needs a generic way to refer to the conforming type. That's when `Self` comes in. Using `Self` inside the protocol, we can make sure that when a type conforms to the protocol, `Self` will 'transform' to the conforming type, in this case `Person` type, and this will allow us to use `Person` as the transformed version of the function: `greet(_ other: Person)`.
