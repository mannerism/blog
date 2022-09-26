---
layout: post
title: "Object Oriented Programming with swift"
date: 2022-09-03 4:48:49 +0900
categories: Ideas
---

## Introduction

We are gonna learn about the `Swift` way of handling variables and functions.

---

## Storing number to a variable

```swift
let a = 3 // 1. this is to store number 3 to a variable named a
let b = 4 // 2. store 5 to b
let sum = a + b // 3. if we add the two the sum will be 7
```

Storing numbers inside a variable is quite easy in swift. All you have to do is assign a number to a variable using `=` sign.

## Basic Function

```swift
func addThreeAndFour() { // 1. name a function
    let a = 3 // 2. variable 1
    let b = 4 // 3. variable 2

    let sum = a + b // 4. adding
    print(sum) // 5. printing
}
```

Function can be created with `func` keyword. Much like an english sentence, swift is known for highly readable code. Inside the function, I added variables we've discussed previously. With functions, we can recycle our codes easily.

Here is another example of a function:

```swift
func add(a: Int, b: Int) { // 1. with inputs
    let sum = a + b
    print(sum)
}
```

This functions can take two inputs: `a: Int` and `b: Int`. `:Int` means the input `a` can only take an integer. So if we put a non-integer value, the function won't work.

Here is another example of a function:

```swift
func add2(a: Int, b: Int) -> Int { // 1. with return
    let sum = a + b
    return sum
}
```

This time the function has another feature: `-> Int`. This means the function will return a value when it's called. Previous functions don't return any value. They just print the output using `print`. This time the function returns the calculated `sum`.

---

## Conclusion

We've covered:

1. setting a number to a variable.
1. A function that takes 0 input and returns nothing.
1. A function that takes inputs and returns nothing.
1. A function that takes inputs and returns something.
