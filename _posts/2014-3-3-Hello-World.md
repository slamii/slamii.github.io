---
layout: post
title: Monads
---

# Why?
Functional programming languages are founded on the idea of function 
composition; having {% raw %}$f: A \rightarrow B${% endraw %} and 
{% raw %}$g: B \rightarrow C${% endraw %} we can create a composition
{% raw %}$g \circ f: A \rightarrow C$.{% endraw %}
Suppose now those functions have some additional, common
flavour e.g. they may not return a result or print something in addition
to returning it. In a strongly typed language we'd capture it by altering
the return type.

Let's stick to the example of possibly not returning a value. We'd use
a type constructor called `Maybe`, `M` for short, and our function 

# What?

# Where?