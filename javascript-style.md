---
title: Bob's Javascript Style Guide
layout: default
---

# JavaScript coding guidelines

Always use semi-colons.

## 1. Whitespace

### 1.1. Horizontal whitespace

#### 1.1.1. Indentation

Use two-space indents. Configure your editor accordingly. Or install its `editorconfig` plugin. This also applies to CSS and HTML.

#### 1.1.2. Use spaces around operators

    1 + 2
    a === 'b'
    flag ? 1 : 2

However spaces around the dot operator are optional:

    a.b
    a . b

#### 1.1.2. Use spaces after commas separating items in lists

    func(a, b)
    [1, 2]

This also applies to object literals, where you should also use spaces after the colon in property definitions (but *not* before the colon).

    {a: 1, b: 2}

#### 1.1.3. Delimiters

By "delimiter" we mean parentheses, curly brackets, and square brackets. Put a space before opening delimiters and after closing delimiters. This includes the parentheses around the expressions in `if`, `for`, `while`, and `switch` statements. Do not put spaces *after* opening delimiters or *before* closing delimiters. Do not put spaces between adjacent opening delimiters or closing delimiters.

    if (flag) x = 1;
    for (let x
    while (true)
    switch (val) {

The exception is the parenthesis introducing a named function's parameter list, and the last closing delimiter before a semicolon.

#### 1.1.4. Alignment

Avoid using horizontal whitespace to line things up.

    const a   = 1;
    let  flag = true;

### 1.2. Vertical whitespace

#### 1.2.1. Separate functions and methods

Use vertical whitespace to separate functions and methods. Two blank lines may be used (sparingly) to separate larger logical sections.

#### 1.2.2. Use vertical space within methods

Use vertical space within functions to separate sections of code.

#### 1.2.3. Newline at EOF

End files with a newline. `editorconfig` settings will enforce this.

## 2. Comments

### 2.1. Comment style

Prefer `//` comments.

### 2.2. Comment at top of file

Start the file with a comment giving the file path and overall purpose.

### 2.3. Comments for method or function

Precede each method or function with at least one `//` comment.

### 2.4. Comments on sections of methods

Precede each section within a function with a `//` comment.

### 2.5. `FIXME` and `TODO` comments

Use as needed. Feel free to use the `FIXMEEEE` convention to indicate priority.

### 2.6. Old commented code

Commented code is usually never addressed. Try not to leave commented code unless really required.

## 3. Blocks

### 3.1. Blocks in curlies

In general, blocks start with a curly bracket on the first line, continue on additional lines, and end with a closing bracket on the last line.
Examples include the body of `if` or `for` statements.

    if (flag) {
      doFlagAction();
    }

### 3.2. Same-line functions

However, a short function may and often should be placed on the same line. This avoids vertical sprawl.

    function (a) {return a + 1;}

### 3.3. Braceless blocks

Braces may be, and often should be, omitted for one-statement blocks on the same line.

    if (flag) doFlagAction();

### 3.4. Else blocks

Else blocks follow the same rule:

    if (flag) {
      doFlagAction();
    } else {
      doNotFlagAction();
    }

However, it is also completely accepetable to abbreviate this as

    if (flag) doFlagAction();
    else doNotFlagAction();

## 4. Export/import

### 4.1. Imports at top

Put all your imports at the top.

### 4.2. Location of exports

Put all your exports at the bottom, or export individual values when declared, as you please.

## 5. `const`, and `let`

### 5.1. Multiple `const` and `let`

Use multiple ``const`/`let` statements.
This makes them easier to add and delete without worrying about commas.

    const a = 1;
    let b;

Avoid `var`.

## 6. Quotes

### 6.1. Machine-oriented strings

Use single quotes for machine-oriented strings.

### 6.2. Human-oriented strings

Use double quotes for human-oriented strings.

## 7. Variable names

### 7.1. Variable names

camelCased.
Avoid overly short names like `p2`.

    let myVar;

### 7.2. Constant names

Use all upper-case for constant names, with underscores. Prefer declaring them with `const`.

    const TIMEOUT_MS = 1000;

### 7.3. Class or import names

Use Pascal case (initial cap).

    import CoolMixin from '../mixins/cool';

### 7.4. Meaningful names

Use meaningful names; avoid names such as `flag` or `value` or `object`.

### 7.5. Really long names

Avoid ridiculously long names.

### 7.6. Boolean variables

Prefer to prefix boolean variables with `is`, `has`, `can`, etc.

### 7.7. Clear variable names

If `model` is a model, and you want to refer to its name or ID, do not name the variable `model`, but rather `modelName` or `modelId`.

### 7.8. Array names

If something is an array, call it `dogs`, not `dogArray`.

## 8. Operators

### 8.1. Use `===` and `!==`

### 8.2. Falsiness

Although Crockford disagrees, use `!` to check for null, undefined, zero, false, and empty strings.
Use `if (array.length)`, not `if (array.length > 0)`.

## 9. Chaining calls

### 9.1. Dot spacing

When a chain does not fit on one line, use the dot on the following line:

    find(id)
      .filter(isActive)
      .map(getName);

## 10. Other issues

### 10.1. Reference to `this`

Avoid using `self`, `_this`, or `that` to save references to `this` for use within inner functions. Instead, in order of preference:

 * Use ES6 arrow functions.
 * Use the `thisArg` parameter to `forEach` etc.
 * Bind the function.
 * Obtain the value you need from `this` outside the inner function.

### 10.2. Property access

Use `foo.bar`, not `foo['bar']`.
Do not quote property names in object literals unless you have to.

### 10.3. Conditionals

Never write `x = condition ? true : false;`.
Just say `x = condition;`
Do not write `if (x) return true; else return false;`.
Do not compare booleans to `true` or `false`.
Use the ternary operator for brevity.

### 10.4. Early returns

Early returns promote code readability and avoid nesting.

    function foo(err) {
      if (err) return;
      ...
    }

### 10.5. Parentheses

#### 10.5.1. Unnecessary parens

Use parentheses only where required by the syntax and semantics.
But feel free to use parentheses in boolean expressions involving `&&` and `||` to make the grouping clear.

#### 10.5.2 No parens on unary operators

Do not use parentheses for unary operators such as delete, typeof and void, or after keywords such as `return`, `throw`, `case`, `in` or `new`.

### 10.6. ES6 syntax

Prefer the use of ES6 syntax for brevity, including:

 * Shorthand object literals, such as `{x, y}`, and `{ foo() { } }` for methods
 * Arrow functions
 * Deconstruction in assignments: `const {a, b} = object;`
 * Spread operator: `function F(...a)`
 * Template literals (`foo.com?param=${param}`)
 * Computed property names
 * Default parameters

### 10.6.1 Arrow functions

Use "concise bodies" (single returned expression) where possible.
Do not parenthesize single arguments:

    a => a + 1

### 10.6.2 Modules

Prefer default exports where possible.
Instead of exporting an object of values, export each value as a named export.
When importing, if the import is "nearby", use `import foo from './foo';`.

#### 10.6.2. ES6 features

## 10.7. Objects

## 10.8. Functions

Prefer function declarations to function expression assignments.

Use object literals to create objects.

## 11. CSS

Use the same rules as for JS.

### 11.1. Horizontal spacing

Simple rules may be placed on the same line:

    .big {font-size: x-large;}

### 11.2. Vertical spacing

Use vertical spacing to delineate rulesets.

### 11.3. Comments

11.3.1. Place a comment for entire file at top; comment blocks of declarations.

11.3.2. Use `/* */ comments, even in SCSS which supports `//`.

### 11.3. Things to avoid

 * in SCSS/SASS/LESS, unreasonably deep nesting (more than 2-3 levels)
 * unnecessary dependencies on particularities ofthe HTML structure
 * `@extend`
 * hard-wired colors (use variables if available)

## 12. HTML

Use the same rules as for JS.

### 12.1. Whitespace

#### 12.1.2. Horizontal whitespace

Do not put spaces after `<` beginning an HTML element, or before `>` closing it.
Do not put spaces around `=` giving attribute values or named parameters to helpers/views.
Do not close tags such as `<img>` which do not need to be closed.
Quote attributes.

    <a href="boo.html">
    {{view 'index' model=foo}}

### 12.2. Quotes

Use double quotes for HTML element attribute values.

### 12.3. Comments

#### 12.3.1. Comment at top

Place comment for entire file at top; comment blocks of code.

#### 12.3.2. Comment syntax

Use your templating engine's commenting mechanism, if available,
in preference to `<!-- -->`, which remain in the generated HTML.

## 13. Differences from other standard styleguides

 * Allow braces to be omitted for one-statement blocks on the same line.
 * Distinguish between single (machine) and double (human) quotes.
 * Use multiple `const`/`let` statements.

## 15. Use JSCS or eslint  to check your code styling before pushing it

### 15.1 Installing JSCS

JSCS is a simple npm module which can be installed globally.
Run the command:

    npm install jscs -g

### 15.2 Usage

You can check your code by typing this command at the root of your project:

    jscs [path-name]

### 15.3 Auto-formatting your code styling

This will apply fixes to some style rules. (whitespace rules, EOF rule, and validateIndentation)
Use this command:

    jscs --fix [path-name] or jscs -x [path-name]
