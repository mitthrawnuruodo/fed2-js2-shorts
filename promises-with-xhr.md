# Promises with XHR (and Amiibos)

## Promise

A **Promise** in JavaScript is an object that represents the eventual completion (or failure) of an asynchronous operation — and its resulting value.

In simpler terms, it’s a way to handle things that take time (like network requests) without blocking the rest of the code.

You can think of a Promise as a placeholder for a value that you’ll get later. It can be in one of three states:
1. Pending – the operation hasn’t finished yet.
2. Fulfilled – the operation succeeded, and you have a result.
3. Rejected – the operation failed, and you have an error.

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

This pattern helps you avoid “callback hell” and makes async code easier to reason about — especially when chaining or combining multiple async tasks.

## XHR - a brief history lesson

Before Promises (and before [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch)), data was often fetched using **[XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) (XHR)** — one of the oldest ways to make HTTP requests in JavaScript.

A typical XHR flow relied on **callbacks** and `onreadystatechange` events, which made code harder to read and maintain.

Here’s a simple example that fetches posts from [JSONPlaceholder](https://jsonplaceholder.typicode.com/), a free fake and reliable API for testing and prototyping:
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
* `xhr.open(method, url)` prepares the request.
* `xhr.onreadystatechange` runs every time the request’s state changes.
* When `readyState` is `4` (request finished) and `status` is `200` (OK), we know we got a successful response.
* We then parse the JSON text (`xhr.responseText`) into a JS object and use it.

[It works](https://codepen.io/xiaolasse/pen/VYaLxLb?editors=0012) — but it’s clunky.

## Using a Promise and XHR to make a simpler `fetch`

Instead of nesting callbacks inside `onreadystatechange`, we can wrap the XHR logic inside a Promise.

That way, we can use `.then()` and `.catch()` just like with modern `fetch()`:

```js
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);

    xhr.onload = function() {
      if (xhr.status === 200) {
        const data = JSON.parse(xhr.responseText);
        resolve(data);
      } else {
        reject(`Error: ${xhr.status}`);
      }
    };

    xhr.onerror = function() {
      reject("Network Error");
    };

    xhr.send();
  });
}

// Using it
getData("https://jsonplaceholder.typicode.com/todos/3")
  .then(data => console.log("Data:", data))
  .catch(err => console.error(err));
```

Here’s what happens, step by step:
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
      if (xhr.status === 200) {
        const data = JSON.parse(xhr.responseText);
        resolve(data);
      } else {
        reject(`Error: ${xhr.status}`);
      }
    };
```
3. The `onload` event runs once the request is *done* (see sidenote).
    * If the server responded with a `200` OK status, we parse the JSON and call `resolve(data)`, which fulfills the Promise.
    * If not (e.g., `404` or `500`), we call `reject()` to signal an error and include the status code.
```js
    xhr.onerror = function() {
      reject("Network Error");
    };
```
4. The `onerror` handler catches *network-level* problems (like no connection) and rejects the Promise.
```js
    xhr.send();
  });
}
```
5. Finally, the request is sent.

Then you use it like this to get to the data:
```js
getData("https://jsonplaceholder.typicode.com/todos/3")
  .then(data => console.log("Data:", data))
  .catch(err => console.error(err));
```

### Sidnote: Why we don’t need `readyState` anymore:
In older XHR code, people used `xhr.onreadystatechange` and checked `xhr.readyState === 4` to know when the request had finished.

However, `xhr.onload` is a modern event that only fires once — when the request *is complete*.

So `readyState` is redundant here: `onload` already means “we’re done, and here’s the response.”

## Why this `getData` is good
* It’s minimal and easy to read.
* It returns actual data, not raw text.
* It gracefully handles both HTTP errors and network errors.
* And, because it’s Promise-based, you can chain it *or* use `await` in an `async` function:
```js
async function fetchData(url) {
  try {
    const data = await getData(url);
    console.log("Data (async):", data);
  } catch (err) {
    console.error(err);
  }
}
// Call the function
fetchData("https://jsonplaceholder.typicode.com/todos/1");
```

So this version is now essentially a modern, Promise-wrapped XHR — simple, readable, and solid.

[See it live here, with both chained and `async` usage.](https://codepen.io/xiaolasse/pen/QwNbrQX?editors=0012)


## So, as promised (pun intended) some Amiibos

The **Amiibo API** (available at https://www.amiiboapi.com) is a free, public web API that provides detailed information about Nintendo’s **Amiibo** figurines and cards — including their characters, game series, release dates, and images.

It returns data in JSON format, which makes it easy to use with JavaScript apps, web projects, and teaching examples.

**Base URL:**
[`https://www.amiiboapi.com/api/amiibo/`](https://www.amiiboapi.com/api/amiibo/)

**Examples**
* All Amiibos:
[`https://www.amiiboapi.com/api/amiibo/`](https://www.amiiboapi.com/api/amiibo/)
* By character:
[`https://www.amiiboapi.com/api/amiibo/?character=Luigi`](https://www.amiiboapi.com/api/amiibo/?character=Luigi)
* By game series:
[`https://www.amiiboapi.com/api/amiibo/?gameseries=Super%20Mario`](https://www.amiiboapi.com/api/amiibo/?gameseries=Super%20Mario)

**Example response (shortened):**
```json
{
  "amiibo": [
    {
      "amiiboSeries": "Super Smash Bros.",
      "character": "Luigi",
      "gameSeries": "Super Mario",
      "head": "00010000",
      "tail": "000c0002",
      "image": "https://raw.githubusercontent.com/N3evin/AmiiboAPI/master/images/icon_00010000-000c0002.png",
      "release": { "jp": "2014-12-06" },
      "name": "Luigi"
    }
  ]
}
```

It’s a simple, read-only REST API — great for practicing HTTP requests, JSON parsing, and DOM rendering in frontend projects without needing authentication or API keys.

#### Sidenote about the Amiibo id

Each Amiibo in the API has two key identifiers:
* `head` – identifies the character or series group
* `tail` – identifies the specific figure or variation

Together, they form a unique 16-character Amiibo ID, which you can use to fetch that exact item.

Example (Luigi figure):
* `head` = "00010000"
* `tail` = "000c0002"
* `id` = "00010000000c0002"
* URL: https://www.amiiboapi.com/api/amiibo/?id=00010000000c0002

### 1. Make an `index.html` file: 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Amiibo Example</title>
  <script src="main.js" defer></script>
</head>
<body>
  <h1>Luigi Amiibos</h1>
  <ul id="amiibo-list"></ul>
</body>
</html>
```

#### Sidenote: Why `defer` in the HTML matters
* Because the script tag uses `defer`, the browser parses HTML first, then executes `main.js`.
* That guarantees elements like `<ul id="amiibo-list">` exist before `renderAmiiboList` runs — no need for `DOMContentLoaded` handlers.

### 2. Then add `main.js` file: 

Her we:
1. Create a Promise-wrapped XHR helper function
2. Make a Render helper that updates the DOM
3. Orchestrate: fetch then render, with async/await
4. Kick it off

```js
// 1. Fetch data from a URL using XHR wrapped in a Promise
function getData(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();       // Create XHR object
    xhr.open("GET", url);                   // Configure GET request

    xhr.onload = function() {               // Runs when request completes
      if (xhr.status === 200) {             // Check for success (HTTP 200)
        const data = JSON.parse(xhr.responseText);
        resolve(data);                      // Resolve with parsed JSON
      } else {
        reject(`Error: ${xhr.status}`);     // Reject on non-200 status
      }
    };

    xhr.onerror = function() {              // Network-level failure
      reject("Network Error");
    };

    xhr.send();                             // Send the request
  });
}

// 2. Render list items to the DOM
function renderAmiiboList(data) {
  const list = document.getElementById("amiibo-list");
  list.innerHTML = "";                      // Clear any previous content

  data.amiibo.forEach(item => {             // Loop through all amiibos
    const li = document.createElement("li");
    li.textContent = `${item.name} (${item.gameSeries})`;
    list.appendChild(li);                   // Add <li> to the <ul>
  });
}

// 3. Main async function: fetch + render
async function loadAmiibos() {
  try {
    const url = "https://www.amiiboapi.com/api/amiibo/?character=Luigi";
    const data = await getData(url);        // Wait for the Promise
    renderAmiiboList(data);                 // Update the DOM
  } catch (err) {
    console.error(err);                     // Handle any errors
  }
}

// 4. Start the process once script runs (after HTML is parsed via "defer")
loadAmiibos();
```

**Note**: The AmiiboAPI doesn’t return the array by itself — it wraps it inside an object.

When you fetch from https://www.amiiboapi.com/api/amiibo/?character=Luigi,
the response looks like this (simplified):
```json
{
  "amiibo": [
    { "name": "Luigi", "gameSeries": "Super Mario" },
    { "name": "Luigi", "gameSeries": "Super Smash Bros." }
  ]
}
```
So the variable `data` holds that entire object.

To reach the actual list of amiibo items, you have to access its `amiibo` property:
```js
data.amiibo
```
That’s the array — and that’s what we want to loop through.

If the API instead returned a bare array (like `[ {…}, {…} ]`),  
you could just write `data.forEach(...)`.  
But since it’s nested inside `data.amiibo`,  
we need to “drill into” the object to reach the array first:
```js
  data.amiibo.forEach(item => { ... };
```

There is - at the time of writing - **929** Amiibos in the Amiibo API, and to reduce the list we've filtered the search and only included the character `Luigi` (and - for some reason - `Waluigi`, probably since `Luigi` is a substring of `Waluigi`, since we don't get `Wario` when searching for the character `Mario`.)

To get the characters you want, refer to the AmiiboAPI Documentation https://www.amiiboapi.com/docs/

## Challenge

[Play a bit with this example](https://codepen.io/xiaolasse/pen/vEGOaBY?editors=1010) and maybe update the `renderAmiiboList` to make a list of cards or similar, and not just simple list items, and make it nicer with some CSS? 