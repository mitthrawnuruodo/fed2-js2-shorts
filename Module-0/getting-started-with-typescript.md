# Getting Started with TypeScript

This is a short warm-up for students who are new to TypeScript, or who need a quick refresher. Work through it before the course begins.

You already know JavaScript. TypeScript is not a replacement - it is JavaScript with a type system added on top. Every valid JavaScript file is already valid TypeScript. You are not starting over; you are adding one new layer.

---

## 1. Read and Learn

**Start here:** [typescriptlang.org/docs/handbook/typescript-in-5-minutes.html](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html)

This page is short and written specifically for people who already know JavaScript. It covers the key mental shift: TypeScript infers types where it can, and asks you to be explicit where it cannot. Read it once before doing anything else.

**Optional - video introductions:**

- "TypeScript in 100 Seconds" by Fireship: [youtube.com/watch?v=zQnBQ4tB3ZA](https://www.youtube.com/watch?v=zQnBQ4tB3ZA) - a very brief overview, good if you just want a quick sense of what TypeScript is before diving in.
- "Learn TypeScript in 50 Minutes": [youtube.com/watch?v=3mDny9XAgic](https://www.youtube.com/watch?v=3mDny9XAgic) - a more complete beginner crash course covering the most common features with clear examples.

---

## 2. Practice: The TypeScript Playground

No setup needed. Open the Playground in your browser and start writing:
[typescriptlang.org/play](https://www.typescriptlang.org/play)

The Playground compiles your TypeScript live and shows you errors immediately. It is the fastest way to experiment without having to configure a project.

Work through the exercises below. Solutions are in the separate solutions document.

---

### Exercise 1 - Variable type annotations

The following code has no type annotations. Add explicit types to each variable declaration.

```ts
let username = "Ada";
let age = 29;
let isLoggedIn = false;
```

---

### Exercise 2 - Typed function

Add type annotations for the parameters and the return type of this function.

```ts
function greet(name) {
  return "Hello, " + name + "!";
}
```

---

### Exercise 3 - Optional parameter

The `age` parameter should be optional. If it is not provided, the function should return just the name. Add the correct annotation.

```ts
function introduce(name, age) {
  if (age !== undefined) {
    return name + " is " + age + " years old.";
  }
  return name;
}
```

---

### Exercise 4 - Type alias for an object

Define a type alias called `Product` that describes the object used in the function below, then annotate the parameter.

```ts
function printProduct(product) {
  console.log(product.name + " costs " + product.price + " kr");
}
```

---

### Exercise 5 - Typed array

Declare a variable `scores` that is an array of numbers, containing the values 10, 7, and 14. Then write a function `getAverage` that takes that array and returns the average as a number.

---

### Exercise 6 - Union type

The function below should accept either a `string` or a `number` as its argument. Add the correct union type annotation, and make sure the return type is always a `string`.

```ts
function toMessage(value) {
  return "Value is: " + value;
}
```



---

## Appendix - Further Resources

**FreeCodeCamp - Learn TypeScript (Full Course):** [youtube.com/watch?v=30LWjhZzg50](https://www.youtube.com/watch?v=30LWjhZzg50)

A comprehensive 4+ hour deep dive into TypeScript. Too long to use as a warm-up, but useful if you want to revisit topics in more depth after the course has started. The chapter markers let you jump to specific topics as needed.