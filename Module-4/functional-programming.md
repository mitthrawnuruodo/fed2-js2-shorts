# Functional Programming

## 1. What is Functional Programming?

**Functional Programming** (FP) is a programming paradigm — a way of thinking about and organizing code — that treats **functions as the main building blocks** of programs.

Instead of changing state and modifying objects (as in OOP), functional programming focuses on data transformation: you take an input, process it through a series of pure functions, and produce an output.

In FP, **functions are first-class citizens**, meaning:
* You can **store functions in variables**
* You can **pass them as arguments** to other functions
* You can **return functions** from functions

This allows for **declarative**, **composable**, and **predictable** code — focusing on what to do rather than how to do it.

## 2. Functional Programming vs Object-Oriented Programming

<small style="font-size: 1.55rem">

| Aspect | Object-Oriented Programming (OOP) | Functional Programming (FP) |
| ------ | --------------------------------- | --------------------------- |
| **Core idea** | Model real-world entities using objects and classes | Model computation as evaluation of pure functions |
| **Main units** | Objects (with data + behavior) | Functions (that transform data) |
| **State** | Often [mutable](https://developer.mozilla.org/en-US/docs/Glossary/Mutable) (object properties change) | [Immutable](https://developer.mozilla.org/en-US/docs/Glossary/Immutable) (data never changes; new data returned) |
| **Approach** | Imperative – describe *how* to perform tasks | Declarative – describe *what* to do |
| **Example thinking** | “A `Car` object has methods like `start()` and `stop()`.” | “A `drive` function takes a car state and returns a new one.” |
| **Focus** | Structure and relationships | Data flow and transformation |


## 3. A Simple Comparison Example
**Object-Oriented Style**

<small style="font-size: 1.4rem">

```js
class Counter {
  constructor() {
    this.value = 0;
  }

  increment() {
    this.value++;
  }

  getValue() {
    return this.value;
  }
}

const counter = new Counter();
counter.increment();
console.log(counter.getValue()); // 1
```


Here we store state inside the object and change it over time.

**Functional Style**

```js
const increment = (value) => value + 1;

let count = 0;
count = increment(count);
console.log(count); // 1
```

Here we don’t change existing state — we return a new value instead.

## 4. How FP and OOP Can Work Together
Although FP and OOP are often presented as opposites, **modern JavaScript allows mixing both styles**.

You might use objects to organize data and functions to perform transformations.

For example:
* React uses **functional components** (FP) that return UI elements (objects).
* Classes in JS can contain **pure utility methods** (FP style) to process data.

The key is not to choose one over the other, but to understand when each is most effective.

## Summary
| Concept | OOP | FP |
| ------- | --- | -- |
| Building block | **Objects** | **Functions** |
| State | Mutable | Immutable |
| Side effects | Common | Avoided |
| Emphasis | Structure | Transformation |
| JavaScript&nbsp;support | `class`, `this`, `new` | arrow functions, higher-order functions, <br>like `map`/`filter`/`reduce` |

## Declarative Programming

### 1. Declarative vs Imperative (the broader idea)
Declarative and imperative describe how we express what the program should do:
| Paradigm | Focus | Example |
| -------- | ----- | ------- |
| **Imperative**  | ***How*** to do something, step by step | “Loop over this array, add each number to a sum.” |
| **Declarative** | ***What*** the end result should be     | “Sum all the numbers in this array.” |

**In other words:**
* **Imperative** = Tell the computer every step.
* **Declarative** = Describe the outcome; let the computer handle the steps.

### 2. Where Functional Programming fits in
**Functional programming** is one of several declarative paradigms, along with others like:
* Logic programming (e.g. Prolog)
* Reactive programming (e.g. RxJS streams)
* Constraint-based programming

FP is declarative because it focuses on **expressing transformations** rather than *mutating state* or *controlling flow manually*.

Example comparison:
**Imperative**
```js
let total = 0;
for (let i = 0; i < numbers.length; i++) {
  total += numbers[i];
}
```
**Functional (Declarative)**
```js
const total = numbers.reduce((sum, n) => sum + n, 0);
```
You’re declaring *what* should happen (“reduce this list by summing its elements”), 
not *how* (loop, index, accumulate).

### 3. Relationship summary

| Concept                     | Category                  | Example                                        |
| --------------------------- | ------------------------- | ---------------------------------------------- |
| **Declarative Programming** | Big umbrella term         | SQL, HTML, Functional Programming              |
| **Functional Programming**  | A *subset* of declarative | JavaScript (`map`, `filter`, `reduce`), Haskell, Elm |
| **Imperative Programming**  | The opposite approach of declarative | Traditional JS loops, C-style code             |

### 4. Analogy
Think of it like this:
* **Declarative programming** is the *style* 
(*“describe, don’t instruct”*)
  * **Functional programming** is a *specific discipline* within that style 
(*“use pure functions and immutability to describe transformations”*)
* **Imperative programming** is the opposite style
(*“Give the computer detailed instructions, step by step”*)

**In short:**
* All functional programming is declarative,
* but not all declarative programming is functional.

## 3 Main Principles of Functional Programming
Functional programming is built around three main ideas:
1. **Immutability**
2. **Separation of Data and Functions**
3. **First-Class Functions**

### 1. Immutability — “Don’t change data, create new data.”
In functional programming, **data is never modified** after it’s created.

Instead of changing an existing value, you create a **new version** of it.

This makes code:
* **Predictable** – no hidden changes to shared data
* **Easier to debug** – data never “mysteriously” changes elsewhere
* **Safe for concurrency** – no race conditions from changing state

Example
```js
// Imperative (mutable)
let user = { name: "Lasse", age: 55 };
user.age = 56; // directly changes the object

// Functional (immutable)
const user = { name: "Lasse", age: 55 };
const updatedUser = { ...user, age: 56 }; // new object created
```
Here we don’t touch the original user — we make a copy with the change applied.

### 2. Separation of Data and Functions — “Keep logic and data independent.”
In OOP, **data and behavior** live together (inside objects).

In FP, we separate them: functions take data as input and return new data — they don’t belong to the data.

This makes code:
* **Reusable** – one function can handle many kinds of data
* **Easier to test** – no hidden dependencies or internal state
* **Composable** – small pure functions can be chained together

Example
```js
// OOP-style
const user = {
  name: "Lasse",
  greet() {
    return `Hello, ${this.name}!`;
  },
};

console.log(user.greet());

// FP-style
const greet = (name) => `Hello, ${name}!`;
console.log(greet("Lasse"));
```

In FP, `greet` doesn’t belong to any object — it simply **takes input and returns output**.
It can work with *any* data, not just a specific `user`.

### 3. First-Class Functions — “Functions are just values.”
In JavaScript (and FP in general), functions are **first-class citizens**, meaning you can:
* Assign them to variables
* Pass them as arguments to other functions
* Return them from other functions

This allows **higher-order functions** — functions that operate *on* other functions — which is key to expressive, composable code.

Example:
```js
// Store a function in a variable
const double = (x) => x * 2;

// Pass a function as argument
const numbers = [1, 2, 3];
const doubled = numbers.map(double);
console.log(doubled); // [2, 4, 6]

// Return a function from another function
const multiplier = (n) => (x) => x * n;
const triple = multiplier(3);
console.log(triple(5)); // 15
```

Functions here behave like **data** — you can store, combine, and reuse them freely.

### Summary Table
| Principle                            | Description                                     | Benefit                          |
| ------------------------------------ | ----------------------------------------------- | -------------------------------- |
| **Immutability**                     | Never change existing data; create new data     | Predictable and safe code        |
| **Separation of Data and Functions** | Logic operates *on* data,<br>not *inside* it       | Reusable and testable code       |
| **First-Class Functions**            | Functions can be passed, returned, and composed | Flexible and expressive programs |

## Closures — Functions that “remember”

A closure is one of the most important — and sometimes most misunderstood — concepts in JavaScript.

**In simple terms:**
A closure happens when an inner function “remembers” the variables from the function where it was created — even after that outer function has finished running.

### Basic Example
```js
function makeCounter() {
  let count = 0; // a local variable

  return function () {
    count++;
    return count;
  };
}

const counter = makeCounter();

console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

[Codepen](https://codepen.io/xiaolasse/pen/jEWzrqL?editors=0012)

#### What happens here?
1. When you call `makeCounter()`, a new `count` variable is created inside it.
2. `makeCounter()` returns the `inner function` (which increases `count`), but doesn’t destroy the variable.
3. The returned function still has **access to `count`**, even though `makeCounter()` has finished running.
4. That’s a **closure** — the function “closes over” its surrounding variables and keeps them alive.

#### Think of it like…


A **backpack** — Every function carries a small “backpack” with it, containing the variables it needs from where it was created. Even when it’s used later, somewhere else in your code, it still has access to those values in its backpack.


### Why Closures Matter in Functional Programming
Closures allow functions to:
* **Maintain private state** without using objects or classes
* **Create configurable functions** that “remember” some settings
* **Build powerful higher-order functions**

<small style="font-size: 1.25rem">

**Flashback**: That last part of the example from 3. First-Class Functions, is **actually a closure**:
```js
const multiplier = (n) => (x) => x * n;
const triple = multiplier(3);
console.log(triple(5)); // 15
```

Here’s what’s happening:
* `multiplier(3)` returns a `function: (x) => x * n`
* Even though `n` is no longer in scope after `multiplier` has finished running...
* ...the returned function **remembers** `n` — in this case, `3`

That “remembering” is exactly what a closure is.

You can test that it's not just hardcoded:
```js
const timesFour = multiplier(4);
console.log(timesFour(2)); // 8

const timesTen = multiplier(10);
console.log(timesTen(1)); // 10
```
Each returned function keeps its own copy of `n`, thanks to closure.


<!--
_footer: "[Codepen](https://codepen.io/xiaolasse/pen/dPGmXVy?editors=0012)"
-->

### Example — Function Factory

```js
function makeMultiplier(factor) {
  return function (number) {
    return number * factor;
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

<small style="font-size: 1.55rem">

Each returned function **remembers** its own `factor` — even though `makeMultiplier` has long finished.
This is functional programming in action: **functions creating new functions with remembered data**.


<!--
_footer: "[Codepen](https://codepen.io/xiaolasse/pen/KwVoMMX?editors=0012) - <small>PS! Did you notice that this example is identical to the one above (difference is that this use the `function` keyword, the above is an arrow function?)</small>"
-->

### Example: Day Planner (Closure with Two Functions)
We’ll make a function that helps you **store a daily task** — and gives you two functions:
* One to **get** today’s task
* One to **change** it

The task is **stored in a closure**, so it’s private — not accessible directly.

[Codepen](https://codepen.io/xiaolasse/pen/LEGdZZv?editors=0012)

```js
function createPlanner(initialTask) {
  let task = initialTask;

  function getTask() {
    return `Today’s task: ${task}`;
  }

  function setTask(newTask) {
    task = newTask;
  }

  return { getTask, setTask };
}
// Usage
const monday = createPlanner("Write lesson plan");
console.log(monday.getTask()); // "Today’s task: Write lesson plan"

monday.setTask("Review code examples");
console.log(monday.getTask()); // "Today’s task: Review code examples"
```

##### Why this works
* `createPlanner("...")` creates a **closure** with a private `task` variable.
* `getTask()` and `setTask()` both share access to that task, even **after `createPlanner()` has returned**.
* You can **create multiple planners**, each with their own private task:

```js
const tuesday = createPlanner("Finish documentation");

console.log(tuesday.getTask()); // "Today’s task: Finish documentation"

// Change Tuesday’s task
tuesday.setTask("Team meeting at 14:00");

console.log(tuesday.getTask()); // "Today’s task: Team meeting at 14:00"

// Monday’s planner is unaffected:
console.log(monday.getTask()); // Still: "Today’s task: Review code examples"
```

#### Example: Bank Account (Closure with Two Functions)

```js
function createBankAccount(initialBalance) {
  let balance = initialBalance;

  function deposit(amount) {
    balance += amount;
    return `Deposited $${amount}. New balance: $${balance}`;
  }

  function withdraw(amount) {
    if (amount <= balance) {
      balance -= amount;
      return `Withdrew $${amount}. New balance: $${balance}`;
    } else {
      return `Insufficient funds. Current balance: $${balance}`;
    }
  }

  // Return both functions so they share the same closure
  return { deposit, withdraw };
}

const myAccount = createBankAccount(100);

console.log(myAccount.deposit(50));   // "Deposited $50. New balance: $150"
console.log(myAccount.withdraw(30));  // "Withdrew $30. New balance: $120"
console.log(myAccount.withdraw(200)); // "Insufficient funds. Current balance: $120"
```

##### How this closure works:

* `createBankAccount` creates a local `balance` variable initialized with the starting amount
* It defines **two functions** that both have access to the same `balance` variable
    * `deposit` - adds to the balance
    * `withdraw` - subtracts from the balance (with a check)
* Both functions "close over" the `balance` variable, maintaining their own private copy that persists between calls
* Each call to `createBankAccount` creates a separate closure with its own isolated balance

[Codepen](https://codepen.io/xiaolasse/pen/NPxYrbj?editors=0012)

### Summary
| Concept             | Description                                                     | Example                                  |
| ------------------- | --------------------------------------------------------------- | ---------------------------------------- |
| **Closure**         | A function that remembers variables from its creation context   | Inner function using `count` or `factor` |
| **Why it’s useful** | Lets functions keep private data and customize behavior         | Counters, factories, event handlers      |
| **Functional link** | Enables functions to “carry state” without mutating global data | Pure, reusable, safe                     |

## 2. Higher-Order Functions (HOFs)


A **higher-order function** is any function that:
* Takes one or more functions as arguments, or
* Returns a function as its result.

Example:
```js
function repeat(fn, times) {
  for (let i = 0; i < times; i++) {
    fn();
  }
}

repeat(() => console.log("Hello!"), 3);
// → prints "Hello!" three times
```

Here, `repeat()` is a **higher-order function** because it takes another function as an argument.</small>



### 2b. Callback functions

A **callback function** is a function that you **pass as an argument** to another function, so that it can be **“called back” later** inside that function.

A **higher-order function** (HOF) is the one that receives (or returns) such a function.


**Example:**
```js
function greet(name, callback) {
  console.log(`Hello, ${name}!`);
  callback(); // <-- calling the callback
}

function afterGreeting() {
  console.log("Nice to meet you!");
}

greet("Lasse", afterGreeting);
```

**Here:**
* `greet` → **higher-order function** (because it takes another function as input)
* `afterGreeting` → **callback function** (because it’s passed in and called back later)

So, **callbacks are the functions you pass**, while **HOFs are the functions that use them**.


## 3. Working with Arrays Functionally
JavaScript arrays have several **built-in higher-order methods** that let you process and transform data without mutating it.

Let's re-visit three of these: 
* `.map()` – Transform Each Item
* `.filter()` – Select Certain Items
* `.reduce()` – Combine Items into One Value


### `.map()` – Transform Each Item
**Purpose**: Creates a new array by applying a function to each element.
```js
const numbers = [1, 2, 3, 4];

// double each number
const doubled = numbers.map(num => num * 2);

console.log(doubled); // [2, 4, 6, 8]
```
* Takes a callback function (`num => num * 2`)
* Returns a new array
* Does not modify the original numbers


### `.filter()` – Select Certain Items
**Purpose**: Creates a new array containing only elements that pass a test.
```js
const ages = [12, 25, 17, 30, 16];

const adults = ages.filter(age => age >= 18);

console.log(adults); // [25, 30]
```
* The callback returns `true` or `false`
* Keeps only values where the callback returns `true`
* Original array is unchanged


### `.reduce()` – Combine Items into One Value
**Purpose**: Reduces an array to a single value (number, string, object, etc.).
```js
const prices = [10, 20, 30];

const total = prices.reduce((sum, price) => sum + price, 0);

console.log(total); // 60
```
* The callback receives an **accumulator** (`sum`) and the **current value** (`price`).
* The `0` is the initial value.
* Returns one result — often used for sums, counts, or building new objects.


#### Example with Objects

<small style="font-size: 1.4rem">

You can also use these methods on arrays of objects — very common in real projects.
```js
const products = [
  { name: "Book", price: 10 },
  { name: "Pen", price: 2 },
  { name: "Bag", price: 25 },
];

// Filter, then map, then reduce:
const totalExpensive = products
  .filter(p => p.price > 5)       // keep products over 5
  .map(p => p.price * 1.25)       // add 25% tax
  .reduce((sum, price) => sum + price, 0); // sum up

console.log(totalExpensive); // 43.75
```

Each step:
1. `.filter()` selects items,
2. `.map()` transforms them,
3. `.reduce()` combines them — showing function composition in action.



#### Chaining

<small style="font-size: 1.65rem">

In the example above, we’re **chaining multiple higher-order functions** — each method (`filter`, `map`, `reduce`) is itself a **HOF**, and we combine them in sequence so the output of one becomes the input of the next:
```js
const totalExpensive = products
  .filter(p => p.price > 5)       // HOF #1 → filters items
  .map(p => p.price * 1.25)       // HOF #2 → transforms items
  .reduce((sum, price) => sum + price, 0); // HOF #3 → combines items
```

This is called **function chaining** (or **method chaining**) and it’s a very common **functional pattern** in JavaScript:
* Each higher-order function returns a new array (or value),
* Which allows the next one to run immediately,
* Producing clean, readable, and declarative code.



## 4. Other Useful Functional Array Methods



| Method       | Description                                          | Example                            |
| ------------ | ---------------------------------------------------- | ---------------------------------- |
| [`.forEach()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach) | Runs a function for each element (side-effects only) | <span style="white-space: nowrap;">`arr.forEach(x => console.log(x))`</span> |
| [`.find()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find)    | Returns the first matching element                   | `users.find(u => u.id === 2)`      |
| [`.some()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some)    | Returns `true` if *any* match                        | `arr.some(x => x > 10)`            |
| [`.every()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every)   | Returns `true` if *all* match                        | `arr.every(x => x > 0)`            |
| [`.flatMap()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap) | Maps and flattens in one step                        | `arr.flatMap(x => x.items)`        |

We'll look at a few of these, and some others in a moment.


## 5. Summary
| Concept                         | Meaning                                                                    |
| ------------------------------- | -------------------------------------------------------------------------- |
| **Higher-order function (HOF)** | Takes one or more functions as arguments, or returns a function            |
| **Callback function**           | A function passed *as an argument* to another function, to be called later |
| **Pure function**               | Always gives the same output for the same input, with no side effects      |
| **Immutability**                | Do not change (mutate) the original data                                   |
| **Composition**                 | Combine small, focused functions into bigger ones                          |
| **`map` / `filter` / `reduce`**       | Common higher-order functions for transforming arrays                      |


## 6. Other Common Higher-Order Functions
All of the following methods also **take callback functions**** — that’s what makes them **higher-order**.

<!--_footer: "** There's an exception, we'll get back to that..."-->

* `.every()` – Check if *all* elements pass a test
* `.some()` – Check if *any* element passes a test
* `.slice()` – Copy a Portion of an Array**
* `.sort()` – Order Elements in an Array


### `.every()` – Check if *all* elements pass a test
**Purpose**: Returns true if every element in the array satisfies the condition.
```js
const scores = [80, 90, 75, 88];

const allAbove70 = scores.every(score => score >= 70);

console.log(allAbove70); // true
```
**Explanation:**
* The callback `(score => score >= 70)` runs for each item.
* If all return `true`, `.every()` returns `true`.
* If any return `false`, it stops early and returns `false`.


### `.some()` – Check if *any* element passes a test
**Purpose**: Returns `true` if **at least one** element satisfies the condition.
```js
const ages = [12, 16, 18, 21];

const hasAdult = ages.some(age => age >= 18);

console.log(hasAdult); // true
```
**Explanation:**
* The callback runs for each item until it finds one that’s `true`.
* Returns `true` immediately if a match is found.
* Returns `false` is no match is found.

**Tip**: Think of `.every()` as “AND” logic, and `.some()` as “OR” logic.


### `.slice()` – Copy a Portion of an Array


**Purpose**: Creates a new array with selected elements (non-mutating).
```js
const fruits = ["apple", "banana", "cherry", "date"];

const someFruits = fruits.slice(1, 3);

console.log(someFruits); // ["banana", "cherry"]
console.log(fruits);     // ["apple", "banana", "cherry", "date"]
```
**Explanation:**
* Takes `start` and `end` indices (`end` not included).
* Returns a **shallow copy** of that segment.
* Doesn’t modify the original — fits perfectly with FP principles.

***Note**: `.slice()` doesn’t use a callback — so it’s not a HOF by definition, but it’s often mentioned together with FP-friendly array methods because it’s pure and non-mutating.*



### `.sort()` – Order Elements in an Array

<small style="font-size: 1.55rem">

**Purpose**: Sorts elements **in place** — but can be made functional by copying first.
```js
const numbers = [5, 12, 3, 9];

const sorted = [...numbers].sort((a, b) => a - b);

console.log(sorted);  // [3, 5, 9, 12]
console.log(numbers); // [5, 12, 3, 9] — unchanged
```
**Explanation:**
* The callback `(a, b)` compares two elements.
* Return value determines order:
    * `< 0` → `a` before `b`
    * `> 0` → `a` after `b`
    * `0` → keep same order
* The spread `[...numbers]` makes a copy first to preserve immutability.



### Summary
| Method     | Type          | Returns | Mutates?                     | Example Use                |
| ---------- | ------------- | ------- | ---------------------------- | -------------------------- |
| `.every()` | HOF           | Boolean | No                            | “Are all users active?”    |
| `.some()`  | HOF           | Boolean | No                            | “Is there any admin user?” |
| `.slice()` | Pure function | Array   | No                            | Copy part of array         |
| `.sort()`  | HOF           | Array   | Yes <small style="font-size: 1rem">(unless copied first)</small> | Sort numbers or strings    |


### Sidenote: The new method: `toSorted()`
The standard mutable method:
```js
const numbers = [3, 1, 4];
numbers.sort((a,b) => a - b);
// numbers is now mutated (sorted)
```

The modern immutable alternative:
```js
const numbers = [3, 1, 4];
const sorted = numbers.toSorted((a,b) => a - b);
// sorted is the new sorted array
// numbers remains unchanged
```

The `toSorted()` method of Array instances is the copying version of the `sort()` method. It **returns a new array with the elements sorted**. 


#### Why this is important (especially for FP)
* Mutating the original array (as `sort()` does) violates immutability — a core FP principle.
* Using `toSorted()` helps you write **pure functions**: no hidden side-effects on your inputs.
* It improves predictability, easier debugging, easier reasoning about state changes.
* Prior to `toSorted()`, the common workaround was:
    ```js
    const sorted = [...array].sort(compareFn);
    ```
    Which works, but is a bit more verbose and still a hack. 


#### Example with objects
```js
const products = [
  { name: "Book",     price: 10 },
  { name: "Pen",      price: 2 },
  { name: "Backpack", price: 25 },
];

// Sort by price, but immutably:
const sortedByPrice = products.toSorted((a, b) => a.price - b.price);

console.log(sortedByPrice);
// original `products` array remains in original order
```


#### Quick overview of the new immutable array methods (ES2023+)

| New Method | Immutable version of | Description |
| ---------- | -------------------- | ----------- |
| [`toSorted()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted) | `sort()` | Returns a new sorted array (original untouched) |
| [`toReversed()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toReversed ) | `reverse()` | Returns a new array in reverse order |
| [`toSpliced()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSpliced) | `splice()` | Returns a new array with items added/removed, without changing the original |
| [`with(index, value)`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/with) | direct index assignment | Returns a copy with one element replaced |

All these are **pure, non-mutating** versions — perfect for functional programming.


<!-- _backgroundColor: #ffeaea; -->

#### A few caveats
* `toSorted()` is relatively new: [The MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted) note “Baseline 2023” for support. 
* It returns a *shallow copy*, so if the array contains objects and you mutate an object inside, you’re still mutating the object not the array reference.
* If you’re targeting older environments, you might not have native support yet — fallback to `[...array].sort()` or a [polyfill](https://developer.mozilla.org/en-US/docs/Glossary/Polyfill).

<br>

That said, there is support for `toSorted()`, and above mentioned `toReversed()`, `toSpliced()` and `with()`, in all _modern_ browsers and Node 20+.


## 7. Currying and Partial Application

These two concepts are about breaking down functions and reusing them more flexibly, both very common in Functional Programming (FP).

<small style="display: block; border: 1px solid gray; margin-top: 40px; padding: 10px 30px; font-size: 1rem; background: #def">

### The name “Currying” comes from a person — not curry, the food
It’s named after the American logician and mathematician
**Haskell Brooks Curry (1900 – 1982).**
He worked on the theory of combinatory logic — a mathematical foundation that later influenced functional programming languages like Haskell (which is also named after him).
#### What he formalized
Haskell Curry (and others before him, like Moses Schönfinkel) explored the idea that:
> any function taking multiple arguments can be rewritten as a chain of single-argument functions.

So, a function `f(a, b)` can be expressed as `f(a)(b)`.
That transformation process is what we now call **currying** — in his honor.

</small>

### Currying — “One parameter at a time”

Currying means transforming a function that takes **many arguments** into a chain of functions that each take **one argument**.


**Example (normal function):**
```js
function add(a, b) {
  return a + b;
}

add(2, 3); // 5
```
**Curried version:**
```js
function addCurried(a) {
  return function (b) {
    return a + b;
  };
}

addCurried(2)(3); // 5
```
The first call “remembers” the `a` value, and returns a new function waiting for `b`.


You can also store the first call for reuse:
```js
const addTwo = addCurried(2);
console.log(addTwo(5)); // 7
console.log(addTwo(10)); // 12
```

**Step-by-step:**
1. `addCurried(2)` is **called**
    * The function `addCurried` receives `a = 2`.
    * It **returns a new function** that expects another parameter `b`.
2. **That returned function “remembers” `a = 2`**
    * Thanks to **closures**, the inner function keeps access to variables from its parent scope.
    * So even after `addCurried` has finished running, the value `a = 2` is still stored.
3. We store that returned function in `addTwo`
    * Now, `addTwo` is a function that adds 2 to whatever number you give it.
4. When we call `addTwo(5)` → it runs `a + b` → `2 + 5` = 7
    When we call `addTwo(10)` → `2 + 10` = 12


This lets you **reuse** specialized versions of a function — a key FP idea.


#### In plain words:
* `addCurried(2)` creates a customized function that always adds `2`.
* This is powerful because you can reuse `addTwo` anywhere — it’s a tiny, specialized function built from a more general one.

That’s one of the main ideas in functional programming:
* Build small, reusable, specialized functions from more general ones.


#### Sidenote: Currying and Closures

Currying **relies on closures** to work — every time you call a curried function, it returns another function that **remembers** the earlier arguments through a closure.

So you can think of it like this: **Currying = nesting + closures**

Each inner function “closes over” (keeps access to) the parameters of the outer ones.
That’s how it can use those earlier values later on.

**Example recap:**
```js
function addCurried(a) {
  return function (b) {
    return a + b; // ← closure keeps access to 'a'
  };
}

const addTwo = addCurried(2);
console.log(addTwo(10)); // 12
```
Here, the inner function forms a closure around `a`, allowing the curried structure to “remember” it even after the first function has finished.


### Partial Application — “Some arguments now, the rest later”

<small style="font-size: 1.4rem">

**Partial application** is similar, but slightly looser:
You call a function with **some** of its parameters now, and get back a new function expecting the rest.

**Example:**
```js
function multiply(a, b, c) {
  return a * b * c;
}

// Partially apply the first argument:
function partialMultiply(a) {
  return function (b, c) {
    return multiply(a, b, c);
  };
}

const double = partialMultiply(2);
console.log(double(3, 4)); // 24 (2 * 3 * 4)
```
Here, `partialMultiply(2)` creates a new function where the first argument is fixed — that’s **partial application**.


#### Step-by-step:
1. **`partialMultiply(2)` is called**
    * The outer function gets `a = 2`.
    * It returns a new inner function that still needs `b` and `c`.
2. **The inner function forms a closure**
    * It “remembers” the value of `a` (2), even after `partialMultiply` has finished running.
    * That’s why we can still use `a` later inside `multiply(a, b, c)`.
3. **We store that returned function in `double`**
    * `double` now represents a *specialized* version of `multiply` where the first argument is always `2`.
4. **When we call `double(3, 4)` → it runs `multiply(2, 3, 4)` → returns `24`.**

**In plain words:**
`partialMultiply(2)` **pre-fills** one argument `(a = 2)` and gives you back a reusable function that multiplies any two other numbers by 2.

This is **partial application** — it uses a **closure** to remember preset arguments, letting you create flexible, reusable helper functions.



### Quick Summary

<small style="font-size: 1.45rem">

| Term | What it does | Example |
| ---- | ------------ | ------- |
| **Currying** | Converts a multi-argument function into a chain of single-argument functions | `addCurried(2)(3)`                                |
| **Partial Application** | Pre-fills some arguments and returns a new function that remembers them | `const double = partialMultiply(2);`<br>`double(3, 4)` |


### Practical Example 1: Currying with `map()`

Imagine you want to **add tax** to a list of prices — and the tax rate might change.

Instead of hardcoding it each time:
```js
const prices = [100, 200, 300];
const taxed = prices.map(price => price * 1.25);
```

You can use ***currying*** to make this reusable:

```js
// Curried function: one argument at a time
const addTax = rate => price => price * (1 + rate);

// Create a reusable 25% tax function
const add25Percent = addTax(0.25);

// Use in map
const prices = [100, 200, 300];
const taxed = prices.map(add25Percent);

console.log(taxed); // [125, 250, 375]
```

**Why it’s nice:**
* You can easily reuse `addTax(0.25)` elsewhere.
* It’s *pure* — no side effects, doesn’t modify data.
* It works naturally with higher-order functions like `.map()`.


### Practical Example 2: Partial Application with Event Handlers

Sometimes, you need to pass **extra data** to an event handler.

Instead of wrapping everything in anonymous functions:
```js
button.addEventListener("click", () => handleClick("save"));
```
You can use partial application to pre-fill one argument:


```js
function handleClick(action, event) {
  console.log(`Button clicked: ${action}`);
  console.log(event.type); // e.g. "click"
}

// Create a partially applied version:
const handleSaveClick = handleClick.bind(null, "save");

// Add event listener:
button.addEventListener("click", handleSaveClick);
```

Why it’s nice:
* You “preload” data (`"save"`) into a function.
* Keeps code cleaner — especially in loops or many buttons.

> We'll get back to `bind()` a little later in this lesson.


### How They Fit into Functional Programming

| Concept | Purpose in FP | Everyday Use |
| ------- | ------------- | ------------ |
| **Currying** | Makes functions more composable and reusable; allows building complex behavior from smaller pieces | Create reusable logic for `.map()`, `.filter()`, etc. |
| **Partial Application** | Lets you “preset” some arguments and reuse the function later | Pre-fill parameters for event handlers or utility functions |
| **Both** | Promote **pure**, **reusable**, and **modular** functions — core FP principles | Encourage cleaner, more declarative, and maintainable code  |

## Functions as Objects
In JavaScript, **functions are special kinds of objects**.

That means:
* You can store them in variables
* Pass them around as arguments
* Return them from other functions
  …but also — just like any other object —
  they can have **properties and methods**.

**Example:**
```js
function greet(name) {
  console.log(`Hello, ${name}!`);
}

console.log(typeof greet); // "function"
console.log(greet.name);   // "greet"
console.log(greet.length); // 1 (number of parameters)
```
Under the hood, every function is really an object with extra callable behavior.

### Built-in Function Methods
Every function object automatically has three powerful built-in methods:
* [`call()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call), 
* [`apply()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply), and 
* [`bind()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 

— they all control **how a function is executed** and **what `this` refers to** inside it.

Let’s go through them one by one:

#### 1. `call()`
**Purpose**: Calls a function immediately, but lets you set what `this` refers to.
```js
function introduce() {
  console.log(`Hi, I'm ${this.name}`);
}

const person = { name: "Lasse" };

introduce.call(person); // "Hi, I'm Lasse"
```

**How it works:**
* First argument → the value of `this`
* Remaining arguments → normal parameters

So `call()` **calls** the function *right away*, with a custom `this`.

#### 2. apply()
**Purpose**: Just like `call()`, but takes arguments as an array.
```js
function showInfo(city, country) {
  console.log(`${this.name} lives in ${city}, ${country}.`);
}

const user = { name: "Anna" };

showInfo.apply(user, ["Bergen", "Norway"]); // "Anna lives in Bergen, Norway."
```

<small style="font-size: 1.5rem">

**Difference from `call()`**
| Method    | How it passes arguments                        |
| --------- | ---------------------------------------------- |
| `call()`  | comma-separated list (`fn.call(obj, a, b, c)`) |
| `apply()` | single array (`fn.apply(obj, [a, b, c])`)      |

This is especially useful when your arguments are already stored in an array.


#### 3. bind()


**Purpose**: Doesn’t call the function right away — instead, it **creates a new function** where `this` (and optionally some arguments) are permanently fixed.
```js
function greet(greeting) {
  console.log(`${greeting}, I'm ${this.name}`);
}

const user = { name: "Sofie" };

// Create a new function bound to user
const greetUser = greet.bind(user);

greetUser("Hello"); // "Hello, I'm Sofie"
```

You can think of `bind()` as **partial application + “this” control**.
It’s great for callbacks and event handlers where you don’t want to lose context.

#### Summary
| Method | Calls Immediately? | Arguments Form | What It Does |
| ------ | ------------------ | -------------- | ------------ |
| **`call()`**  | Yes | List (`a, b, c`) | Calls the function with a given `this` value |
| **`apply()`** | Yes | Array (`[a, b, c]`) | Calls the function with a given `this`, using arguments from an array |
| **`bind()`**  | No | List (`a, b, c`) | Returns a *new* function with a fixed `this` (and optional preset arguments) |

#### Why this matters in Functional Programming
* It proves that **functions are objects**, not just “pieces of code.”
* You can **manipulate** and reuse functions dynamically —
for example, pre-binding context or arguments.
* They enable powerful patterns like **partial application**, **method borrowing**, and **callback composition**.