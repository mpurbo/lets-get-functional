---
title: "Monoid"
date: 2021-12-25T00:24:43+09:00
draft: true
weight: 40
---

## Monoid

A **Monoid** is a structure or pattern that appears in many programming tasks, often without us realizing it. We write code that uses Monoids all the time—even if we don’t call them that.
Take this simple example of summing all the numbers in a list:

```java
static Integer imperativeSum(List<Integer> numbers) {
    int sum = 0;
    for (int number : numbers) {
        sum = sum + number;
    }
    return sum;
}
```

Or this example, where we concatenate words to build a sentence:

```java
static String imperativeStringConcat(List<String> words) {
    String sentence = "";
    for (String word : words) {
        if (sentence.length() > 0) {
            // add space only after the first word
            sentence = sentence + " ";
        }
        sentence = sentence + word;
    }
    return sentence;
}
```

In both cases, we’re doing something very similar: **we're taking multiple values of the same type (numbers or words) and combining them into a single value** (a sum, or a sentence).

This pattern—of squashing or aggregating a collection of items into one—is at the heart of what a Monoid represents.

Let’s say we want to generalize this pattern so we can reuse the loop logic. We can see that the structure breaks down into two variable parts:

```
T output = initial_value [1]
for (T item : items) {
    main_logic [2]
}
return output
```

1. **The initial value**: a neutral or identity value—one that, when combined with any other value of the same type, leaves the other value unchanged. For example, `0` is the identity for addition (`x + 0 == x`), and an empty string `""` is the identity for string concatenation (`x + "" == x`).
2. **The main logic**: where the items are "squashed" together to produce the final output. This logic is essentially a binary operator—an operation that takes two values of the same type and returns another value of that type. In Java, this can be represented by the `BinaryOperator<T>` functional interface.

```java
public static <T> T reduce(T identity, BinaryOperator<T> operation, Collection<T> values) {
    T reduced = identity;
    for (T value : values) {
        reduced = operation.apply(reduced, value);
    }
    return reduced;
}
```

This generic function captures the essence of a Monoidal structure: it defines how we can combine multiple values of the same type using a consistent rule (operation) and a starting value that doesn’t interfere with the result.

For example:
- `Integer` with addition: `0` is the identity, and `+` is the operation that combines values.
- `String` with concatenation: `""` is the identity, and `+` (string join) is the operation.
- `List` with appending: an empty list is the identity, and the combining rule is simply adding elements to the end.

We can now rewrite our earlier examples using this `reduce` function.

Sum of integers:

```java
static public Integer functionalSum(List<Integer> numbers) {
    return reduce(0, (sum, number) -> sum + number, numbers);
}
```

Concatenating words into a sentence:

```java
static String functionalStringConcat(List<String> words) {
    return reduce(
            "",
            (sentence, word) -> {
                if (sentence.length() > 0) {
                    sentence = sentence + " ";
                }
                return sentence + word;
            },
            words);
}
```

With this approach, the boilerplate is gone, and what's left is the essence of the logic. Instead of specifying **how** to loop through items and accumulate results (as in imperative code), we declare **what** we want to do: combine values using a specific operation.

This is one of the key ideas in functional programming: **separating the structure of computation from the specific operation**—and Monoids are one of the most elegant abstractions for doing just that.

### More Practical Example: [LeetCode 896. Monotonic Array](https://leetcode.com/problems/monotonic-array/description/)

#### Problem Statement

An array is considered monotonic if it is either entirely non-increasing or non-decreasing. Given an integer array `nums`, the task is to determine if the array is monotonic

Examples:
```
Input: nums = [1, 2, 2, 3]
Output: true
```
```
Input: nums = [6, 5, 4, 4]
Output: true
```
```
Input: nums = [1, 3, 2]
Output: false
```

#### Kotlin Solution

Let’s now look at how we can apply the same Monoid structure to solve a real-world problem—in this case, [LeetCode 896: Monotonic Array](https://leetcode.com/problems/monotonic-array/description/).

Here’s the Kotlin solution:

```kotlin
fun isMonotonic(nums: IntArray): Boolean {
    val pairs = nums.toList().zipWithNext()
    return pairs.fold(true) { result, (current, next) -> result && current <= next } ||
           pairs.fold(true) { result, (current, next) -> result && current >= next }
}
```

Let’s break it down and see how this is an application of the Monoid pattern in disguise:
- We’re ultimately computing a **Boolean** result: is the array monotonic or not? So the Monoid here operates on `Boolean`. The identity value is `true`, and the combining operation is logical AND (`&&`). In other words, if we’re checking whether all comparisons hold, then `true` is the value that won’t change the outcome when combined—just like `0` for addition or `""` for string concatenation.
- To check whether an array is non-decreasing (or non-increasing), we need to compare **each element with the next one**. In Java, this would require writing a loop and managing indices manually, which adds a lot of boilerplate. In Kotlin, we can simply use zipWithNext() to create a list of adjacent pairs.
- Each of these pairs represents a “unit of comparison”—it’s the minimal chunk of information we need to make a decision (e.g., is `a <= b`?). So we treat each pair as a value that we want to “squash down” to a single `Boolean` result: either all comparisons hold, or one of them fails.
- Once we have these pairs, we apply `fold(true) { acc, pair -> ... }`. The `fold` function here plays the same role as the `reduce` function we discussed earlier: it takes an identity value and a binary operation and walks through the list, combining the results into a single value.

By expressing the solution this way, we avoid writing imperative loops, and we clearly separate the concerns: generating the data (pairs), transforming it (by mapping to a Boolean check), and combining it (with a Monoid-like structure). It’s declarative, readable, and captures the intent without getting lost in the mechanics of iteration.

#### Notes on Performance

In general, the functional approach is not always the most performant solution—especially in environments where allocations, temporary objects, and function calls add overhead. For instance, the same isMonotonic check can be written imperatively like this:

```kotlin
fun isMonotonic(nums: IntArray): Boolean {
    var increasing = true
    var decreasing = true

    for (i in 1 until nums.size) {
        if (nums[i] > nums[i - 1]) decreasing = false
        if (nums[i] < nums[i - 1]) increasing = false
        if (!increasing && !decreasing) return false
    }

    return true
}
```

This version short-circuits early when it knows the result is false, avoiding unnecessary work. It's also allocation-free, using plain variables and a simple loop.

However, while the functional version may introduce some overhead (e.g., temporary pairs, intermediate lists), it comes with **several advantages** that can make it more desirable in many cases:
- **Immutability**: There are no mutable variables to reason about or protect from race conditions in concurrent settings.
- **Side-effect freedom**: Each expression is self-contained, making it easier to isolate, test, and reuse.
- **Referential transparency**: Given the same input, the function always returns the same output, which enables caching, memoization, and parallelization opportunities.

These traits are especially valuable in larger systems where correctness, maintainability, and composability often outweigh micro-optimizations.

Kotlin’s standard library functions like `fold`, `map`, and `zipWithNext()` are optimized under the hood—thanks to features like function inlining and the JVM’s JIT compiler, their overhead often disappears entirely after a few runs of the program. That said, functional code isn't always the fastest option. For example, it may not short-circuit early like an imperative loop can. Still, the performance difference is often negligible in practice, and the benefits in clarity, immutability, and correctness can far outweigh the cost. The balanced approach is: _write functional code by default, and only optimize when necessary_.

### A More Formal Monoid Definition

A **Monoid** is a structure that consists of:
1. A **binary operation** that combines two values of the same type.
2. An **identity element** for that operation.

This concept can be expressed in Java as an `interface`:

```java
public interface Monoid<T> {
    T operate(T a, T b);
    T identity();
}
```

A **binary operation** here is simply a function that takes two values of type `T` and returns another value of type `T`. For a Monoid, this operation must satisfy the **associativity** property, meaning the grouping of operations doesn't affect the final result:

**Binary operation** is basically a function that operates on two values of the same type `T` and returns another value of `T`. The requirement for a Monoid is that this operation to be _associative_, that is:

```java
// For any values a, b, and c:
monoid.operate(a, monoid.operate(b, c))
    == monoid.operate(monoid.operate(a, b), c);
```

An example of associativity from basic arithmetic:

```
1 + (2 + 3) = (1 + 2) + 3
```

Addition and multiplication of integers are associative. In contrast, subtraction and division are **not**:

```
1 - (2 - 3) ≠ (1 - 2) - 3
```

The **identity element** is a special value that, when combined with any other value using the binary operation, leaves that value unchanged. That is:

```java
// For any value a:
monoid.operate(a, monoid.identity()) == a;
monoid.operate(monoid.identity(), a) == a;
```

For example:
- In integer addition, the identity is `0`, because `a + 0 == a`.
- In integer multiplication, the identity is `1`, because `a * 1 == a`.

The term _Monoid_ comes from **category theory**, a branch of mathematics that explores structure and composition at a very abstract level. While we've approached Monoids from a practical, programming-oriented perspective, it's worth noting that this idea has deep mathematical roots. Category theory offers a powerful and unifying way to think about patterns that show up across many areas of programming. If you're curious to explore further, you’ll find that Monoid is just the beginning of a much broader and beautifully consistent world. But for now, it's enough to know that this simple pattern—an identity and a way to combine values—is part of something much bigger.