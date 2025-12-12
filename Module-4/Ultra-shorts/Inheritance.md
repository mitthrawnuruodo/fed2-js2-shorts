# What is inheritance?
**Inheritance** means one class can **reuse** and **extend** another class.
The new class *inherits* properties and methods from the old one.
## Example 1 — Basic inheritance
```js
class Animal {
  speak() {
    console.log("Some sound");
  }
}

class Dog extends Animal {
  bark() {
    console.log("Woof!");
  }
}

const d = new Dog();
d.speak(); // "Some sound" (inherited)
d.bark();  // "Woof!"      (its own method)
```
* `Dog` **extends** `Animal` → gets all of Animal’s methods for free.
* You can also add new ones (like `bark()`).

## Example 2 — Overriding a method
```js
class Animal {
  speak() {
    console.log("Some sound");
  }
}

class Cat extends Animal {
  speak() {
    console.log("Meow!");
  }
}

const c = new Cat();
c.speak(); // "Meow!"
```
* The `Cat` class replaces the parent’s `speak()` with its own version.

## Example 3 — Using `super`
```js
class Animal {
  speak() {
    console.log("Some sound");
  }
}

class Dog extends Animal {
  speak() {
    super.speak(); // call parent method
    console.log("Woof!"); // then add something
  }
}

const dog = new Dog();
dog.speak();
// "Some sound"
// "Woof!"
```
* `super` lets you call the parent’s version **inside** the child’s method.

## In short:
Inheritance lets you **build on** existing classes instead of starting from scratch.