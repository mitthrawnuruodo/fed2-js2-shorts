# Object oriented programming



## Static



## Fetch and Sort Books (as objects)

**All code can be found on this repo: [book-example @ GitHub](https://github.com/mitthrawnuruodo/book-example).**

### 1) Setup

**Folder**
Set up the folder (within you Repos/Projects folder) in **VS Code**:
```
books-app/
├── index.html   # Page markup and buttons
├── script.js    # Logic: class, fetch, render, sort
└── books.json   # Book data (plain objects)
```

**Run it**
Use **Live Server** to open `index.html` <small>(this gives you `https://...`, which `fetch` requires)</small>.

### 2) HTML: what loads and when
The HTML declares the page, the list container, and three sort buttons. 

It also loads `script.js` with defer (so the script executes after the HTML is parsed, but before `DOMContentLoaded` fires), which is perfect for grabbing DOM nodes and kicking off `fetch`.

<small style="font-size: 1.4rem">

```html
<!doctype html>
<html lang="en">
<head>
  ...
  <script src="script.js" defer></script>
</head>
<body>
  <h1>Books, mandatory reading list</h1>
  <ol id="myList"></ol>

  <p>
    Sort by:
    <button id="sortByTitle">Title</button>
    <button id="sortByAuthor">Author</button>
    <button id="sortByYear">Year Published</button>
  </p>
</body>
</html>
```

**Key points**
* defer means the browser parses HTML first, then runs `script.js`. So when your JS calls `document.getElementById("myList")`, the element exists.
* The buttons have **IDs**, not inline `onclick`, so your JS attaches listeners programmatically.

</small>

### 3) Data file (JSON): just data, no methods
The `books.json` holds an object with an array of objects—only properties, no class instances yet. That’s ideal for `fetch` + `map`.
```json
{
  "books": [
    { 
      "title": "Hitchhikers Guide to the Galaxy",
      "authorFirstName": "Douglas",
      "authorLastName": "Adams",
      "published": 1979
    },
    ...
  ]
}
```

### 4) JavaScript: what runs, in what order, and why
Below are the key codebits from script.js:

#### 4.1 Class definition (data + behavior)
```js
// 1) Book "blueprint" with data + behavior
class Book {
  constructor(title, authorFirstName, authorLastName, yearPublished) {
    this.title = title;
    this.authorFirstName = authorFirstName;
    this.authorLastName = authorLastName;
    this.published = yearPublished;
  }
  
  // 2) Pretty print a book
  toString() {
    const {
      authorFirstName: af,
      authorLastName: al,
      title,
      published: pub
    } = this;
    return `${af} ${al}: ${title} (${pub})`;
  }
```

* **What**: `Book` is your model. Instances hold one book’s data. `toString()` formats it for display.
* **Why**: Keeping formatting logic on the class keeps rendering simple elsewhere.
* **When**: Defined immediately on script load (before any data is fetched).

#### 4.2 Rendering (static utility on the class)
```js
  // 3) Render a list of books into the <ol>
  static printList(booklist) {
    const listElement = document.getElementById("myList");
    let html = "";
    for (const book of booklist) {
      html += `<li>${book.toString()}</li>`;
    }
    listElement.innerHTML = html;
  }
```
* **What**: Creates the `<li>` items by calling each book’s `toString()` and injects them into `<ol id="myList">`.
* **Why**: Centralizes rendering in one place: call it after initial load and after each sort.
* **When**: Called after `fetch` (initial render) and after every sort.

#### 4.3 Comparators (for Array.sort)
```js
  // 4) Sort helpers (comparators)
  static sortByTitle(a, b) {
    const aTitle = a.title.toLowerCase();
    const bTitle = b.title.toLowerCase();
    if (aTitle > bTitle) return 1;
    if (aTitle < bTitle) return -1;
    return 0;
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

    return 0;
  }

  static sortByYear(a, b) {
    return a.published - b.published;
  }
}
```
* **What**: These functions tell `Array.prototype.sort` how to order two books.
* **Why**: Title and author sorts are **case-insensitive**; author sort falls back to first name when last names match; year sort is numeric.
* **When**: Used each time you click a sort button.

**Note**: There's an alternative title sort in the code, that uses `localeCompare` for sorting: 

```js
  static sortByTitleAlternative(a, b) {
    return a.title.localeCompare(b.title, "en", { sensitivity: "base" });
  }
```

#### 4.4 App state
```js
// 5) App state: our list (const ref; contents can change)
let myBooks = [];
```

* **What**: A single array that holds your current books. It starts empty.
* **Why**: You reassign it once after fetching; then you **sort it in place** on clicks.
* **When**: Declared before fetching.

#### 4.5 Initial data load (fetch JSON → class instances → render)
```js
// 6) Load data from JSON, convert to Book instances, show on page
async function init() {
  const res = await fetch("books.json");
  if (!res.ok) throw new Error("Failed to load books.json");
  const data = await res.json();

  // Convert plain objects → Book instances so methods work
  myBooks = data.books.map(
    b => new Book(b.title, b.authorFirstName, b.authorLastName, b.published)
  );

  Book.printList(myBooks);
}
```

* **What**: 
  * `fetch("books.json")` gets the raw objects; 
  * `map(...)` constructs real `Book` instances; 
  * finally it renders.
* **Why**: Turning plain objects into `Book` instances lets you use class methods like `toString()` and consistent sorting behavior.
* **When**: Called once at the bottom of the file to kick off the app.

> Live Server is essential here — `fetch` won’t work from `file://`, but you’ve already got that covered.

**Tip**: Normally you want some error handling (at least a [`try...catch`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch)) when fetching data.

#### 4.6 Sort handlers (called by button clicks)
```js
// 7) Sort handlers (called from the buttons)
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
```

* **What**: Each handler sorts the same `myBooks` array using one comparator, then re-renders.
* **Why**: Sorting in place keeps memory use simple and predictable for a small list.
* **When**: Fired whenever you click a sort button.

#### 4.7 Wiring up the buttons (unobtrusive JS)
```js
// 8) Attach listeners to the three buttons by id
document.querySelector("#sortByTitle").addEventListener("click", sortByTitle);
document.querySelector("#sortByAuthor").addEventListener("click", sortByAuthor);
document.querySelector("#sortByYear").addEventListener("click", sortByYear);
```

* **What**: Hooks up the UI to the logic.
* **Why**: Keeps HTML clean (no inline `onclick`), and makes it easy to change behavior in JS.
* **When**: Runs after parsing, thanks to `defer`.


#### 4.8 Start the app
```js
// 9) Kick things off
init();
```

* **What**: Triggers the initial fetch/render.
* **Why**: Without this call, the list would stay empty.
* **When**: Immediately after the listeners are set up.

> *(There’s also a commented-out alternative with catch that would render a “Failed to load books” message on network errors.)*

### TL;DR flow
1. **HTML parsed** → elements exist.
1. **`script.js` (defer) runs** → class defined, state prepared, listeners attached.
1. `init()` fetches `books.json`, maps to `Book` instances, **renders**.
1. Click **buttons** → sort the same `myBooks` array in place → **re-render**.
