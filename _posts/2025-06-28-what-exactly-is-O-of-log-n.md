---
layout: post
title: "Understanding Big O: Why Binary Search is O(log n) and Linear Search is O(n)"
date: 2025-06-28 10:00:00 +0900
categories: Algorithms
---

## Introduction

Ever wondered why some algorithms are lightning fast while others crawl? The secret lies in understanding Big O notation and how algorithms scale. Today we'll explore the difference between O(n) and O(log n) by diving deep into binary search - one of the most elegant algorithms in computer science.

---

## What is a Logarithm?

Before we jump into algorithms, let's understand what a logarithm actually is. A logarithm answers this question: "What power do I need to raise a number to get another number?"

```js
log₂(8) = 3, because 2³ = 8
log₂(16) = 4, because 2⁴ = 16
log₁₀(1000) = 3, because 10³ = 1000
```

Think of logarithms as the "inverse" of exponentiation. If 2⁵ = 32, then log₂(32) = 5. This concept is crucial for understanding why binary search is so efficient.

## Linear Search: The O(n) Approach

Imagine you're looking for a specific name in an unsorted phone book with 1000 entries. In the worst case, you might have to check every single name - all 1000 of them.

```python
def linear_search(arr, target):
    for i in range(len(arr)):  # might check all n elements
        if arr[i] == target:
            return i
    return -1
```

This is O(n) because:

- 1000 names = up to 1000 checks
- 2000 names = up to 2000 checks
- Double the data = double the time

## Binary Search: The O(log n) Magic

Now imagine that same phone book, but sorted alphabetically. You can use a much smarter approach: binary search.

```python
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = (left + right) // 2
        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1
    return -1
```

Let's trace through finding the number 21 in this sorted array:
`[1, 3, 5, 7, 9, 11, 13, 15, 17, 19, 21, 23, 25, 27, 29, 31]`

**Step 1:** Look at the middle (position 7, value 15)

- Is 21 = 15? No. Is 21 > 15? Yes.
- Eliminate left half: `[17, 19, 21, 23, 25, 27, 29, 31]`

**Step 2:** Look at middle of remaining elements (value 19)

- Is 21 = 19? No. Is 21 > 19? Yes.
- Eliminate left half: `[21, 23, 25, 27, 29, 31]`

**Step 3:** Look at middle again (value 25)

- Is 21 = 25? No. Is 21 < 25? Yes.
- Eliminate right half: `[21, 23]`

**Step 4:** Look at middle (value 21)

- Is 21 = 21? YES! Found it!

## Why Binary Search is O(log n)

Notice the pattern in our search:

- Start: 16 elements
- After step 1: 8 elements (16 ÷ 2)
- After step 2: 4 elements (8 ÷ 2)
- After step 3: 2 elements (4 ÷ 2)
- After step 4: Found it!

We took 4 steps total, and log₂(16) = 4 because 2⁴ = 16.

Each step cuts the problem in half. We're essentially asking: "How many times can I divide n by 2 until I get to 1?" That's exactly what log₂(n) represents!

**Mathematical proof:**

- After k steps, we have n/2ᵏ elements left
- We stop when n/2ᵏ = 1
- Solving: n = 2ᵏ
- Therefore: k = log₂(n)

## The Power of Logarithmic Growth

Here's where it gets impressive:

```js
1,000 elements: log₂(1000) ≈ 10 steps maximum
1,000,000 elements: log₂(1,000,000) ≈ 20 steps maximum
1,000,000,000 elements: log₂(1,000,000,000) ≈ 30 steps maximum
```

While linear search might need to check a billion items, binary search will never need more than about 30 comparisons. That's the difference between waiting seconds versus microseconds!

## The Key Insight

**O(n) - Linear Time:**

- Double the data = double the time
- Growth is proportional to input size

**O(log n) - Logarithmic Time:**

- Double the data = just one more step
- Growth slows dramatically as input increases

This is why understanding Big O notation is crucial for writing efficient code. The choice between O(n) and O(log n) can mean the difference between a responsive application and one that grinds to a halt.

---

## Conclusion

We've explored:

1. What logarithms represent and why they matter in algorithms
2. How linear search works and why it's O(n)
3. The step-by-step process of binary search
4. Mathematical proof of why binary search is O(log n)
5. Real-world implications of logarithmic vs linear growth

Next time you're faced with searching through data, remember: if you can sort it first, binary search will reward you with logarithmic performance that scales beautifully no matter how large your dataset grows.
