---
title: The JavaScript pick operator
layout: default
---

This document presents a new "pick" operator for JavaScript.

## Pick operator

Consider a **pick operator**, also called the "boberator":

    p # o                 // o.p
    q # p # o             // o.p.q

We can pick into an object:is

    { p1, p2 } # o        // { p1: o.p1, p2: o.p2 }

To pick more deeply

    { p1, q # p2 } # o    // { o.p1, o.p2.q }

We can pick into an array with

    [ p1, p2 ] # o        // [o.p1, o.p2]

We can provide default values

    p = 42 # o

or change names on the fly using the `from` keyword:

    { x from p1, p2 } # o // { x: o.p1, p2: o.p2 }

or both

    { x from p1 = 42, p2 } # o

We will call the construct `[name from] key [= default]` a **picker**. Then the left hand operator of `#` is always one of

 1. a picker

 1. an object-like construct containing pickers

 1. an array-like construct containing pickers

 1. a parenthesized list of pickers, used in assignments (see below)

In terms of associativity, therefore

    q # p # o

is parsed as

    q # (p # o)

and indeed `(q # p) # o` is invalid syntax and in fact meaningless.

### Pick assignment

There is an obvious relationship to deconstructing assignments. Instead of

    { p } = o;

which has always struck me as a bit odd, and I have found newcomers wrestling with it, in vague analogy to the addition assignment operator `+=`, we propose a **pick assignment operator**, as in

    p #= o;

Multiple pickers can be specified by providing a parenthesized list called a **variable picklist**:

    ( p, q ) #= o;

or

    let ( p, q ) #= o;

Like all pickers, the picker on the left side of `#=` could include defaults and renamers, so

    x from p = 42 #= o;

means to find the `p` property in `o`, or use 42 if it is not present, and assign the result to `x`.

### Existential operator

To handle the existential/null propagation case, we introduce a variant of # called `# if` (or `#?` if you prefer), named the **maybe pick** operator:

    p #? o

which returns void 0 is o is not an object, and can be chained as in

    q #? p #? o

which is the equivalent of

    o?.p?.q

Perhaps the `if` alternative is more readable:

    q # if p # if o

Below we will use `# if`. Defaults and renamers can be used with the maybe pick operator just as with the pick operator, so

    x from p = 42 # if o

The maybe pick operator and be combined with the assignment pick operator, so

    p #= if o

essentially means

    p = o-is-object ? o.p : undefined

An advantage of some currently proposed existential operator syntax, consider an object

    {a: { b : { c1: 1, c2: 2 } }

where we want to obtain

    { c1: a.b.c1, c2: a.b.c2 }

but with currently proposed syntax for existential checks, so we would have to write

    { c1: a?.b?.c1, c2: a?.b?.c2 }

whereas with the above instead we could write

    { c1, c2 } # if b # if a

### Spreads in pick lists

In object-style (`{ a } # o`) and array-style (`[ a ] # o`) pick lists, we support an empty spread operator in the final position, as in

    { x from a, ... } # o    // { x: o.a, p1: o.p1, p2: o.p2, ... }

or

    {x from a = 42, ...} # if o

With some trepidation, we could extend the use of the spread in parenthesized pick lists used in pick assignments:

    (x from a, ...) #= o

which would create a variable `x` from the `a` property and additional variables from the remaining properties, although I have no idea how to lint that.

### Computed property names

We support computed computed property names in renamers:

    { [name] from a } # o

### Picking from arrays

To pick from arrays, we introduce the array pick operator `@`, analogous to `#`:

    { a, b } @ array

which results in `{ a: array[0], b: array[1] }`.

The array pick operator has the same "maybe" variant, the **maybe array pick operator**:

    { a } @ if array

A single picker is the equivalent of `head`:

    a @ array

We can also use the empty spread operator here, so tail is

    [, ...] @ array

We can declare/assign using the **array pick assignment operator**, '@=':

    let (a, b) @= array;

With the array picker, when the LHS is object-style or array-style, the order of pickers corresponds to the order of elements in the array, with the normal convention of being able to elide elements:

    { a, , b } @ array

We can pick from an array onto an array:

    [ 1, 0 ] @ array

### Picking from arguments

To provide the equivalent of deconstructing function arguments:

    function f({ a }) {

we use the pick (or array pick) operator, including their maybe variants, with *no right operand*. The right operand is implicitly the argument in that position in the argument list, so

    function f(a #) {

corresponds to the above. Similarly, we can deconstruct array arguments with

    function f([a, b] @) {

or pick out nested properties:

    function f(c # b #) {

or give defaults and do renaming and do existential checking:

    function f(x from c = 42 # b # if) {

## Grammar

### Operators

| Operator | Meaning |
| -------- | -------- |
| #        | pick     |
| # if     | maybe pick |
| #=       | pick assignment |
| #= if    | maybe pick assignment |
| @        | array pick |
| @ if     | maybe array pick |
| @=       | array pick assignment |
| @= if    | maybe array pick assignment |
| -------- | -------- |

### Pickers and picklists

| Syntax   | Name | Meaning |
| -------- | ---- | ------- |
| a        | simple picker | pick property `a` |
| a = 42   | picker with default | pick property `a` with default |
| a from b   | renaming picker | pick property `b` and rename to `a` |
| ...      | spread picker | pick remaining properties/elements |
| { a, b } | object picklist | pick into object |
| [ a, b ] | array picklist | pick into arary |
| ( a, b ) | variable picklist | pick into variables (assignment case) |
| -------- | -------- |

## Prototype
