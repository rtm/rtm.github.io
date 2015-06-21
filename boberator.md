---
title: The JavaScript pick operator
layout: default
---

# JavaScript "Pick"  Operator

This document proposes a "pick" operator for JavaScript, also known as the "boberator".
It is represented by the sharp sign.
The pick operator can be considered a generalization of deconstruction.

This proposal falls into the category of "syntactic sugar" for JS.
It is intended to allow commonly-used idioms related to object property manipulation to be written more succinctly.
It may also allow engines to optimize better.
It also provides an elegant solution to the so-called existential operator problem.

## Motivation

Some meaningful portion of JS written today is concerned with manipulating and extracting properties from objects.

The representative case is to create an object with two properties from another object:

    { p1: o.p1, p2: o.p2 }

In other words, creating a new object containing properties `p1` and `p2` drawn from `o`.
This is generally referred to as "picking".
But the above has several drawbacks:

 1. `o`, `p1` and `p2` all must be repeated.

 1. The construct is wordy and prone to typos.

 1. The JS interpreter may have to retrieve `o` twice, a potential efficiency concern.

 1. There is no easy way to short-circuit the property access if `o` is not an object.

### Underscore's `_.pick`

Underscore has acknowledged the importance of property arithmetic with APIs such as `_.pick`, `_.omit`, and `_.pluck`.

#### `_.pick`

The `_.pick` API, called as:

    _.pick(o, 'p1, 'p2')

Drawbacks here include:

 1. We have to inconviently quote each property name.

 1. The object `o` is accessed anew each time through a loop across properties, a potential inefficiency.

 1. There is no way to rename properties as they are picked.

 1. There is no way to specify a default for a missing property. We'd have to do something like `_.defaults(_.pick(o, 'p1'), { p1: 99 })`.

 1. It is impossible to pick more deeply within an object. Saying `_.pick(o, 'p1', 'p2.p22')` is not supported.

`_.pick` does implement short-circuiting; it returns `{}` if `o` is not an object.
That is a feature that currently we would have to write as:

    o && typeof o === 'object' ? { p1: o.p1, p2: o.p2 } : {}

`_.pick` also acknoledges the usefulness of selecting properties to be picked via a predicate:

    _.pick(o, p => /^p/.test(p))

In plain old JavaScript, we'd have to do something like

    Object.keys(o) . filter(p => /^p/.test(p)) . reduce((result, p) => Object.defineProperty(result, p, { value: o[p]), {})

#### `_.pluck`

"Plucking" is a sort of map of picks. We can write

    _.pluck([ o1, o2 ], 'p')

which returns `[ { p: o1.p }, { p: o2.p }]

and is essentially equivalent to

    [ o1, o2 ].map(o => _.pick(o, 'p')

In other words, it provides a version of `_.pick` which acts as a callback for iterators.

#### `_.omit`

Omitting is a sort of reverse picking--specifying properties **not to pick**.
Underscore's `_.omit` API also supports specifying properties to omit via a predicate.


#### Other Underscore property manipulators

Underscore has also seen fit to provide the following property-related utilities:

 * `_.matcher`: ensure an object contains designated properties

 * `_.property`: create a function to retrieve a specified property from some object

* `_.propertyOf` create a function to retrieve some property from a specified object

### Other libraries

Other libraries have also acknolwedged the importance of pick-like operations.

 * Ember has `Ember.Object#getProperties.


### Other current approaches

### Picking with destructuring assignment

Actually, ES6 does support picking in the form of destructuring assignment.
This capability does address most of the drawbacks mentioned above for `_.pick`.
We write

    { p1, p2 } = o

The fundamental limitation of this feature is that **it is limited to picking into variables**.
If I want to create an object with properties `p1` and `p2` from another object, to use this feature I must write

    var { p1, p2 } = o;
    var pickedObj = { p1, p2 };

This gives us defaults, renaming, and deep picking, but the property names must still be written out twice.

We could write

    (({p1, p2}) => ({p1, p2})(o)

Using destruturing of arguments, but this seems more trouble than it's worth, and the properties must still be written out twice.

### Summary

In the simplest case where we are picking a single property from an object, we can write `o.p`, but have to worry about null objects and defaults ourselves. We look for syntactic sugar which allows to say the equivalent of:

> for property p, using default d, pick from object o, assuming it's an object

instead of writing

    o && typeof o === 'object' ? 'p' in o ? o.p : 42 : void 0

In the equally common case, where we are picking two properties from an object, we have to write `{ p1: o.p1, p2: o.p2 }`, which allows us renaming, but no default or null object handling.

> create an object with the value of property p1, using default d1, and renamed to q1, and the value of property p2, using default d2, and renamed to q1, pick from object o, assuming it's an object

instead of writing

    o && typeof o === 'object' ? { q1: 'p1' in o ? o.p1 : void 0, q2: 'p2' in o ? o.p2 } : {}



## Basics

Our proposed solution to the motivations above is based on a new operator, the pick operator. The pick operator picks a property from an object. In its simplest form, it is the inverse of the dot operator.

    p # o                 // o.p

But it also provides defaults:

    p = 42 # o

and, using the `?` modifier to `#`,  null-object checking, called "maybe picking":

    p ?# o               // o && typeof o === 'object' ? o.p : void o

We can pick **into** objects, with

    { p1, p2 } # o

with defaults and null-object checking as above, in addition to renaming:

    { p1: q1, p2 } # o   // {q1: o.p1, p2: o.p2 }

Combining these

    { q1: p1 = 42, p2 } ?# o

For deep-picking, assume `o = { p1: 1, p2: { p21: 21, p22: 22 } }`, we could write

    { p1, p2: { p22 } } # o

but this would yield `{ p1: 1, p2: { p22: 22 } }`. To obtain `{ p1: 1, p22: 22 }`, we use nested picks:

    { p1, p22 # p2 } # o

By extension, we can pick one or more properties into an array:

    [ p1, p2 ] # o        // [ o.p1, o.p2 ]

We can assign variables, current deconstructing assignment does,  based on a pick using the **pick assignment** operator.

    let a =# o;          // let a = o.a;

We can pick from an array instead of an object using the **array pick** operator.

    { a, b } @ [ 1, 2 ]  // { a: 1, b: 2 }

with analogous **assignment array pick** and **maybe array pick** operators also available as `@=` and `?@`.


In all of the above, the basic form of a "pick" is

    picker [=][?]# object

In the next section, we describe the basic form of the left hand side, the picker.


## Pickers

The left operand of the pick operator is a **picker**.
A picker is one of a simple picker, an object picklist, an array picklist, or a variable picklist.

### Simple picker

A **simple picker** has the following syntax.

    key[modifier] [:newname] [= default]

`modifier` refers to the mandatory or not allowed modifiers discussed below.

### Subpicker

Another form of the simple picker is the **subpicker** mentioned above, which has the form

    picker # pick

which means to take the result of `pick`, and then pick from from it using `picker`.
This allows us to say

    { a # b, c # d } # { b: { a: 1 }, d: { c: 2 } }      // { a: 1, c: 2 }

Subpickers can be any number of levels deep:

    { a # b # c } # { c: { b: { a: 99 } } }              // { a: 99 }

Subpickers can have a "picklist" on their left side,
to allow picking of multiple properties from an intermediate pick.

    { (a, b) # c } # { c: { a: 1, b: 2 } }               // { a : 1, b : 2 }


### Picklists

A picklist is one of the following.

 1. an object-like construct containing pickers (**object picklist**), used to pick properties into an object

 1. an array-like construct containing pickers (**array picklist**), used to pick properties into an array

 1. a parenthesized list of pickers, used in assignments (**variable picklist**), used to pick properties into variables, or pick multiple properties in a subpicker


### Spreads in picklists

In object picklists and array picklists, we support an empty spread operator in the final position. It refers to "remaining" properties.

    { a: x, ... } # o      // { x: o.a, p1: o.p1, p2: o.p2, ... }

We can also use spreads with the **maybe pick operator** (see below).

    {a: x, ... } ?# null   // { x: undefined }


### Computed pickers and other exotica

To pick a property whose name is given by an expression, we use the suffix `*` operator:

    propname* # o            // o[propname]

If the value of a computed picker resolves to a string, it is interpreted as a property name.

    propname = 'ab'
    { propname* # { ab: 1 }  // { ab: 1 }

If it is a string-valued expression, no `*` is necessary:

    propname.toLowerCase() # o

If the value of a computed picker resolves to an array, its elements are interpreted as property names:

    propnames = [ 'p1', 'p2' ]
    { propnames* } # o       // { p1: o.p1, p2: o.p2 }

If it is an array-valued expression, no `*` is necessary:

    propnames.map(p => p.toLowerCase()) # o

If the value of a computed picker resolves to an object, its keys are used as the properties to be picked:

    propobj = { p1: 1, p2: 2 }
    { propobj* } # o         // { p1: o.p1, p2: o.p2 }

If it is an object-valued expression, no `*` is necessary:

    Object.assign(propobj, { p3: 3 }) # o

If the picker is a regular expression, it yields all matching property names:

    { /^p/ } # o             // { p1: o.p1, p2: o.p2 }

A function given as a pickname acts as a filter on property names:

    { p => /2/.test(p) } # o // { p2: o.p2 }

We can rename properties based on an expression using the computed picker operator `*`:

    { p: newname* } # o    // { [newname]: o.p }

We can rename properties, including multiple ones, by giving a function as the operand of `as`.
The function is invoked with the property name, and must return a string.

    { /^p/: p => p.replace('p', 'q' } # { p1: 1, p2: 2 }   // { q1: 1, q2: 2 }


### Mandatory and disallowed picking

We use the exclamation mark to indicate that a key specified in a picker *must* exist, otherwise a ReferenceError is thrown.

    p! # o                    // { p: o.p }; throws if p is missing

The mandatory operator and a default specifier are mutually exclusive.

We use the caret to indicate that a key specified in a picker *must not* exist.

    (p, q^) # o               // { p: o.p }; throws if q is present

and can specify its default if desired, which will always be used.

    (p, q^ = 99) # o               // { p: o.p, q: 99 }; throws if q is present

We can check that all desired keys, given as an array of strings, exist:

    { keys*! } # o

and to add a check that no other keys exist:

    { keys*!, ...^ } # o

Check that no key starts with `q`:

    { /^q/^ } # o


## Pick assignment

Picking has an obvious relationship to deconstructing assignments, of the form

    { p } = o;

To generalize and enhance deconstruction, we propose a **pick assignment operator** `#=`, as in

    p #= o;                     // p = o.p;, or { p } = o

Multiple pickers (target variables) can be specified by providing a parenthesized list called a **variable picklist**:

    ( p1, p2 ) #= o;            // p1 = o.p1; p2 = o.p2; or, { p1, p2 } = o;

or to also declare the variables:

    let ( p1, p2 ) #= o;        // let p1 = o.p1, p2 = o.p2;

Like all pickers, the picker on the left side of `#=` could include defaults and renamers, so

    p: x = 42 #= o;

means to find the `p` property in `o`, or use 42 if it is not present, and assign the result to `x`.

The left hand operand of the pick assignment operator must be a simple picker, or a variable picklist. It is meaningless to assign to an array or object.

      p   #= o   // assign o.p to variable p
    ( p ) #= o   // same as above
    { p } #= o   // syntax error
    [ p ] #= o   // syntax error


## The "maybe pick" operator

To handle the existential/null propagation case, we introduce a variant of `#` called `?#`,
named the **maybe pick operator**:

    p ?# o

which returns void 0 is `o` if not an object, and can be chained as in

    q ?# p ?# o             // CoffeeScript o?.p?.q

Defaults and renamers can be used with the maybe pick operator just as with the pick operator, so

    p: x  = 42 ?# o

The maybe pick operator and be combined with the assignment pick operator,
yielding the **maybe pick assignment operator**, `#= if`.

    p ?#= o

This means

    p = Object.is(o) ? o.p : undefined

An advantage over of some currently proposed existential operator syntax is the following.
Consider an object

    {a: { b : { c1: 1, c2: 2 } }

where we want to obtain

    { c1: a.b.c1, c2: a.b.c2 }

With currently proposed syntax for existential checks, so we would have to write

    { c1: a?.b?.c1, c2: a?.b?.c2 }

whereas with the above instead we can write

    { c1, c2 } ?# b ?# a


## Picking from arrays: the array pick operator

To pick from arrays, we introduce the **array pick operator** `@`, analogous to `#`:

    { a, b } @ array      // { a: array[0], b: array[1] }

The array pick operator has the same "maybe" variant, the **maybe array pick operator**:

    { a } ?@ array

A single picker is the equivalent of `head`:

    a @ array

When picking from arrays, when the LHS is a picklist,
the order of pickers corresponds to the order of elements in the array,
with the normal convention of being able to elide elements:

    { a, , b } @ array    // { a: array[0], b: array[2] }

We can pick from an array onto an array:

    [ 1, 0 ] @ array

### Array pick assignment

We can declare/assign using the **array pick assignment operator**, '@=':

    let (a, b) @= array;

By extension, we also have a **maybe array pick assignment operator**.

    [a, b] ?@= array

The equivalent of today's `[a, b] = [b, a];` is

    (a, b) @= [b, a];

The power of this syntax is demonstrated by this example. Given an array of sort indexes, we can apply it using

    [ indexes* ] @= array


## Picking from arguments

Currently function arguments can be deconstructed as follows:

    function f({ a }) { }

The equivalent using pick notation is to use the pick (or array pick) operator,
including their maybe variants, with *no right operand*.
The right operand is implicitly the argument in that position in the argument list, so

    function f(a #) { }
    f({a: 1})

corresponds to the above. Similarly, we can deconstruct array arguments with

    function f((a, b) @) { }
    f([1, 2])

or pick out nested properties:

    function f(c # b #) { }
    f({ b: { c: 1 } })

We can give defaults and do renaming.

    function f(c: x = 42 # b #) { }

To my knowledge, no current proposal for deconstructing arguments addresses existential checking.
With the maybe pick operator, it's easy:

    function(c ?#) { }


## Fat arrow picking

It is common to have functions whose only job is picking.
Let's say I wanted to pick the `a` property from an array of objects:

    objects . map(object => a # object)

Due the frequency of this idiom, we provide the fat arrow pick:

    objects . map(a #=>)

Which also comes in a maybe version:

    objects. map(a ?#=>)


## Grammar

### Operators

| Operator | Meaning |
|:-------- |:-------- |
| #        | pick     |
| ?#       | maybe pick |
| #=       | pick assignment |
| ?#=      | maybe pick assignment |
| @        | array pick |
| ?@       | maybe array pick |
| @=       | array pick assignment |
| ?@=      | maybe array pick assignment |
| #=>      | fat arrow pick |
| -------- | -------- |

### Pickers

| Syntax   | Name | Meaning |
|:-------- |:---- |:------- |
| a        | simple picker | pick property `a` |
| a!       | mandatory picker | pick property `a`, throw if missing |
| a^       | disallow picker | throw if property `a` is present |
| a*       | computed picker | pick property with key given by value of `a` |
| a = 42   | picker with default | pick property `a` with default |
| b: a     | picker with rename | pick property `b` and rename to `a` |
| b: a = 42 | picker with rename and default | pick property `b` and rename to `a`, defaulting to 42 |
| b: fn    | picker with functional rename | pick property `b` and rename with result of calling fn |
| b: a*    | picker with computed rename | pick property `b` and rename with value of `a` |
| /regexp/ | regexp picker | pick properties matching regexp |
| fn       | filter picker | pick properties passing filter |
| ...      | spread picker | pick remaining properties/elements |
| a # b    | subpicker     | pick `a` from the result of picking `b` |
| { a, b } | object picklist | pick into object |
| [ a, b ] | array picklist | pick into arary |
| ( a, b ) | variable picklist | pick into variables (assignment only), or pick multiple properties in a subpicker |
| -------- | -------- | -------|

## Revision History

| Version  | Date | Content |
|:-------- |:---- |:------- |
| 0.1      | 2015-06-20 | Change rename to use `:`. Remove "this pickers". Change maybe pick to `?#`. |
| -------- | -------- | -------|
