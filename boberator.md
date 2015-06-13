---
title: The JavaScript pick operator
layout: default
---

# JavaScript "Pick"  Operator

This document proposes a "pick" operator for JavaScript, also known as the "boberator".
It is represented by the sharp sign.
The pick operator can be considered a generalization of deconstruction.
It also provides an elegant solution to the so-called existential operator problem.


## Basics

The pick operator picks a property from an object. In its simplest form, it is the inverse of the dot operator.

    p # o                 // o.p

The basic syntax is

    picker # object

The pick operator right-associates, so we can go down multiple levels.

    q # p # o             // o.p.q

We can pick multiple properties into an object.

    { p1, p2 } # o        // { p1: o.p1, p2: o.p2 }

We can pick object property values into an array.

    [ p1, p2 ] # o        // [o.p1, o.p2]

We can provide default values in case the property is not present on the object, using an equal sign.

    p = 42 # o

We can change names on the fly using a **renamer** specified by the `as` keyword.

    { p1 as x, p2 } # o // { x: o.p1, p2: o.p2 }

We use `as` rather than the colon, to avoid conceptual confusion with standard object literal syntax.

We can give both a default and a renamer.

    { p1 as x = 42, p2 } # o

### Associativity

How is this parsed?

    q # p # o

It could be parsed as

    q # ( p # o }

were the pick operator right-associative.

But it could equally be parsed as

    ( q # p } # o

where `q # p` is a "subpicker" See below. either interpretation ultimately has the same semantics.


## Pickers

The left operand of the pick operator is a **picker**.
A picker is one of a simple picker, an object picklist, an array picklist, or a variable picklist.

### Simple picker

A **simple picker** has the following syntax.

    key[modifier] [as newname] [= default]

### Subpicker

Another form of the simple picker is the **subpicker**, which has the form

    picker-a # picker-b

which means to take the result of `picker-b`, and then pick from from it using `picker-a`. This allows us to say

    { a # b, c # d } # { b: { a: 1 }, d: { c: 2 } }      // { a: 1, c: 2 }

Subpickers can be any number of levels deep:

    { a # b # c } # { c: { b: { a: 99 } } }              // { a: 99 }

Subpickers can have a "picklist" on their left side, to allow picking of multiple properties from an intermediate pick.

    { (a, b) # c } # { c: { a: 1, b: 2 } }               // { a : 1, b : 2 }


### Picklists

A picklist is one of the following.

 1. an object-like construct containing pickers (**object picklist**), used to pick properties into an object

 1. an array-like construct containing pickers (**array picklist**), used to pick properties into an array

 1. a parenthesized list of pickers, used in assignments (**variable picklist**), used to pick properties into variables, or pick multiple properties in a subpicker


### Spreads in picklists

In object picklists and array picklists, we support an empty spread operator in the final position. It refers to "remaining" properties.

    { a as x, ... } # o      // { x: o.a, p1: o.p1, p2: o.p2, ... }

We can also use spreads with the **maybe pick operator** (see below).

    {a as x, ... } # if o    // { x: undefined }


### Computed pickers and other exotica

To pick a property whose name is given by an expression, we use the suffix `*` operator:

    propname* # o            // o[propname]

If the value of a computed picker resolves to a string, it is interpreted as a property name.

    propname = 'ab'
    { propname* # { ab: 1 }  // { ab: 1 }

If it is a string-valued expression, no `*` is necessary:

    prompname.toLowerCase() # o

If the value of a computed picker resolves to an array, its elements are interpreted as property names:

    propnames = [ 'p1', 'p2' ]
    { propnames* } # o       // { p1: o.p1, p2: o.p2 }

If it is an array-valued expression, no `*` is necessary:

    propnames.map(p => p.toLowerCase()) # o

If the value of a computed picker resolves to an object, its keys are used as the properties to be picked:

    propobj = { p1: 1, p2: 2 }
    { propobj* } # o         // { p1: o.p1, p2: o.p2 }

If it an object-valued epxression, no `*` is necessary:

    Object.assign(propobj, { p3: 3 }) # o

If the picker is a regular expression, it yields all matching property names:

    { /^p/ } # o             // { p1: o.p1, p2: o.p2 }

A function given as a pickname acts as a filter on property names:

    { p => /2/.test(p) } # o // { p2: o.p2 }

We can rename properties based on an expression using the computed picker operator `*`:

    { p as *newname } # o    // { [newname]: o.p }

We can rename properties, including multiple ones, by giving a function as the operand of `as`.
The function is invoked with the property name, and must return a string.

    { /^p/ as p => p.replace('p', 'q' } # { p1: 1, p2: 2 }   // { q1: 1, q2: 2 }


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

    { a, b } @ array      // { a: array[0], b: array[1] }

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
| a!       | mandatory picker | pick property `a`, throw if missing |
| a^       | disallow picker | throw if property `a` is present |
| a*       | computed picker | pick property with key given by value of `a` |
| a = 42   | picker with default | pick property `a` with default |
| b as a   | picker with rename | pick property `b` and rename to `a` |
| b as a = 42  | picker with rename and default | pick property `b` and rename to `a`, defaulting to 42 |
| b as fn  | picker with functional rename | pick property `b` and rename with result of calling fn |
| b as a*  | picker with computed rename | pick property `b` and rename with value of `a` |
| /regexp/ | regexp picker | pick properties matching regexp |
| fn       | filter picker | pick properties passing filter |
| ...      | spread picker | pick remaining properties/elements |
| a # b    | subpicker     | pick `a` from the result of picking `b` |
| { a, b } | object picklist | pick into object |
| [ a, b ] | array picklist | pick into arary |
| ( a, b ) | variable picklist | pick into variables (assignment only), or pick multiple properties in a subpicker |
| -------- | -------- | -------|
