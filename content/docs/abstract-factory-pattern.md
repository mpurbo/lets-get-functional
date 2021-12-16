---
title: "Abstract Factory Pattern"
date: 2021-12-16T22:02:43+09:00
draft: true
---
## Abstract Factory Pattern

The abstract factory is one of the most widely used pattern, almost certain to be found in any large object-oriented projects. As the name suggests, it gives us a "factory" that lets us create a system of related objects in their abstract form. Here's an illustration on how it works:

[insert illustration-1]

In the above illustration, the factory produces notebook computers that look exactly the same on the outside, but the components varies depending on the price. The low-end model uses cheaper and slower processor, less memory, etc. The point of this abstract factory pattern is that:

- We use the output of the factory in exactly the same way.
- The product of the factory may do things differently, and produce different result, even that we operate them in exactly the same way. 
- The reason why the product behaves differently is because internally they are different set of components ("implemented" differently to follow different requirements, in this case material cost).
- The user of the product doesn't need to learn or do things differently in order to take use different models.

The advantage of using this pattern are mostly extensibility: 

- It is easier to upgrade or replace components being produced by the factory without having to update the client of the factory as well.
- It is easier to distribute the implementation effort to multiple independent teams. 

This really is a very powerful abstraction that gets used and, unfortunately, abused a lot. In many instances, this pattern causes a bloated overdone system structure. It can be good when used in moderation for some appropriate problems, but this approach tend to encourage us to have "fun" constructing excessive class structure.

### Example Problem: Ecosystem With Animals

To see how the abstract factory pattern being applied to model a relatively simple problem, let's devise an example involving animals:

[insert illustration-2]

- In this system we have three types of animal: `Dog`, `Bird`, and `Fish`. Each of them have different way to move around.
- These three animals move in different mediums: `Dog` runs on `Earth`, `Bird` flies in the `Air`, and `Fish` swims in the `Water`. The mediums might have some effect on animal's movement.
- Let's say that these animals live in an `Ecosystem` where a natural disaster may occur.
- When there is a natural disaster, let's say that the animals would move away from the place where the disaster happens, until they are at a safe distance. The meaning of "safe" is of course relative to each animal. A distance that is safe for a `Bird` might not be safe for a `Dog`.

Knowing about this kind of problem, a seasoned object-oriented programmer would jump in and eagerly imagine a structure of tens of classes to model the system, all in the name of future extensibility (which most of the time, would never happened because the system got obsolete). 

Before going deeper into how usually we approach this problem in object-oriented imperative way, let's specify some more details to simplify the meaning of animals' "movement":

- Ecosystem where the animals live is represented as a 2-dimensional cartesian plane.
- Animals move when their position (`x` and `y` coordinate) changes. 
- Animals move in units of movement. Different animal move in different unit (that represent speed).
- The unit of movement of each animal is influenced by the medium in which the animal moves.
- The position of the disaster is constant.  

### Ecosystem With Animals: Object-oriented Imperative Approach


<!--stackedit_data:
eyJoaXN0b3J5IjpbOTY3NjA1MDM0XX0=
-->
