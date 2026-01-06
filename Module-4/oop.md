# Object oriented programming and classes in JavaScript

**What is Object-Oriented Programming (OOP)?**

OOP is a way of structuring code around **objects** that:
* represent *things* (users, cars, games, etc.)
* bundle **data (properties)** and **behavior (methods)** together

In JavaScript, since ECMAScript 2015 (ES6), this is may be done with **classes**.

> Classes in JavaScript are *syntactic sugar over prototypes*. They did not change how the language works internally, but they made object-oriented code easier to read and write. 

## 1. A simple class

We start by modelling a real thing: a user.

```js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  greet() {
    return `Hi, I'm ${this.name}`;
  }
}
```

* `constructor` runs when a new object is created
* `this` refers to 
  * the *instance being built* inside constuctors
  * the *instance the method is being called on* elsewhere
* methods are shared by all instances

## 2. Creating objects (instances)

```js
const user1 = new User("Lasse", "lasse@example.com");
const user2 = new User("Anna", "anna@example.com");

console.log(user1.greet()); // "Hi, I'm Lasse"
console.log(user2.greet()); // "Hi, I'm Anna"
```
* Each instance has its own data, but uses the same methods.
* Each user is an independent object created from the same class.


## 3. Encapsulation (data + behavior together)
The object should manage its own state instead of letting outside code freely change it.
```js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
    this.isActive = true;
  }

  deactivate() {
    this.isActive = false;
  }

  activate() {
    this.isActive = true;
  }

  getStatus() {
    return this.isActive ? "Active" : "Inactive";
  }
}
```
Usage:
```js
const user = new User("Lasse", "lasse@example.com");

console.log(user.getStatus()); // "Active"
user.deactivate();
console.log(user.getStatus()); // "Inactive"
```
**Why this is encapsulation**
* State (`isActive`) lives inside the object
* State changes go through methods, not random assignments
* Rules about user status are kept in one place

JavaScript allows direct access to properties, but OOP is about discipline and structure, not just language features.

## 4. Inheritance (sharing behavior)
Classes can build on top of other classes.
```js
class Admin extends User {
  constructor(name, email) {
    super(name, email);
  }

  deleteUser(user) {
    return `${this.name} deleted ${user.name}`;
  }
}
```
* `class Admin extends User` defines `Admin` as a specialized version of `User` that *inherits* all user behavior and can add extra functionality. 
  * It models an “is-a” relationship: an admin *is a* user, with extra capabilities.
* `constructor()` runs when you create a new `Admin`
  * calls the `User` constructor via `super(...)`
  * sets up `name` and `email` exactly like a normal `User`
* `deleteUser()` is an **instance method** that exists only on `Admin`
  * `this` refers to the admin object
  * `user` is another `User` object passed in as an argument
  * It uses data from both objects to produce a result

Usage:
```js
const admin = new Admin("Root", "root@example.com");
const anna = new User("Anna", "anna@example.com");

console.log(admin.greet());          // "Hi, I'm Root"
console.log(admin.deleteUser(anna)); // "Root deleted Anna"; note: it didn't actually delete anything
```

**What inheritance gives us**
* `Admin` automatically gets everything from `User`
* Shared behavior stays in one place
* Specialized behavior lives in the subclass

## 5. Static methods

A **static method** belongs to the **class itself**, not to individual objects created from it.

You call it on the class, not on an instance.

Example:

```js
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  greet() {
    return `Hi, I'm ${this.name}`;
  }

  static isValidEmail(email) {
    return email.includes("@"); 
  }
}
```
Usage:
```js
const lasse = new User("Lasse", "lasse@example.com");

console.log(User.isValidEmail("test@example.com"));   // true
console.log(User.isValidEmail("not-an-email"));       // false

console.log(lasse.isValidEmail("test@example.com"));  // TypeError
```
* `User.isValidEmail(...)` works because `isValidEmail` is a **static method**, called on the class itself.
* It does not use `this` or any instance data.
* `lasse.isValidEmail(...)` fails because **static methods are not available on instances**.
  * `lasse` is an object created from `User`, but static methods must be called on `User` itself.

### When to use static methods

Use a static method when:
* the logic is **related to the concept** of the class
* but **does not depend on a specific instance**

Typical examples:
* validation
* factory helpers
* utility functions tied to a domain concept


## Project: Fetch and Sort Books (as Objects)

### Setup

**Folder**  
Set up a folder `book-app` or similar (within your Repos/Projects folder) and open it in VS Code:
```
books-app/
├── index.html   # Page markup and buttons
├── books.json   # Book data (plain objects)
└── script.js    # Logic: class, fetch, render, sort
```

### `index.html`

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Books, mandatory reading list</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <script src="script.js" defer></script>
</head>
<body>
  <h1>Books, mandatory reading list</h1>
  <ol id="myList"></ol>
  <div id="errorMessage" style="color: red"></div>

  <p>
    Sort by:
    <button id="sortByTitle">Title</button>
    <button id="sortByAuthor">Author</button>
    <button id="sortByYear">Year Published</button>
  </p>

</body>
</html>
```

This HTML file defines a **simple, semantic page structure** for displaying and interacting with a book list.
* The `<head>` sets character encoding, page title, mobile-friendly viewport, and loads `script.js` with `defer` so JavaScript runs after the HTML is parsed.
* The `<h1>` provides the main page heading.
* The ordered list `<ol id="myList">` is an empty placeholder that JavaScript will populate with books.
* The `<div id="errorMessage">` is reserved for displaying errors to the user.
* The buttons provide UI controls for sorting the book list by title, author, or publication year.

In short: the HTML defines the structure and UI; the behavior and data handling are expected to live in `script.js`.

### `books.json`

```json
{
  "books": [
    { "title": "Hitchhikers Guide to the Galaxy", "authorFirstName": "Douglas", "authorLastName": "Adams", "published": 1979 },
    { "title": "Neuromancer", "authorFirstName": "William", "authorLastName": "Gibson", "published": 1984 },
    { "title": "Snow Flower and the Secret Fan", "authorFirstName": "Lisa", "authorLastName": "See", "published": 2005 },
    { "title": "1Q84, book 1+2", "authorFirstName": "Haruki", "authorLastName": "Murakami", "published": 2009 },
    { "title": "1Q84, book 3", "authorFirstName": "Haruki", "authorLastName": "Murakami", "published": 2010 },
    { "title": "Night Train to Rigel", "authorFirstName": "Thimothy", "authorLastName": "Zahn", "published": 2005 },
    { "title": "Old Man's War", "authorFirstName": "John", "authorLastName": "Scalzi", "published": 2005 },
    { "title": "Agent to the Stars", "authorFirstName": "John", "authorLastName": "Scalzi", "published": 2005 },
    { "title": "Douglas Adams's Starship Titanic", "authorFirstName": "Terry", "authorLastName": "Jones", "published": 1997 },
    { "title": "Dirk Gently's Holistic Detective Agency", "authorFirstName": "Douglas", "authorLastName": "Adams", "published": 1987 },
    { "title": "Pattern Recognition", "authorFirstName": "William", "authorLastName": "Gibson", "published": 2003 },
    { "title": "Fingersmith", "authorFirstName": "Sarah", "authorLastName": "Waters", "published": 2002 },
    { "title": "BattleAxe/The Wayfarer Redemption", "authorFirstName": "Sara", "authorLastName": "Douglass", "published": 1995 },
    { "title": "White Cat", "authorFirstName": "Holly", "authorLastName": "Black", "published": 2010 },
    { "title": "Hidden Empire", "authorFirstName": "Kevin J.", "authorLastName": "Anderson", "published": 2002 },
    { "title": "The Windup Girl", "authorFirstName": "Paolo", "authorLastName": "Bacigalupi", "published": 2009 },
    { "title": "The Girl With All the Gifts", "authorFirstName": "M.R.", "authorLastName": "Carey", "published": 2014 },
    { "title": "Dissolution", "authorFirstName": "C.J.", "authorLastName": "Sansom", "published": 2004 },
    { "title": "The Name of the Rose", "authorFirstName": "Umberto", "authorLastName": "Eco", "published": 1980 },
    { "title": "Roman Blood", "authorFirstName": "Steven", "authorLastName": "Saylor", "published": 2000 },
    { "title": "Felidae", "authorFirstName": "Akif", "authorLastName": "Pirinçci", "published": 1989 }
  ]
}
```

This file is a JSON data file that contains a collection of books, meant to be loaded and used by JavaScript.
* The outer object has a single property, `"books"`, which holds an **array**.
* Each item in the array is a **book object** with the same structure:
  * `title`: the book title
  * `authorFirstName`: the author’s first name
  * `authorLastName`: the author’s last name
  * `published`: the year the book was published

In short: it’s structured, machine-readable data that the `script.js` can fetch, loop over, display, and sort.

### `script.js`

```js
// 1) Book "blueprint" with data + behavior
class Book {
  constructor(title, authorFirstName, authorLastName, yearPublished) {
    this.title = title;
    this.authorFirstName = authorFirstName;
    this.authorLastName = authorLastName;
    this.published = yearPublished;
  }

  // 2) Pretty print a book as a string
  toString() {
    const {
      authorFirstName: afn,
      authorLastName: aln,
      title,
      published: pub
    } = this; // Destructure the instance
    return `${afn} ${aln}: ${title} (${pub})`;
  }

  // 3) Render a list of books into the <ol>
  static printList(booklist) {
    const listElement = document.getElementById("myList");
    let html = "";
    for (const book of booklist) {
      html += `<li>${book.toString()}</li>`;
    }
    listElement.innerHTML = html;
  }

  // 4) Sort helpers (comparators)
  static sortByTitle(a, b) {
    const aTitle = a.title.toLowerCase();
    const bTitle = b.title.toLowerCase();
    if (aTitle > bTitle) return 1;
    if (aTitle < bTitle) return -1;
    return 0;
  }

  static sortByTitleAlternative(a, b) {
    return a.title.localeCompare(b.title, "en", { sensitivity: "base" });
  }

  static sortByAuthor(a, b) {
    const aLast = a.authorLastName.toLowerCase();
    const bLast = b.authorLastName.toLowerCase();
    if (aLast > bLast) return 1;
    if (aLast < bLast) return -1;

    const aFirst = a.authorFirstName.toLowerCase();
    const bFirst = b.authorFirstName.toLowerCase();
    if (aFirst > bFirst) return 1;
    if (aFirst < bFirst) return -1;

    // (Optional) then by title if needed
    // const aTitle = a.title.toLowerCase();
    // const bTitle = b.title.toLowerCase();
    // if (aTitle > bTitle) return 1;
    // if (aTitle < bTitle) return -1;

    return 0;
  }

  static sortByYear(a, b) {
    return a.published - b.published;
  }
} // class

// 5) App state: our list (const ref; contents can change)
let myBooks = [];

// 6) Load data from JSON, convert to Book instances, show on page
async function init() {
  try {
    const res = await fetch("books.json");
    console.log(res);
    if (!res.ok) throw new Error("Failed to load books.json");
    const data = await res.json();

    console.log(data);
    console.log(data.books);
    console.log(data.books[0]);
    console.log(data.books[0].title);

    // Convert plain objects → Book instances so methods work
    myBooks = data.books.map(
      b => new Book(b.title, b.authorFirstName, b.authorLastName, b.published)
    );

    Book.printList(myBooks); // Render the list after fetching
  } catch (e) {
    console.error(e.message);
    document.getElementById("errorMessage").innerText = e.message;
  } finally {
    console.log ("This runs on success and failure...");
  }
} // init

// 7) Sort handlers (called from the buttons in HTML)
function sortByTitle() {
  myBooks.sort(Book.sortByTitle);
  Book.printList(myBooks);
}
function sortByAuthor() {
  myBooks.sort(Book.sortByAuthor);
  Book.printList(myBooks);
}
function sortByYear() {
  myBooks.sort(Book.sortByYear);
  Book.printList(myBooks);
}

// 8) Add Eventlisteners
document.querySelector("#sortByTitle").addEventListener("click", sortByTitle);
document.querySelector("#sortByAuthor").addEventListener("click", sortByAuthor);
document.querySelector("#sortByYear").addEventListener("click", sortByYear);

// 9) Kick things off
init();

/*
// 9) Kick things off, alternative with error handling
init().catch(err => {
  console.error(err);
  document.getElementById("errorMessage").innerText = `Failed to load books.`;
});
*/
```

#### 1) `class Book` (a blueprint)
You define a **Book type** so each book can have:
* **data** (title, author names, year)
* **behavior** (methods like `toString()`)

Why: it keeps book-related logic in one place, instead of scattering it across functions.
```js
class Book { ... }
```
##### 1a) `constructor(...)`
Runs when you do `new Book(...)`. It copies values into the instance.
```js
constructor(title, authorFirstName, authorLastName, yearPublished) {
  this.title = title;
  this.authorFirstName = authorFirstName;
  this.authorLastName = authorLastName;
  this.published = yearPublished;
}
```
Why: you turn raw values into a consistent object shape you control.

#### 2) `toString()` (instance method)
Turns one book into a nice display string.
```js
toString() { ... return `${af} ${al}: ${title} (${pub})`; }
```
* Destructuring is just a clean way to read `this.authorFirstName` etc.

Why: the **book decides how it should be shown**, not the code that loops it.

#### 3) `static printList(booklist)` (class-level method)
Renders a list of books into `<ol id="myList">`.
* Finds the `<ol>`
* Builds an HTML string of `<li>` elements
* Writes it to `innerHTML`

Why static: printing a whole list is not something one book does; it’s a helper tied to the Book concept.

Why build one string: fewer DOM updates than appending many `<li>` nodes one by one.

#### 4) Static “sort helpers” (comparators)
These functions are meant to be used with [`Array.prototype.sort()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort).

##### `sortByTitle(a, b)`
* compares titles case-insensitively
* returns 1, -1, or 0 as required by `sort()`

##### `sortByTitleAlternative(a, b)`
uses `localeCompare`, which is often cleaner and handles language rules better

##### `sortByAuthor(a, b)`
* sorts by last name first
* if last names match, sorts by first name
* (optionally could sort by title as a third tie-breaker)

##### `sortByYear(a, b)`
* numeric sort: earlier years first

Why static: again, sorting is a “list operation”, not something a single instance needs to do.

#### 5) `let myBooks = []` (app state)
This is your “current list of books” in memory.

Why `let`: you replace it later after fetching (`myBooks = data.books.map(...)`), then you mutate it when sorting.

> The alternative is to use `const` and *never reassign the variable*; populate it with something like `myBooks.push(...newBooks);`, and only mutate its contents.

#### 6) `init()` function (load + convert + render)
This is the main startup function.

##### a) `fetch("books.json")`
Requests the JSON file.

##### b) `if (!res.ok) throw ...`
Ensures you catch HTTP errors (like 404) as real errors.

##### c) `const data = await res.json()`
Parses JSON text into a JavaScript object:
* `data.books` becomes an array of plain objects

##### d) A bunch of `console.log(...)`
These are just debug lines to confirm what’s coming back. Notice which data comes out of each, and comment out when done.

##### e) Convert plain objects into Book instances
```js
myBooks = data.books.map(
  b => new Book(b.title, b.authorFirstName, b.authorLastName, b.published)
);
```
Why this matters: JSON gives you **plain objects** with no class methods. Converting makes `book.toString()` work.

##### f) Print the initial list
```js
Book.printList(myBooks);
```

This line calls the **static method** on the `Book` class, from above, that:
* takes the array `myBooks`
* loops over all `Book` objects in it
* converts each book to a string using `book.toString()`
* renders the result as `<li>` elements inside the `<ol id="myList">`

In short: it tells the `Book` class to **render the current list of books to the page**.

##### g) catch
If anything fails (network, JSON parse, etc.):
* logs the error
* shows the message in `#errorMessage`

##### h) finally
Runs whether it succeeded or failed.

Why: good place for cleanup/logging that should always happen.

#### 7) Sort handlers (triggered by button clicks)
Each function:
* sorts the `myBooks` array in place, using the corresponding *static* helper method (one of the comparators) from the `Book` class
* re-renders the list using the *static* `printList()` method

Example:
```js
function sortByTitle() {
  myBooks.sort(Book.sortByTitle);
  Book.printList(myBooks);
}
```

Why: the DOM doesn’t magically update; *you sort the data, then re-render*.

#### 8) Add event listeners
Connects the HTML buttons to the JS functions.
```js
document.querySelector("#sortByTitle").addEventListener("click", sortByTitle);
```

#### 9) Call `init()` (start the app)
Runs the fetch + render process as soon as the script loads (after HTML parsing because of `defer`).

#### Quick “big picture” summary
* Fetch JSON
* Convert data into Book objects (so methods work)
* Render the list
* Let the user sort via buttons, then re-render after each sort

**Run it**: Use **Live Server** to open `index.html` (this gives you `https://...`, which `fetch` requires).

### Challenges

Add some of your own favourite books to the `books.json` file, then:

#### 1. Replace `innerHTML` with real DOM nodes
Challenge: Rewrite `Book.printList()` so it uses `document.createElement("li")` + `appendChild()` (or a `DocumentFragment`) instead of building an HTML string.
Why: safer rendering, avoids HTML injection, and shows efficient DOM patterns.

#### 2. Use `localeCompare` everywhere (also for author)
Challenge: Replace the manual `if (a > b)` comparisons in `sortByTitle` and `sortByAuthor` with `localeCompare` using `{ sensitivity: "base" }`.
Why: Makes for more robust, readable string sorting and avoids subtle case/diacritics issues.

#### 3. Add a stable tie-breaker
Challenge: Ensure deterministic results by adding tie-breakers (for example: author sort falls back to title, then year).
Why: shows why “equal” comparisons matter, especially when many items share the same year/author.

#### 4. Make sorting “toggle” ascending/descending
Challenge: Clicking the same sort button again should reverse the order (A→Z then Z→A, oldest→newest then newest→oldest).
Why: introduces state (current sort key + direction) and reinforces comparator thinking.

#### 5. Add a “Search” filter without refetching
Challenge: Add an `<input>` that filters the displayed list by title or author as you type, but keeps the original data unchanged.
Why: introduces derived UI state: keep `myBooks` as the source of truth and compute a `filteredBooks` array for rendering.

#### 6. Read one of the books
Challenge: Pick one book from the list and actually read it. Did you enjoy it? If so: pass it forward. 