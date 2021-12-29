---
title: "Functor"
date: 2021-12-25T00:24:43+09:00
draft: true
weight: 45
---

_Wrapper_ is a concept frequently used in object-oriented programming. We use wrappers for many reasons. Sometimes we use it to hide some specific implementations so that we can just interface with a simpler, more abstract form of structure. Sometimes we just want to extend the wrapped object with some operations, especially when the class of the wrapped object is `final` (cannot be extended).

Consider following example:

