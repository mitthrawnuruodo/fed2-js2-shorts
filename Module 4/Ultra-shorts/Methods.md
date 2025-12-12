# What is a method?
A **method** is just a **function that belongs to a class** (or an object).
It’s something the object *can do*.

## Example 1 — A simple class with one method
```js
class Person {
  sayHello() {
    console.log("Hello!");
  }
}

const p = new Person();
p.sayHello(); // "Hello!"
```
* `sayHello()` is a method.
* You call it on the object (`p.sayHello()`).

## Example 2 — Method using this
```js
class Person {
  constructor(name) {
    this.name = name; // property
  }

  greet() {
    console.log(`Hi, I'm ${this.name}`);
  }
}

const anna = new Person("Anna");
anna.greet(); // "Hi, I'm Anna"
```
* Inside a method, `this` refers to the current object.

## Example 3 — Multiple methods
```js
class Counter {
  constructor() {
    this.value = 0;
  }

  increase() {
    this.value++;
  }

  show() {
    console.log(this.value);
  }
}

const c = new Counter();
c.increase();
c.show(); // 1
```
Each method adds some behavior to the class:
* `increase()` changes data
* `show()` displays it

## In short:
Methods are functions inside a class that let objects *do things* or *react* to things.