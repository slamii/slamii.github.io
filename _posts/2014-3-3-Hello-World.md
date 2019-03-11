---
layout: post
title: Monads
---

# Why?
Functional programming languages are founded on the idea of function 
composition; having {% raw %}$f: A \rightarrow B${% endraw %} and 
{% raw %}$g: B \rightarrow C${% endraw %} we can create a composition
{% raw %}$g \circ f: A \rightarrow C${% endraw %}.
Suppose now those functions have some additional, common
flavour e.g. they may not return a result or print something in addition
to returning it.

In a strongly typed language we'd represent this additional behaviour by
a different type of the return value. Let's stick to the example of 
possibly not returning a value. We'd introduce a type constructor >>Maybe<<,
>>M<< for short, and our function would have signatures 
{% raw %}$f:A \rightarrow TB${% endraw %} and {% raw %}g:B \rightarrow TC${% endraw %}.
We still do want to compose them; conceptually the additional behaviour
should hinder us in no way, but now the types are incompatible. We are stuck.

Taking a step back, another thing besides composition of functions that
we possess is the identity function {% raw %}id: A \rightarrow A{% endraw %},
a trivial computation for any type >>A<<. Now, it turns out that there is a
mathematical structure built on top of these two called a >>category<<.

It belongs to Category Theory, a mathematical area that tries to describe
things in the most general way possible. Thus the notion of a category
{% raw %}$\mathcal{C}${% endraw %} is very simple: it's a collection of 
objects >>A<<, >>B<<, ..., together with a collection of >>morphisms<<, 
which you could think of as maps, with objects of {% raw %}$\mathcal{C}{% endraw %}
as domain and codomain, e.g. {% raw %}$f:A \rightarrow B${% endraw %}.
The morphisms of corresponding domains and codomains are composable in
the way you'd expect and each object has it's >>identity morphism<<
{% raw %}$id_A:A \rightarrow A${% endraw %} that acts trivially in
composition. For more details refer to .

# What?

# Where?
