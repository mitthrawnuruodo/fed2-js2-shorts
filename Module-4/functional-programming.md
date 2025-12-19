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
