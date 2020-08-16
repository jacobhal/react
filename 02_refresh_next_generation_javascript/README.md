# Section 2: Refreshing Next Generation JavaScript

## let vs const vs var
var, let and const are different ways of creating variables.

The reason why let keyword was introduced to the language was function scope is confusing and was one of the main sources of bugs in JavaScript.

### var
* The old-school basic JavaScript way to define a variable. You are however encouraged to use either let or const.
* Variables declared by var keyword are scoped to the immediate function body (function scope) which can introduce a lot of bugs.

### let
* let would be the "new" var keyword.
* let variables are scoped to the immediate enclosing block denoted by { } (block scope).

### const
* Variables that do not change, constant variables.

## Arrow Functions
Arrow functions are an alternative way to declare JavaScript functions.

### Why were Arrow Functions introduced?
* The arrow functions declarations does not need a "function" keyword so it is a bit shorter.
* Solves a lot of problems with the *this* keyword. When using *this* in an arrow function, the reference will always keep it context and not change during runtime.

### Examples

Below are some examples of equivalent function declarations:

```JSX
// Example 1
function myFnc() { }

const myFnc = () => { }

// Example 2
function myFnc(param1) { }

const myFnc = param1 => { }

// Example 3
function myFnc(param1, param2) { }

const myFnc = (param1, param2) => { }

// Example 4
function double(number) {
    return number * 2;
}

const double = number => number * 2; // We omit the return keyword
```

## Exports and Imports
Another next generation JavaScript feature is writing modular code, that is, splitting your code into multiple files.

### export default vs named exports
The default export is the default thing that gets exported without a name. If we for instance want to import the person object in the code example below, we can name it whatever we want because it is the default export of that module. There can only be one default export in a file.

There can however be multiple named exports like in utility.js below.

```JSX
// person.js
const person = {
    name: 'Jacob'
}

export default person

// utility.js
export const clean = () => { ... }

export const baseData = 10;

// app.js
import person from './person.js'
import prs from './person.js'

import { baseData } from './utility.js'
import { clean } from './utility.js'
import { baseData as baseDataAlias } from './utility.js' // Import with an alias
import * as bundled from './utility.js' // Import everything as an object called bundled and call functions like this: bundled.clean
```


## Understanding Classes

## Classes, Properties and Methods

## The Spread & Rest Operator

## Destructuring

## Reference and Primitive Types Refresher

## Refreshing Array Functions
