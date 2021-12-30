---
title: "Monoid"
date: 2021-12-25T00:24:43+09:00
draft: true
weight: 40
---

## Monoid

Monoid is a structure or pattern that can be found in many programming tasks. We write Monoids all the time without being aware of them. Consider the following simple function that simply sum all the numbers in a list:

```java
static Integer imperativeSum(List<Integer> numbers) {
    int sum = 0;
    for (int number : numbers) {
        sum = sum + number;
    }
    return sum;
}
```
In other time we concatenate words to build a sentence like:

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
In both cases, we're doing a very similar thing: creating one thing (a sum, or a sentence) out of multiple things of the same type (numbers or words). Let's say that we want to generalize this function so that we can reuse this loop logic. We can identify that there are two places that are "different":

```
T output = initial_value [1]
for (T item : items) {
    main_logic [2]
}
return output
```

1. Initial value.
2. The main logic where the items are operated to build the output. This main logic is basically a binary operator (operate on two values of the same type and producing something of the same type). In Java it can be represented by `BinaryOperator<T>` functional interface.

```java
public static <T> T reduce(T identity, BinaryOperator<T> operation, Collection<T> values) {
    T reduced = identity;
    for (T value : values) {
        reduced = operation.apply(reduced, value);
    }
    return reduced;
}
```
This generic function is basically showing what we can do with a Monoid structure. We can operate it with another Monoid of the same type, to produce another Monoid of the same type. `Integer` _addition_ is a Monoid. `String` and `List` are some examples of another Monoidal structure. 

As a Monoid, the imperative code for summing all `Integer` numbers can be expressed as follows:

```java
static public Integer functionalSum(List<Integer> numbers) {
    return reduce(0, (sum, number) -> sum + number, numbers);
}
```
The code for concatenating words to build a sentence becomes:

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
As can be seen here, the boilerplate code is gone, and what left is the actual logic that declares what we want to do. This is the essence of functional programming: we don't specify _how_ to do things (loop as in imperative code), but we specify only _what_ we want to do (summing, building sentence). 

### Formal Monoid Definition

Monoid is defined as a structure that has:
1. An associative binary operation.
2. An identity.

This definition can be expressed as a Java `interface` as follows:

```java
public interface Monoid<T> {
    T operate(T a, T b);
    T identity();
}
```

**Binary operation** is basically a function that operates on two values of the same type `T` and returns another value of `T`. The requirement for a Monoid is that this operation to be _associative_, that is:

```java
// assuming m is an instance of a Monoid, following statement evaluates to true
m.operate(value1, m.operate(value2, value3) == m.operate(m.operate(value1, value2), value3);
```

An example of associativity in simple mathematics is:

```
1 + (2 + 3) = (1 + 2) + 3
```

Integer addition and multiplication are associative, however integer subtraction and division are not, for example:

```
1 - (2 - 3) ≠ (1 - 2) - 3
```

**Identity** is a value of type `T` such that:

```java
// assuming m is an instance of a Monoid, following statements evaluates to true
m.operate(value, identity) == value;
m.operate(identity, value) == value;
```

In integer addition, `0` is the identity, as `1` is for integer multiplication:

```
2 + 0 = 2
2 x 1 = 2
```

### Fold and Reduce

Fold and reduce is one of the most used practical application of Monoid. The idea of _folding_ is essentially to produce a value out of multiple values (in a collection). It's a generalization of the "aggregation" or "summarization" process. Some languages use the term `reduce` as a synonym to `fold`, some are not. We will see how `reduce` could be different from (or a special case of) `fold` in some later examples. 

[illustration-1]

Let's revisit the imperative code for summing up some values in a list in Java:

```java
static Integer imperativeSum(List<Integer> numbers) {
    int sum = 0;
    for (int number : numbers) {
        sum = sum + number;
    }
    return sum;
}
```
This code shows that we're producing a value (`sum`) out of multiple numbers in a list (`numbers`). We can say that we're _folding_ or _reducing_ these multiple numbers into one number representing their sum:

```java
static Integer functionalSumReduce(List<Integer> numbers) {
    return numbers
            .stream()
            .reduce(0, (sum, number) -> sum + number);
}
```
There are a lot of noises when writing functional code in Java. Consider following equivalent code in Kotlin:

```kotlin
fun functionalSumFold(numbers: List<Int>) = numbers.fold(0) { sum, number -> sum + number }
```

### Fold and Reduce in Different Languages

