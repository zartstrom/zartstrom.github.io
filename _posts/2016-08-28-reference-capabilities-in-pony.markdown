---
layout: post
title:  "Reference Capabilites in Pony for everybody"
date:   2016-08-28 19:10:10 +0200
categories: pony
---

This post is about illustrating reference capabilities in the Pony programming language. The [Pony homepage](http://www.ponylang.org) states:
"Pony is an open-source, object-oriented, actor-model, __capabilities-secure__, high performance programming language."

Reference capabilities achieve this capabilities security and hence constitute an essential part of the Pony programming language.

Now, that said, what was the reason to introduce the concept of reference capabilities in the first place?
Reference capabilities are a means to

* safely share data concurrently
* avoid race conditions and
* check this at compile time.

This post should give you a better feeling what reference capabilities are, and after reading you should understand:

* In what sense reference capabilities form a type hierarchy
* What guarantees every reference capability gives

Prerequisites: Maybe playing around some minutes with Pony code from the [tutorial][pony_tutorial].


## References and Objects

Every reference to an object in Pony exists within an actor.
There can be one or more references pointing to the same object in memory and those references may be distributed arbitrarily over any number of actors.
This is how I think about references to objects in Pony:

![refs_to_objects_in_actors](/assets/images/refs_to_objects_in_actors.svg)

And, by the way, actors are objects too:

![actors_are_objects_too](/assets/images/actors_are_objects_too.svg)

## The Type Hierarchy of Reference Capabilities

A common error message in Pony looks like
`val is not a subtype of iso`.
So these reference capabilities form some kind of type hierarchy.

In Pony reference capabilities are about *denying* capabilities of an *alias* (= another reference to the same object). The capabilities in question are the *read* and *write* access to the object from the *alias*.

For example the __val__ reference capability guarantees that no *alias* has *write* access to the object, but allows *read* *aliases*.

Furthermore there is a distinction between *local* and *global* *aliases*. From the point of view of a variable in an actor, a *local* *alias* is another variable to the same object in the same actor. A *global* *alias* denotes a reference to the same object in another actor.

A __box__ reference capability guarantees that no *alias* in any other actor has *write* access to the object (*deny global write alias*). In the *local* actor there may be an *alias* with *write* access to the object.

Now lets think of the __box__ and __val__ reference capabilities as if

* they were classes
* and implement *deny*-functions.

Like this:

{% highlight pony %}
// non-compiling pseudo Pony code
class Box
    fun deny_global_write_alias()

class Val
    fun deny_global_write_alias()
    fun deny_local_write_alias()
{% endhighlight %}

The `class Box` has a `deny_global_write_alias` function, `class Val` has the same function and furthermore a `deny_local_write_alias` function. Now we remove duplicated code (the `deny_global_write_alias`-function) by making `class Val` a subclass of `class Box`:

{% highlight pony %}
// more pseudo Pony code - there is no inheritance in Pony
class Val extends Box
    fun deny_local_write_alias()
{% endhighlight %}

That is `class Val` is a subtype of `class Box`. By translating back to __val__ and __box__ we result with: __val__ is a subtype of __box__! This thought experiment indeed reflects the relationship between the two reference capabilities.

We can do the same exercise with every pair of reference capabilities. Here comes a table that displays every reference capability with its "*deny*-functions":

![hierarchy_method_table](/assets/images/hierarchy_method_table.svg)

By analogy to the above example we deduce the actual type hierarchy of reference capabilities:

![refcap_hierarchy_flat](/assets/images/refcap_hierarchy_flat.svg)


## The Complete Picture

A reference capability tells you about two things:

* The read and write access to the object where the reference points to.
* The guarantees about read and write access of aliases

This picture gives a complete overview over the features of every of the six reference capabilities.

![all refcaps](/assets/images/refcap_all.svg)

Further random notes:

* An actor is always opaque to other actors, therefore a variable with reference to an actor must have the __tag__ reference capability.
* The __trn__ capability is hardly ever used in code, it is safe to forget about __trn__ completely. In the current [standard library](https://github.com/ponylang/ponyc/tree/master/packages) there is only one usage, namely in the file [net/_test.pony](https://github.com/ponylang/ponyc/blob/master/packages/net/_test.pony).
* A box reference allows a local alias with write permission and a global alias with read permission. In fact only one of the two aliases (local write / global read) can exist for a box reference at a given moment. Can you imagine why? Pony takes care that this does not happen, i.e. look for what are sendable objects in the [Pony tutorial][pony_tutorial].

[pony_tutorial]: http://tutorial.ponylang.org
