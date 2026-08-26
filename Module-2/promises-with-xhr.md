# Promises with XHR (and Ghibli Films)

## Promise

A **Promise** in JavaScript is an object that represents the eventual completion (or failure) of an asynchronous operation - and its resulting value.

In simpler terms, it's a way to handle things that take time (like network requests) without blocking the rest of the code.

You can think of a Promise as a placeholder for a value that you'll get later. It can be in one of three states:
1. Pending - the operation hasn't finished yet.
2. Fulfilled - the operation succeeded, and you have a result.
3. Rejected - the operation failed, and you have an error.

Example:
```js
const promise = new Promise((resolve, reject) => {
  const success = true;

  if (success) {
    resolve("It worked!");
  } else {
    reject("Something went wrong");
  }
});

promise
  .then(result => console.log(result))   // runs if resolved
  .catch(error => console.error(error)); // runs if rejected
```

This pattern helps you avoid "callback hell" and makes async code easier to reason about - especially when chaining or combining multiple async tasks.

## XHR - a brief history lesson

Before Promises (and before [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)), data was often fetched using **[XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) (XHR)** - one of the oldest ways to make HTTP requests in JavaScript.

A typical XHR flow relied on **callbacks** and `onreadystatechange` events, which made code harder to read and maintain.

Here's a simple example that fetches a todo from [JSONPlaceholder](https://jsonplaceholder.typicode.com/), a free fake and reliable API for testing and prototyping:
```js
const xhr = new XMLHttpRequest();

// Step 1: Open a connection (GET request)
xhr.open("GET", "https://jsonplaceholder.typicode.com/todos/1");

// Step 2: Listen for changes in readyState
xhr.onreadystatechange = function() {
  // readyState 4 means "done"
  if (xhr.readyState === 4 && xhr.status === 200) {
    // Step 3: Parse and use the data
    const data = JSON.parse(xhr.responseText);
    console.log(data);
    // Send data wherever you want to do something with it...
  }
};

// Step 4: Send the request
xhr.send();
```

Explanation:
1. `xhr.open(method, url)` prepares the request.
2. `xhr.onreadystatechange` runs every time the request's state changes.
    * When `readyState` is `4` (request finished) and `status` is `200` (OK), we know we got a successful response.
3. We then parse the JSON text (`xhr.responseText`) into a JS object and use it.
4. We need to actually send the request.

[It works](https://codepen.io/xiaolasse/pen/VYaLxLb?editors=0012) - but it's clunky.

## Using a Promise and XHR to make a simpler `fetch`

Instead of nesting callbacks inside `onreadystatechange`, we can wrap the XHR logic inside a Promise.

That way, we can use `.then()` and `.catch()` just like with modern `fetch()`:

```js
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();       // Create XHR object
    xhr.open("GET", url);                   // Configure GET request

    xhr.onload = function() {               // Runs when request completes
      if (xhr.status !== 200) {             // Check for success (HTTP 200)
        // Reject on non-200 status, then stop - never parse an error page
        reject(new Error(`HTTP error: ${xhr.status}`));
        return;
      }

      // JSON.parse throws on a bad body, so guard it
      try {
        const data = JSON.parse(xhr.responseText);
        resolve(data);                      // Resolve with parsed JSON
      } catch {
        // Reject here, or the Promise would hang pending forever
        reject(new Error("Could not parse the response as JSON"));
      }
    };

    xhr.onerror = function() {              // Network-level failure
      reject(new Error("Network error"));
    };

    xhr.send();                             // Send the request
  });
}

// Using it
getData("https://jsonplaceholder.typicode.com/todos/3")
  .then(data => console.log("Data:", data)) // Runs if resolved
  .catch(err => console.error(err.message)); // Runs if rejected
```

Here's what happens, step by step:
```js
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
```
1. We create a new `XMLHttpRequest` and open a connection to the given URL.
2. The whole function returns a **Promise**, which lets us handle success or failure with `.then()` and `.catch()` later.
```js
    xhr.onload = function() {
      if (xhr.status !== 200) {
        reject(new Error(`HTTP error: ${xhr.status}`));
        return;
      }
```
3. The `onload` event runs once the request is *done* (see sidenote). We deal with the failure case first: anything other than a `200` OK (say a `404` or a `500`) rejects the Promise with the status code.
    * That `return` is load-bearing. `reject()` settles the Promise but it does *not* stop the function - without the `return` we would carry straight on and try to parse a `404` error page as if it were our data.
```js
      try {
        const data = JSON.parse(xhr.responseText);
        resolve(data);
      } catch {
        reject(new Error("Could not parse the response as JSON"));
      }
    };
```
4. If the status was fine, we parse the JSON text into a JS object and `resolve()` with it, which fulfills the Promise.
    * The `try` is not decoration. `JSON.parse` throws when the body isn't valid JSON, and that is easier to trigger than you'd think - a server having a bad day will happily hand you an HTML error page with a `200` status attached.
    * Normally a `throw` inside a Promise is turned into a rejection automatically, but only for code running *synchronously* inside the executor function. `onload` fires much later, long after the executor returned, so there is nothing left to catch a stray exception. Without the `try`, the Promise would neither resolve nor reject - it would sit pending forever, and your `.catch()` would never run. Silent hangs are much harder to debug than errors.
5. Note that we reject with `new Error(...)` rather than a plain string. You *can* reject with anything, but an `Error` gives you a stack trace, and it means `catch (err)` always hands you the same kind of thing - so `err.message` works no matter which branch failed.
```js
    xhr.onerror = function() {
      reject(new Error("Network error"));
    };
```
6. The `onerror` handler catches *network-level* problems (like no connection) and rejects the Promise. Note that a `404` does **not** land here - that's a successful request that returned a bad status, so it goes through `onload` above.
```js
    xhr.send();
  });
}
```
7. Finally, the request is sent.

Then you use it like this to get to the data:
```js
getData("https://jsonplaceholder.typicode.com/todos/3")
  .then(data => console.log("Data:", data)) // Runs if resolved
  .catch(err => console.error(err.message)); // Runs if rejected
```

### Sidenote: Why we don't need `readyState` anymore

In older XHR code, people used `xhr.onreadystatechange` and checked `xhr.readyState === 4` to know when the request had finished.

However, `xhr.onload` is a modern event that only fires once - when the request *is complete*.

So `readyState` is redundant here: `onload` already means "we're done, and here's the response."

## Why this `getData` is good

* It's minimal and easy to read.
* It returns actual data, not raw text.
* It handles HTTP errors, network errors and malformed responses.
* And, because it's Promise-based, you can chain it *or* use `await` in an `async` function:
```js
async function fetchData(url) {
  try {
    const data = await getData(url);        // Wait for the Promise to settle
    console.log("Data (async):", data);
  } catch (err) {
    console.error(err.message);             // Catches any of the three rejections
  }
}
// Call the function
fetchData("https://jsonplaceholder.typicode.com/todos/1");
```

So this version is now essentially a modern, Promise-wrapped XHR - simple, readable, and solid.

[See it live here, with both chained and `async` usage.](https://codepen.io/xiaolasse/pen/QwNbrQX?editors=0012)

## So, as promised (pun intended) some Ghibli films

The **Studio Ghibli API** (available at https://ghibliapi.vercel.app) is a free, public web API with data about the films of Studio Ghibli - titles, directors, release years, descriptions, artwork, and the characters, locations and vehicles that appear in them.

It returns data in JSON format, needs no authentication and no API key, which makes it easy to use in web projects and teaching examples.

**Base URL:**
[`https://ghibliapi.vercel.app`](https://ghibliapi.vercel.app)

**Some endpoints**
* All films:
[`https://ghibliapi.vercel.app/films`](https://ghibliapi.vercel.app/films)
* One film, by id:
[`https://ghibliapi.vercel.app/films/58611129-2dbc-4a81-a72f-77ddfc1b1b49`](https://ghibliapi.vercel.app/films/58611129-2dbc-4a81-a72f-77ddfc1b1b49)
* People, locations, species and vehicles work the same way:
`/people`, `/locations`, `/species`, `/vehicles`

**Example response (one film, shortened):**
```json
[
  {
    "id": "dc2e6bd1-8156-4886-adff-b39e6043af0c",
    "title": "Spirited Away",
    "description": "Spirited Away is an Oscar winning Japanese animated film about a ten year old girl who wanders away from her parents...",
    "director": "Hayao Miyazaki",
    "producer": "Toshio Suzuki",
    "release_date": "2001",
    "running_time": "124",
    "rt_score": "97",
    "image": "https://image.tmdb.org/t/p/w600_and_h900_bestv2/39wmItIWsg5sZMyRUHLkWBcuVCM.jpg"
  }
]
```

The real records have a few more fields, including the original Japanese titles and arrays of URLs pointing at related people, species, locations and vehicles. Open the endpoint in a browser tab and have a look.

### Sidenote: check the shape before you loop

`/films` gives you a **bare array**:

```json
[
  { "title": "Castle in the Sky", "director": "Hayao Miyazaki" },
  { "title": "Grave of the Fireflies", "director": "Isao Takahata" }
]
```

So `data` *is* the list, and you can go straight to `data.forEach(...)`.

Plenty of APIs don't do this. Many wrap the array inside an object, so you get something like `{ "results": [ ... ] }` or `{ "films": [ ... ] }`, and then you have to drill in first with `data.results.forEach(...)`.

Neither is wrong - but you can't guess which one you're dealing with. Look at the actual response before you write the loop. `console.log(data)` is the whole trick.

### Sidenote: one film, one object

`/films` returns an array. `/films/{id}` returns a single **object**:

```js
const film = await getData("https://ghibliapi.vercel.app/films/58611129-2dbc-4a81-a72f-77ddfc1b1b49");
console.log(film.title); // "My Neighbor Totoro"
```

Same API, same endpoint family, two different shapes. Which is the point of the sidenote above.

### 1. Make an `index.html` file

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Studio Ghibli Example</title>
  <script src="main.js" defer></script>
</head>
<body>
  <h1>Films by Hayao Miyazaki</h1>
  <ul id="film-list"></ul>
</body>
</html>
```

#### Sidenote: Why `defer` in the HTML matters
* Because the script tag uses `defer`, the browser parses HTML first, then executes `main.js`.
* That guarantees elements like `<ul id="film-list">` exist before `renderFilmList` runs - no need for `DOMContentLoaded` handlers.

### 2. Then add a `main.js` file

Here we:
1. Create a Promise-wrapped XHR helper function
2. Make a render helper that updates the DOM
3. Orchestrate: fetch, filter, then render, with async/await
4. Kick it off

```js
// 1. Fetch data from a URL using XHR wrapped in a Promise
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();       // Create XHR object
    xhr.open("GET", url);                   // Configure GET request

    xhr.onload = function() {               // Runs when request completes
      if (xhr.status !== 200) {             // Check for success (HTTP 200)
        // Reject on non-200 status, then stop - never parse an error page
        reject(new Error(`HTTP error: ${xhr.status}`));
        return;
      }

      // JSON.parse throws on a bad body, so guard it
      try {
        const data = JSON.parse(xhr.responseText);
        resolve(data);                      // Resolve with parsed JSON
      } catch {
        // Reject here, or the Promise would hang pending forever
        reject(new Error("Could not parse the response as JSON"));
      }
    };

    xhr.onerror = function() {              // Network-level failure
      reject(new Error("Network error"));
    };

    xhr.send();                             // Send the request
  });
}

// 2. Render list items to the DOM
function renderFilmList(films) {
  const list = document.getElementById("film-list");
  list.innerHTML = "";                      // Clear any previous content

  films.forEach(film => {                   // Loop through the films we were given
    const li = document.createElement("li");
    li.textContent = `${film.title} (${film.release_date})`;
    list.appendChild(li);                   // Add <li> to the <ul>
  });
}

// 3. Main async function: fetch, filter, render
async function loadFilms() {
  try {
    const url = "https://ghibliapi.vercel.app/films";
    const films = await getData(url);       // Wait for the Promise

    // Keep only Hayao's films - "===" is exact, see the sidenote below
    const miyazakiFilms = films.filter(film => film.director === "Hayao Miyazaki");

    renderFilmList(miyazakiFilms);          // Update the DOM
  } catch (err) {
    console.error(err.message);             // Handle any errors
  }
}

// 4. Start the process once script runs (after HTML is parsed via "defer")
loadFilms();
```

Note that we build each `<li>` with `createElement` and `textContent` rather than gluing together a string of HTML. Keep that habit - we'll come back to *why* it matters later in the course.

### Sidenote: filtering here, not there

The Amiibo API let you narrow the list on the server, with a query string like `?character=Luigi`. The Ghibli API doesn't offer that - you get all 22 films, every time.

So we filter in JavaScript instead:

```js
const miyazakiFilms = films.filter(film => film.director === "Hayao Miyazaki");
```

That's fine for 22 records. It would not be fine for 22 000 - at that point you'd want the server to do the narrowing, and an API that can't is an API you'd probably reject. Worth noticing that the choice exists.

Also worth noticing: `===` is an *exact* match. Studio Ghibli has two directors named Miyazaki - Hayao, and his son Goro, who directed three of the films. Had we written this instead:

```js
films.filter(film => film.director.includes("Miyazaki"));
```

...we'd have quietly picked up Goro's films too. (Old hands may recall the same trap in the Amiibo version of this lesson, where searching for the character `Luigi` also returned `Waluigi`.)

One practical wrinkle if you *do* want to match on Goro: his name is spelled with a macron over the o in the API data. Don't type it from memory - copy the exact string out of the response, or match on something else entirely.

### Sidenote: those numbers are strings

Look closely at the JSON:

```json
"release_date": "2001",
"running_time": "124",
"rt_score": "97"
```

Quotation marks. They're strings, not numbers. This is extremely common in real APIs and it will bite you the first time you try to sort or compare:

```js
"97" > "124"   // true - compared character by character, "9" beats "1"
Number("97") > Number("124")   // false - which is what you actually meant
```

Confusingly, the arithmetic operators *do* convert for you, so `"97" - "124"` gives you `-27` as you'd hope. It's the comparison operators (`>`, `<`) that quietly switch to alphabetical ordering when both sides are strings. That inconsistency is exactly what makes this a good bug to hide in.

If you're going to compare or do arithmetic on a value, convert it first and remove the doubt.

## Challenge

[Play a bit with this example](https://codepen.io/xiaolasse/pen/vEGOaBY?editors=1010) and try a few of these:

* Update `renderFilmList` to build cards instead of plain list items - each film has an `image`, a `description` and a `running_time` to work with. Add some CSS and make it look decent.
* Sort the films by `rt_score`, highest first. Then try writing the same sort using `>` instead of subtraction, and see whether you still trust the result.
* Swap the filter for something else: films from the 1980s, films under 100 minutes, films *not* directed by a Miyazaki.
* Point `getData` at `/people` or `/vehicles` instead and render that. The helper doesn't care what it's fetching - that's rather the point of it.
* Break it on purpose: change the URL to `https://ghibliapi.vercel.app/filmz` and check that your error handling actually does something useful.
