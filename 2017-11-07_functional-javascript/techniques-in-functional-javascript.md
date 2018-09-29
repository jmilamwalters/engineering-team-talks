---
<!-- title: Techniques in Functional JavaScript -->
separator: \n---\n
verticalSeparator: \n----\n
theme: moon
center: false
revealOptions:
    transition: 'fade'
---

# Techniques in Functional JavaScript
### _With code samples_

---

## Summary

We will review:

- Partial function application <!-- .element: class="fragment" data-fragment-index="1" -->
- Currying <!-- .element: class="fragment" data-fragment-index="2" -->
- Function composition <!-- .element: class="fragment" data-fragment-index="3" -->

---

## Why functional?

Programming in the functional paradigm emphasizes:

- Fewer lines of code <!-- .element: class="fragment" data-fragment-index="1" -->
- Fewer side effects <!-- .element: class="fragment" data-fragment-index="2" -->
- Code that is easier to reason about and easier to test (e.g., referential transparency) <!-- .element: class="fragment" data-fragment-index="3" -->
- Code that is reuseable and composable <!-- .element: class="fragment" data-fragment-index="4" -->

Note:
- Not necessarily true, but very often the case

---

## Partial application

> In computer science, _**partial application**_ (or _**partial function application)**_ refers to the process of fixing a number of arguments to a function, producing another function of smaller arity.

&mdash; Stefan Oestreicher, ["Currying & Partial Application"](https://medium.com/javascript-inside/currying-partial-application-f1365d5fad3f)

----

### Partial application is:

- The process of applying a function to some of its arguments; whereupon: <!-- .element: class="fragment" data-fragment-index="1" -->
- The partially applied function gets returned for later use. <!-- .element: class="fragment" data-fragment-index="2" -->

&mdash; Eric Elliott, ["Currying or Partial Application"](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8) 

----

### In other words, a partially applied function is:

- A function that takes a function with multiple parameters and returns a function with fewer parameters.

&mdash; Eric Elliott, ["Currying or Partial Application"](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8)

----

## How about an arbitrary example?

----

### Here's a general add function

```javascript
function sum(a, b) {
  return a + b
}
```

----

## Let's rewrite it as a more specific function generator

----

```javascript
const consSum = (a) => (b) => a + b

// Partially apply function `consSum`
const addOne = consSum(1)

addOne(2)   // 3
addOne(3)   // 4 
```

- We've managed to pre-load `consSum` with its first argument, enabling reuse of `addOne` with other arguments when called

---

## Currying

> In mathematics and computer science, _**currying**_ is the technique of translating the evaluation of a function that takes multiple arguments (or a tuple of arguments) into evaluating a sequence of functions, each with a single argument (partial application).

&mdash; Stefan Oestreicher, ["Currying & Partial Application"](https://medium.com/javascript-inside/currying-partial-application-f1365d5fad3f)

----

### So, when compared with partial application, currying is:

- A function that takes a function with multiple parameters as input and returns a function with exactly one parameter.

&mdash; Eric Elliott, ["Currying or Partial Application"](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8)


Note:
- Whereas currying is a format by which we might compose our functions
- Partial application is a means by which we might _use_ curried functions

----

## How about an arbitrary example?

----

### Consider a logging function that takes three arguments:

```javascript
log(status, context, message)
```

----

### Were we to manually write a curried version, it might look like this:

```javascript
const log = (status) => (context) => (message) => {
  // Fancy logging code here.
}
```

----

### Or, more traditionally:

```javascript
function log(status) {
  return function(context) {
    return function(message) {
      // Fancy logging code here.
    }
  }
}
```

----

### Usage:

```javascript
const log = (status) => (context) => (message) => {...}

// Create a new function `info` by fixing first argument
const info = log('info')

info('db-context')('message')

// Use the original function directly with all arguments
log('other level')('some context')('a message')

// Create a new function again
const dbInfo = info('db')

dbInfo('something happened')
```

----

### By way of a `curry` method, however:

- We can automate this manual process, albeit without needing to invoke strange syntax (e.g., `(a)(b)(c)`) in cases where we supply more than one argument at once.

----

```javascript
import {curry} from './fp'

const log = curry((status, context, message) => {
  // fancy logging code here
})

// Use directly with all args
log('status', 'context', 'message')

// Create a new function
const info = log('info')

info('foo-context', 'foo-message')

// Create a new function again
const apiInfo = info('api')

apiInfo('api message')

// We can also fix more than one arg at once
const dbWarn = log('warn', 'db')

dbWarn('something terrible happened')
```

---

## So, why might these be so necessary?

----

## Function composition

----

### Composition

- Function composition enables us to easily connect a number of functions together to build more complex functions.

----

### Composition

- The most common use-case for that kind of lifting is function composition, e.g., `c(x) = f(g(x))`. 
- Function composition takes the return value of one function and feeds it in as an argument to another function. 
- Since a function can only return one value, the function being applied to the return value must be unary.

&mdash; Eric Elliott, ["Currying or Partial Application"](https://medium.com/javascript-scene/curry-or-partial-application-8150044c78b8)

---

## So, how about some utility functions?

----

## Compose

- Compose matches up more naturally with how functions are written. Using the same data as defined for the flow function:

```javascript
const compose = () => {
  const funcs = Array.protoype.slice.call(arguments)
  return funcs.reduce((f, g) => () => f(g.apply(this, arguments)))
}
```

----

## Compose


From:

```javascript
const getFinalPrice = tax(discount(getPrice(x)))
```

To:

```javascript
const getFinalPrice = compose(tax, discount, getPrice)
map(products, getFinalPrice) // [9.675, 4.8375, 0.9675]
```

----

## Partial

- Partially apply a function by filling in any number of its arguments.

```javascript
const partial = (fn, ...args) => (...newArgs) => fn(...args, ...newArgs)
```

----

## Partial

### Example usage

```javascript
const add = (x,y) => x + y
const add5to = partial(add, 5)

add5to(10) // 15
```

---

## Use case: 
### Create histogram from vehicular data

Note:
- Interview with AutoFi

----

## From:

```javascript
  {
    accountId: 'ricartpreowned',
    bodyStyle: 'SUV',
    certified: 'true',
    cityFuelEfficiency: '19',                  // <- City fuel efficiency
    driveLine: 'FWD',
    engine: 'V-6 cyl',
    exteriorColor: 'Crystal Black Pearl',
    features: [],
    fuelType: 'Premium Unleaded',
    highwayFuelEfficiency: '27',              // <- Highway fuel efficiency
    interiorColor: 'Graystone',
    internetPrice: '36500',
    inventoryType: 'all',
    make: 'Acura',
    model: 'MDX',
    modelCode: null,
    modelYear: '2016',
    msrp: '39975',
    newOrUsed: 'used',
    odometer: '15683',
    optionCodes: null,
    packages: [],
    status: 'Live',
    stockNumber: 'PRT26599',
    transmission: '9-Speed Automatic',
    trim: '3.5L',
    uuid: '76e6301d0a0e0ae757ee4a8214e64e3e',
    vin: '5FRYD3H27GB003986',
  },
  {
    accountId: 'ricartpreowned',
    bodyStyle: 'SUV',
    certified: 'true',
    cityFuelEfficiency: '19',
    driveLine: 'AWD',
    engine: 'V-6 cyl',
    exteriorColor: 'Kona Coffee',
    features: [],
    fuelType: 'Premium Unleaded',
    highwayFuelEfficiency: '27',
    interiorColor: 'Parchment',
    internetPrice: '23200',
    inventoryType: 'all',
    make: 'Acura',
    model: 'RDX',
    modelCode: 'TB4H3FJNW',
    modelYear: '2015',
    msrp: '26975',
    newOrUsed: 'used',
    odometer: '41042',
    optionCodes: null,
    packages: [],
    status: 'Live',
    stockNumber: 'PRT26325',
    transmission: '6-Speed Automatic',
    trim: 'Base',
    uuid: 'd39bb6200a0e0ae965f9dd8451a9d3c0',
    vin: '5J8TB4H34FL014179',
  },
```

----

## We need to abstract:

### City fuel efficiency

```javascript
{
  "0,4": 0,
  "5,9": 0,
  "10,14": 0,
  "15,19": 7,
  "20,24": 8,
  "25,29": 1,
  "30,34": 0,
  "35,39": 0,
  "40,44": 0,
  "45,49": 0
}
```

----

## As well as:

### Highway fuel efficiency

```javascript
{
  "0,4": 0,
  "5,9": 0,
  "10,14": 0,
  "15,19": 7,
  "20,24": 8,
  "25,29": 1,
  "30,34": 0,
  "35,39": 0,
  "40,44": 0,
  "45,49": 0
}
```

----

### An implementation

```javascript
/*
 * Sum.
 * sum :: (Number, Number) → Number
 *
 * Provided two numbers, return their summation.
 **/
const sum = (numX, numY) => numX + numY

/*
 * Increment.
 * incr :: Number → Number
 *
 * Provided a number, return its value incremented by one.
 **/
const incr = num => sum(num, 1)

/*
 * Construct bins.
 * consBins :: Object → [[Number]]
 *
 * Configuration parameters:
 * min - The numerical floor from which to initialize the first interval.
 * max - The numerical ceiling by which we limit the last of intervals.
 * width - The range encompassed by each interval.
 **/
const consBins = ({min = 0, max = 100, width = 1, accum = []}) =>
  // Ensure `max` does not exceed the range of our current interval...
  sum(min, width) > max
    ? // If so, return our accumulator;
      accum
    : // Otherwise, call upon our method recursively until we fulfill our
      // predicating condition.
      consBins({
        // Increment interval, so as to preclude overlap.
        min: incr(sum(min, width)),
        // `max` and `width` remain constant.
        max,
        width,
        // Wax upon our accumulator, combining it with a new array, in which the
        // current interval is represented.
        accum: accum.concat([[min, sum(min, width)]]),
      })

/*
 * Filter property by minimum and maximum values.
 * filterPropByMinMax :: ([Object], Object) → Array
 *
 * From an array of objects, return only those cases where the given object
 * property falls within range of specified minimum and maximum values.
 *
 * Configuration parameters:
 * {String} key - The property to -- and by which to -- filter.
 * {String} min - The minimum value by which to filter our attribute.
 * {String} min - The maximum value by which to filter our attribute.
 **/
const filterPropByMinMax = (arr, {key = '', min = '0', max = '100'}) =>
  arr.filter(
    // The numerical values encompassed by our JSON list assume the form of
    // strings; hence, we need abstract their numerical values via `parseInt`.
    // Our callback will return `true` provided the item's value is greater than
    // or equal to our minimum _and_ less than or equal to our maximum.
    item => parseInt(item[key], 10) >= min && parseInt(item[key], 10) <= max
  )

/*
 * Construct histogram object.
 * consHistObj :: (Array, [[Number]]) → String → Object
 **/
const consHistObj = (arr = [], bins = []) => (key = '') =>
  // For each of the nested arrays within our `bins` list...
  bins.reduce(
    (accum, currVal) => ({
      // Build upon our accumulator, which we initialize as an empty object...
      ...accum,
      // And where each entry is an object, whereof the the key shall represent
      // the minimum and maximum values encompassed by each bin (e.g., `15,19`),
      // and whereof the value shall delineate all vehicles encompassed by such
      // bin. We retrieve such values by of `filterPropByMinMax`...
      [currVal]: filterPropByMinMax(arr, {
        key,
        min: currVal[0],
        max: currVal[1],
        // And we calculate a total by way of `Array.prototype.length`.
      }).length,
    }),
    {}
  )


/*
 * Construct histogram object provided bins.
 * consHistObjWithBins :: (Object → [Object] → String) → Object
 *
 * Configuration parameters:
 * {Number} min - The minimum value by which to filter our attribute.
 * {Number} min - The maximum value by which to filter our attribute.
 * {Number} width - The range encompassed by each interval.
 **/
const consHistObjWithBins = (arr = [], {min = 0, max = 100, width = 1}) => (
  key = ''
) => consHistObj(arr, consBins({min, max, width}))(key)

/*
 * Construct mileage histogram.
 *
 * NB: `getData()` is the function we call, so as to pull in the massive piece
 * of JSON we need process.
 **/
const consMileageHist = (key = '') =>
  consHistObjWithBins(getData(), {min: 0, max: 50, width: 4})(key)

const consCityMileageHist = () => consMileageHist('cityFuelEfficiency')
const consHighwayMileageHist = () => consMileageHist('highwayFuelEfficiency')

console.log('City mileage', consCityMileageHist())
console.log('Highway mileage', consHighwayMileageHist())
```

---

## Use case: eliminate `switch`

----

Though useful, `switch` statements in traditional usage are not:

- Immutable
- Composable

----

And, sometimes, they:

- Lend to side effects
- Use `break`

----

### Can we develop a functional alternative for `switch` statements? Namely, one that is:

- Pure
- Immutable
- Composable

Note:
- Disclaimer: our function alternative will _not_ accommodate fall-through

----

### A functional variant

```javascript
const switchCase = cases => defaultCase => key =>
  key in cases ? cases[key] : defaultCase

const execIfFunc = f =>
  typeof f === 'function' ? f() : f

const switchCaseFunc = cases => defaultCase => key =>
  execIfFunc(switchCase(cases)(defaultCase)(key))
```

----

### Provided such utilities, we can go from:

```javascript
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}
```

----

### To:

```javascript
const counter = (state = 0, action) =>
  switchCaseFunc({
    'INCREMENT': () => state + 1,
    'DECREMENT': () => state - 1,
  })(state)(action.type)
```

----

### We can even do things like:

```javascript
const counter = (state = 0, action) =>
  switchCaseFunc({
    'RESET': 0,
    'INCREMENT': () => state + 1,
    'DECREMENT': () => state - 1,
  })(state)(action.type)
```

----

## Let's try another example.

----

### Here's a common use of `switch`

```javascript
let day

switch (new Date().getDay()) {
  case 0:
    day = 'Sunday'
    break
  case 1:
    day = 'Monday'
    break
  case 2:
    day = 'Tuesday'
    break
  case 3:
    day = 'Wednesday'
    break
  case 4:
    day = 'Thursday'
    break
  case 5:
    day = 'Friday'
    break
  case 6:
    day = 'Saturday'
  }
```

----

### How about this for an alternative?

```javascript
const getDay = switchCase({
    0: 'Sunday',
    1: 'Monday',
    2: 'Tuesday',
    3: 'Wednesday',
    4: 'Thursday',
    5: 'Friday',
    6: 'Saturday'
  })('Unknown')
```

----

## Perhaps our functional alternative is terser, but is that all?

----

## Why, no!

----

### Because we:

- Curried switchCase... <!-- .element: class="fragment" data-fragment-index="1" -->
- We can decompose switchCase unto multiple reusable methods. <!-- .element: class="fragment" data-fragment-index="2" -->

----

### Which enables usage like:

```javascript
const getCurrentDay = () => getDay(new Date().getDay())
console.log(getCurrentDay())
// 'Tuesday'
```

----

## We can use `switchCase` programmatically.

----


```javascript
const days = [
  'Sunday',
  'Monday',
  'Tuesday',
  'Wednesday',
  'Thursday',
  'Friday',
  'Saturday',
]

// Remember: the comma operator evaluates both expressions and returns the
// result of the second expression.
const getDay = switchCase(
  days.reduce((acc, value, index) => ((acc[index] = value), acc), {})
)("I don't know what day it is.")
```

----

### Or, how about:

```javascript
const printDay = switchCase(
  days.reduce((acc, value) => ((acc[value] = `The day is ${value}.`), acc), {})
)("I don't know what day it is.")
```

---

## Libraries 

- _**Ramda -**_ A practical functional library for JavaScript programmers: http://ramdajs.com/
- _**Lodash FP -**_ The lodash/fp module promotes a more functional programming (FP) friendly style by exporting an instance of lodash with its methods wrapped to produce immutable auto-curried iteratee-first data-last methods: https://github.com/lodash/lodash/wiki/FP-Guide

---

## Further reading

- Michael Fogus, _Functional JavaScript_
- Dr. Boolean, _Professor Frisby's Mostly Adequate Guide to Functional Programming,_ https://www.gitbook.com/book/drboolean/mostly-adequate-guide/details
- Eric Elliott, https://medium.com/@_ericelliott

---

## Created using
# Reveal-MD