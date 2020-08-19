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
Classes are just like in any OOPL. A class is instantiated using the *new* keyword like in most languages. Inheritance is available by using the *extends* keyword.

```JSX 
class Person extends BasePerson {
    constructor() {
        super(); // If you implement a constructor in a subclass you NEED the super keyword to call the parent
        this.name = 'Jacob'; // Property
    }      
    call () {...}       // Method
}

const myPerson = new Person();
myPerson.call();
```

## Classes, Properties and Methods
Next gen JavaScript offers a lighter and more modern way of specifying properties and methods in a class.

```JSX
class Person {
    name = 'Jacob'; // No constructor needed
    call = () => {} // ES7 Arrow Function (No problems with the "this" keyword since an arrow function is used)
}
```

## The Spread & Rest Operator

### Spread
The Spread operator is used to split up array elements OR object properties.

The example below uses spread syntax to create a new array/object based on the old versions and add some more properties or values.

```JSX
const oldArray = [3, 4];
const oldObject = {name: 'Jacob'};

const newArray = [...oldArray, 1, 2];           // [3, 4, 1, 2]
const newObject = {...oldObject, newProp: 5};   // {name: 'Jacob', newProp: 5}

const newArray2 = [newArray, 1, 2]              // [[3, 4], 1, 2]
```

### Rest
The Rest operator is used to merge a list of function arguments into an array.

The example below uses rest syntax to allow us to pass more than one argument to the sortArgs function.

```JSX
function sortArgs(...args) {
    return args.sort();
}
```

## Destructuring
Destructuring is a way to easily extract array elements or object properties and store them in variables.

```JSX
[a, b] = ['Jacob', 'Hallman'] // Array destructuring
{firstName} = {firstName: 'Jacob', lastName: 'Hallman'} // Object destructuring

```

## Reference and Primitive Types Refresher
In JavaScript you have reference and primitive types.

Primitive types (numbers, strings, booleans) copy over their value if you assign it to a new variable.

Reference types (objects, arrays) creates a reference (a pointer) if you assign it to a new variable.

```JSX
const number = 1;
const number2 = number;     // A copy of number (primitive type)

const person = { name: 'Jacob' };

const secondPerson = person; // A pointer to the same object in memory
const thirdPerson = {...person};

person.name = 'Max';

console.log(secondPerson.name); // Max
console.log(thirdPerson.name); // Jacob
```

> If you copy objects and arrays like above you will begin to see unexpected behaviour in your apps. You can use lodash functions to do deep copies instead of shallow ones or you can use the spread operator.

## Refreshing Array Functions

```JSX
const numbers = [1, 2, 3, 4, 5];
const numbers2 = [6, 7, 8, 9, 10];

// map
const numbersMapped = numbers.map( (number, i) => <li key={i}>{number * 2}</li> ); // [2, 4, 6, 8, 10]

// filter
const numbersFiltered = numbers.filter( (number) => number % 2 == 0); // [2, 4]

// reduce
const reducerFunc = (accumulator, currentValue) => accumulator + currentValue;;
const numbersReduced = numbers.reduce(reducerFunc); // 15

// sort
const numbersSorted = numbers.sort((a, b) => b - a); // [5, 4, 3, 2, 1]

// find
const numberFound = numbers.find(e => e > 2); // 3 (the first element that satisfies condition)

// findIndex
const numberIndexFound = numbers.find(e => e > 2); // 2 (index of the first element that satisfies condition)

// concat
const numbersMerged = numbers.concat(numbers2); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// slice
// The slice() method returns a shallow copy of a portion of an array into a new array object selected from start to end (end not included) where start and end represent the index of items in that array. The original array will not be modified.
// array.slice(start, end)
const numbersSliced = numbers.slice(2); // [3, 4, 5]
const numbersSliced = numbers.slice(2, 4); // [3, 4]
const numbersSliced = numbers.slice(1, 5); // [2, 3, 4, 5]
const numbersSliced = numbers.slice(2, -1) // [3, 4]
const numbersSliced = numbers.slice(-2, -1) // [4]

// splice
// The splice() method changes the contents of an array by removing or replacing existing elements and/or adding new elements in place.
// array.splice(index, deleteCount, item1, ....., itemX)

numbers.splice(1, 0, 'Hej'); // [1, 'Hej', 2, 3, 4, 5] (insert at index 1)
numbers.splice(4, 1, 'Hej'); // [1, 2, 3, 4, 'Hej'] (replace at index 4)
numbers.splice(2, 2, 'Hej', 'Hallå'); // [1, 2, 'Hej', 'Hallå', 5]
```
