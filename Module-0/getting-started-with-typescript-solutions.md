# Getting Started with TypeScript - Solutions

### Solution 1

```ts
let username: string = "Ada";
let age: number = 29;
let isLoggedIn: boolean = false;

console.log(username);    // "Ada"
console.log(age);         // 29
console.log(isLoggedIn);  // false
```

The syntax `: string`, `: number`, and `: boolean` after the variable name is a type annotation. It tells TypeScript exactly what kind of value this variable is allowed to hold. In these cases TypeScript would actually infer the types on its own, since the values are assigned straight away - but writing them out explicitly is good practice while you are learning to recognise the syntax.

---

### Solution 2

```ts
function greet(name: string): string {
  return "Hello, " + name + "!";
}

console.log(greet("Ada"));    // "Hello, Ada!"
console.log(greet("Turing")); // "Hello, Turing!"
```

Each parameter gets a type annotation after its name. The return type goes after the closing parenthesis, before the function body. Here both the parameter and the return value are strings, so the annotation is `: string` in both places. If you try to call `greet(42)`, TypeScript will flag it as an error before the code ever runs.

---

### Solution 3

```ts
function introduce(name: string, age?: number): string {
  if (age !== undefined) {
    return name + " is " + age + " years old.";
  }
  return name;
}

console.log(introduce("Ada", 29)); // "Ada is 29 years old."
console.log(introduce("Ada"));     // "Ada"
```

Adding `?` directly after a parameter name makes it optional. TypeScript understands that `age` may or may not be provided, so inside the function its type is `number | undefined`. That is why the `if (age !== undefined)` check is necessary - TypeScript will warn you if you try to use `age` as a plain number without checking first.

---

### Solution 4

```ts
type Product = {
  name: string;
  price: number;
};

function printProduct(product: Product): void {
  console.log(product.name + " costs " + product.price + " kr");
}

printProduct({ name: "Keyboard", price: 349 }); // "Keyboard costs 349 kr"
printProduct({ name: "Mouse", price: 199 });     // "Mouse costs 199 kr"
```

A `type` alias lets you give a name to a shape - here, an object with a `name` string and a `price` number. You can then use `Product` as a type annotation anywhere in your code, just like you would use `string` or `number`. The return type `void` means the function does not return a value - it only has a side effect (the `console.log`).

---

### Solution 5

```ts
const scores: number[] = [10, 7, 14];

function getAverage(numbers: number[]): number {
  const total = numbers.reduce((sum, n) => sum + n, 0);
  return total / numbers.length;
}

console.log(getAverage(scores)); // 10.333...
```

The syntax `number[]` means "an array where every element is a number". TypeScript will complain if you try to push a string into `scores`, or pass an array of strings to `getAverage`. This is one of the most common type annotations you will write in real projects.

---

### Solution 6

```ts
function toMessage(value: string | number): string {
  return "Value is: " + value;
}

console.log(toMessage("hello")); // "Value is: hello"
console.log(toMessage(42));      // "Value is: 42"
```

The `|` operator creates a union type - it means the parameter can be either a `string` or a `number`, but nothing else. The return type is always `string`, which is safe here because string concatenation with `+` converts numbers to strings automatically. Union types come up constantly in real TypeScript code, for example when a value might be a string or `null`.

Try adding these two lines to the Playground and see what happens:

```ts
console.log(toMessage(true));     // Error
console.log(toMessage([13, 42])); // Error
```

TypeScript will refuse to compile either of them. `true` is a `boolean` and `[13, 42]` is an array - neither is part of the `string | number` union, so TypeScript flags both as type errors before the code runs. This is the core promise of TypeScript: mistakes like passing the wrong kind of value are caught at compile time, not at runtime in a live application.
