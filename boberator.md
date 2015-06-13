---
title: The JavaScript pick operator
layout: default
---

# JavaScript "Pick"  Operator

This document presents a proposed "pick" operator for JavaScript, also known as a "boberator". Here we use the sharp sign.

## Basics

The pick operator picks a property from an object. In its simplest form, it is the inverse of the dot operator.

    p # o                 // o.p

The basic syntax is

    picker # object

The pick operator right-associates, so we can go down multiple levels.

    q # p # o             // o.p.q

We can pick multiple properties into an object.

    { p1, p2 } # o        // { p1: o.p1, p2: o.p2 }

We can pick some of them more deeply.

    { p1, q # p2 } # o    // { o.p1, o.p2.q }

We can pick into an array.

    [ p1, p2 ] # o        // [o.p1, o.p2]

We can provide default values in case the property is not present on the object, using an equal sign.

    p = 42 # o

We can change names on the fly using a **renamer** specified by the `as` keyword.

    { p1 as x, p2 } # o // { x: o.p1, p2: o.p2 }

We use `as` rather than the colon, to avoid conceptual confusion with standard object literal syntax.

We can give both a default and a renamer.

    { p1 as x = 42, p2 } # o


## Pickers

A picker is one of a simple picker, an object picklist, an array picklist, or a variable picklist. The left operand of the pick operator is a picker.

### Simple picker

A **simple picker** has the following syntax.

    key [as newname] [= default]

### Picklists

A picklist is one of the following.

 1. an object-like construct containing pickers (**object picklist**), used to pick properties into an object

 1. an array-like construct containing pickers (**array picklist**), used to pick properties into an array

 1. a parenthesized list of pickers, used in assignments (**variable picklist**), used to pick properties into variables

### Associativity

Since the left operand must be a picker, whether a simple picker or a picklist,

    q # p # o

is parsed as

    q # (p # o)

It cannot be parsed as `(q # p) # o`, since `(q # p)` is not a picker. This is invalid syntax and in fact meaningless.


### Spreads in pick lists

In object picklists and array picklists, we support an empty spread operator in the final position. It refers to "remaining" properties.

    { a as x, ... } # o     // { x: o.a, p1: o.p1, p2: o.p2, ... }

We can also use spreads with the **maybe pick operator** (see below).

    {a as x, ... } # if o   // { x: undefined }


### Computed pickers and other exotica

To pick a property whose name is given by an expression, we use the prefix `*` operator:

    *propname # o               // o[propname]

When picking into objects, We can rename it if we so choose:

    { *propname as x } # o      // { x: o[propname] }

If the value of a computed picker resolves to an array, its elements are interpreted as property names:

    { *['p1', 'p2'] } # o       // { p1: o.p1, p2: o.p2 }

If the value of a comuputed picker resolves to an object, its keys are used as the properties to be picked:

    { *{ p1: 1, p2: 2 } } # o   // { p1: o.p1, p2: o.p2 }

If the picker is a regular expression, it yields all matching property names:

    { /^p/ } # o                // { p1: o.p1, p2: o.p2 }

A function given as a pickname acts as a filter on property names:

    { p => /2/.test(p) } # o    // { p2: o.p2 }

We can rename properties based on an expression using the computed picker operator `*`:

    { p as *newname } # o       // { [newname]: o.p }

We can also rename properties, including multiple ones, by giving a function as the operand of `as`. The function is invoked with the property name:

    { /^p/ as p => p.replace('p', 'q' } # { p1: 1, p2: 2 }   // { q1: 1, q2: 2 }

### Mandatory and exclusionary picking

We use the exclamation mark to indicate that a key specified in a picker *must* exist, otherwise a ReferenceError is thrown.

    p! # o                    // { p: o.p }; throws if p is missing

We use the caret to indicate that a key specified in a picker *must not* exist.

    (p, q^) # o               // { p: o.p }; throws if q is present


## Pick assignment

Picking has an obvious relationship to deconstructing assignments. Consider:

    { p } = o;

This has always struck me as a bit odd, and something in my experience newcomers typically wrestle with.
Instead, in vague analogy to the addition assignment operator `+=`, we propose a **pick assignment operator** `#=`, as in

    p #= o;                     // p = o.p;

Multiple pickers (target variables) can be specified by providing a parenthesized list called a **variable picklist**:

    ( p1, p2 ) #= o;            // p1 = o.p1; p2 = o.p2;

or to also declare the variables:

    let ( p1, p2 ) #= o;        // let p1 = o.p1, p2 = o.p2;

Like all pickers, the picker on the left side of `#=` could include defaults and renamers, so

    p as x = 42 #= o;

means to find the `p` property in `o`, or use 42 if it is not present, and assign the result to `x`.

The left hand operand of the pick assignment operator must be a simple picker, or a variable picklist. It is meaningless to assign to an array or object.

      p   #= o   // assign o.p to variable p
    ( p ) #= o   // same as above
    { p } #= o   // syntax error
    [ p ] #= o   // syntax error


## The "maybe pick" operator

To handle the existential/null propagation case, we introduce a variant of `#` called `# if` (or `#?` if you prefer), named the **maybe pick operator**:

    p # if o

which returns void 0 is `o` is not an object, and can be chained as in

    q # if p # if o

which is the equivalent of

    o?.p?.q

Defaults and renamers can be used with the maybe pick operator just as with the pick operator, so

    p as x  = 42 # if o

The maybe pick operator and be combined with the assignment pick operator, yielding the **maybe pick assignment operator**, `#= if`.

    p #= if o

This means

    p = Object.is(o) ? o.p : undefined

An advantage over of some currently proposed existential operator syntax is the following. Cconsider an object

    {a: { b : { c1: 1, c2: 2 } }

where we want to obtain

    { c1: a.b.c1, c2: a.b.c2 }

With currently proposed syntax for existential checks, so we would have to write

    { c1: a?.b?.c1, c2: a?.b?.c2 }

whereas with the above instead we can write

    { c1, c2 } # if b # if a


## Picking from arrays: the array pick operator

To pick from arrays, we introduce the **array pick operator** `@`, analogous to `#`:

    { a, b } @ array           // { a: array[0], b: array[1] }

The array pick operator has the same "maybe" variant, the **maybe array pick operator**:

    { a } @ if array

A single picker is the equivalent of `head`:

    a @ array

When picking from arrays, when the LHS is a picklist, the order of pickers corresponds to the order of elements in the array, with the normal convention of being able to elide elements:

    { a, , b } @ array    // { a: array[0], b: array[2] }

We can pick from an array onto an array:

    [ 1, 0 ] @ array

### Array pick assignment

We can declare/assign using the **array pick assignment operator**, '@=':

    let (a, b) @= array;

By extension, we also have a **maybe array pick assignment operator**.

    [a, b] @= if array

The equivalent of today's `[a, b] = [b, a];` is

    (a, b) @= [b, a];

The power of this syntax is demonstrated by this example. Given an array of sort indexes, we can apply it using

    [ *indexes ] @= array


## Picking from arguments

Currently function arguments can be deconstructed as follows:

    function f({ a }) { }

The equivalent using pick notation is to use the pick (or array pick) operator,
including their maybe variants, with *no right operand*.
The right operand is implicitly the argument in that position in the argument list, so

    function f(a #) { }

corresponds to the above. Similarly, we can deconstruct array arguments with

    function f((a, b) @) { }

or pick out nested properties:

    function f(c # b #) { }

We can give defaults do renaming.

    function f(c as x = 42 # b #) { }

To my knowledge, no current proposal for deconstructing arguments addresses existential checking.
With the maybe pick operator, it's easy:

    function(c # if) { }


## Grammar

### Operators

| Operator | Meaning |
|:-------- |:-------- |
| #        | pick     |
| # if     | maybe pick |
| #=       | pick assignment |
| #= if    | maybe pick assignment |
| @        | array pick |
| @ if     | maybe array pick |
| @=       | array pick assignment |
| @= if    | maybe array pick assignment |
| -------- | -------- |

### Pickers

| Syntax   | Name | Meaning |
|:-------- |:---- |:------- |
| a        | simple picker | pick property `a` |
| a!       | simple mandatory picker | pick property `a`, throw if missing |
| a^       | simple exclusion | throw is property `a` is present |
| *a       | simple computed picker | pick property with key given by value of `a` |
| a = 42   | picker with default | pick property `a` with default |
| b as a   | renaming picker | pick property `b` and rename to `a` |
| b as fn  | renaming picker | pick property `b` and rename with result of calling fn |
| /regexp/ | regexp picker | pick properties matching regexp |
| fn       | filter picker | pick properties passing filter |
| b as a = 42  | renaming picker with default | pick property `b` and rename to `a`, defaulting to 42 |
| ...      | spread picker | pick remaining properties/elements |
| { a, b } | object picklist | pick into object |
| [ a, b ] | array picklist | pick into arary |
| ( a, b ) | variable picklist | pick into variables (assignment only) |
| -------- | -------- |
