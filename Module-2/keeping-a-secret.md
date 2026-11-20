# Keeping a secret: API keys and Netlify Functions

Some APIs want a key with every request. The obvious place to put it is in your JavaScript, next to the `fetch` call. It works, and it's a mistake: everything you send to the browser can be read by whoever is using the browser. Open DevTools, look at the Network tab, and there is your key, in the URL of every request.

Hiding it better doesn't help. Minifying the code, splitting the key into pieces or reading it from a config file only changes where it sits. To use the key, the browser has to send it, and anything the browser sends is visible to the person using the browser.

So the browser can't have the key at all. Something else has to make the call for it.

In this lesson we take a small weather app and move its API key out of the browser and into a **Netlify Function**, then deploy the whole thing to Netlify from GitHub. The finished code is in [mitthrawnuruodo/weather-app-test](https://github.com/mitthrawnuruodo/weather-app-test), and there's a [live version](https://verdant-figolla-482aaa.netlify.app/) you can try.

A few terms first:

- **API key** - a string an API uses to know who is asking. It's tied to your account, so anyone who has it can use your quota (or your money) as if they were you.
- **Environment variable** - a named value that comes from the machine or service running your code, rather than from the code itself. The code only refers to it by name: `process.env.OPENWEATHER_API_KEY`.
- **Serverless function** - a small piece of JavaScript that runs on the host's servers when someone requests its URL. "Serverless" doesn't mean there is no server. It means you don't run one: Netlify starts the function when a request comes in, and you never manage the machine.
- **Proxy** - something that makes requests on someone else's behalf. Our function is a proxy: the browser asks it, and it asks the API.

## The idea

Without a function, the browser talks to the API directly, and the key travels with the request:

```text
Browser  --- city + key --->  api.openweathermap.org
```

With a function, the browser only talks to your own site. The function adds the key and makes the real request from Netlify's servers:

```text
Browser  --- city --->  your function on Netlify  --- city + key --->  api.openweathermap.org
         <-- weather --                           <-- weather ------
```

The key lives in an environment variable on Netlify. It never reaches the browser, so there is nothing to find in DevTools.

## Before you start

You need:

1. **Node.js** (a current LTS version) and npm. Check with `node -v`.
2. **Git**, and a **GitHub** account.
3. A free **[Netlify](https://www.netlify.com/)** account. Sign up with GitHub, since we'll deploy from a GitHub repository.
4. A free **[OpenWeatherMap](https://home.openweathermap.org/users/sign_up)** API key. After signing up, you'll find your keys under your profile, under API keys. Get one now: new keys can take a few hours to start working, and a key that isn't active yet gives you a `401` error that looks exactly like a mistake in your code.
5. The **Netlify CLI** (Command Line Interface), installed once:

```bash
npm install -g netlify-cli
```

If the install fails with a permission error (`EACCES`), don't reach for `sudo`. Skip the global install and put `npx` in front instead: `npx netlify-cli dev` instead of `netlify dev`, and so on.

The CLI can do a lot (deploy, manage environment variables, link folders to projects), but in this lesson we only need one command from it: `netlify dev`, which runs your site *and* your functions on your own machine. Everything else we'll do through GitHub and the Netlify dashboard.

If you've done [It works on my machine](it-works-on-my-machine.md), you already have most of this: the OpenWeatherMap key (the same one works here), a Netlify account and the Netlify CLI. You can skip straight to [The project](#the-project).

## The project

```text
weather-app/
  netlify/
    functions/
      fetch-weather.js   - the function (runs on Netlify)
  index.html             - markup
  script.js              - front end (runs in the browser)
  netlify.toml           - tells Netlify where things are
  .env                   - your key, for local use (never committed)
  .gitignore
```

Create the folder and start a Git repository in it:

```bash
mkdir weather-app
cd weather-app
git init
```

There is no `package.json` and no `npm install` in this project. Nothing needs installing: the front end is plain HTML and JavaScript, and the function only uses `fetch`, which is built into Node.

### The front end

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Weather App</title>
</head>
<body>
  <h1>Current Weather</h1>
  <input type="text" id="city" placeholder="Enter city name">
  <button id="getWeather">Get Weather!</button>
  <div id="weather"></div>

  <script src="script.js"></script>
</body>
</html>
```

`script.js`:

```javascript
document.getElementById('getWeather').addEventListener('click', async () => {
  const city = document.getElementById('city').value.trim();
  const weatherDiv = document.getElementById('weather');

  if (!city) {
    weatherDiv.innerHTML = '<p>Please enter a city name.</p>';
    return;
  }

  try {
    const response = await fetch(`/.netlify/functions/fetch-weather?city=${city}`);

    if (!response.ok) {
      weatherDiv.innerHTML = `<p>Error: ${response.status} ${response.statusText}</p>`;
      return;
    }

    const data = await response.json();
    //console.log(data);
    const { name, main: { temp }, sys: { country }, weather: [{description}] } = data;

    weatherDiv.innerHTML = `
      <p>Location: ${name} (${country})</p>
      <p>Temperature: ${temp}°C</p>
      <p>Weather: ${description}</p>
    `;
  } catch (error) {
    console.error('Error fetching weather data:', error);
    weatherDiv.innerHTML = '<p>Something went wrong. Please try again.</p>';
  }
});
```

Look at the `fetch` call. There's no API key, and no `api.openweathermap.org`. The browser asks a path on our own site, `/.netlify/functions/fetch-weather`, and that's all it ever knows about.

That path is where Netlify puts every function: `/.netlify/functions/` followed by the file name without `.js`.

There are two things in this file we'll improve in the exercises. You may spot them already.

### The function

`netlify/functions/fetch-weather.js`:

```javascript
// netlify/functions/fetch-weather.js

export const handler = async function (event, context) {
  const API_KEY = process.env.OPENWEATHER_API_KEY;
  const city = event.queryStringParameters.city || 'New York';
  const url = `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      return { statusCode: response.status, body: response.statusText };
    }
    const data = await response.json();
    return {
      statusCode: 200,
      body: JSON.stringify(data),
    };
  } catch (err) {
    return { statusCode: 500, body: err.toString() };
  }
};
```

This file never goes to the browser. Netlify runs it on its own servers, in Node, every time someone requests the function's URL. Line by line:

- `export const handler` - Netlify looks for an exported function called `handler` and calls it once per request.
- `event` - describes the incoming request. `event.queryStringParameters` is the query string as an object, already decoded: `?city=Bergen` becomes `{ city: 'Bergen' }`. With no query string, it's an empty object, so `city` falls back to `'New York'`.
- `context` - extra information about the function and the request. We don't use it here.
- `process.env.OPENWEATHER_API_KEY` - the key, read from an environment variable. It's set in a `.env` file on your machine, and in Netlify's settings when deployed. It's never in the code.
- `fetch(url)` - the real request to OpenWeatherMap, made from the server, with the key in the URL. The browser never sees this URL.
- The return value is an object with a `statusCode` and a `body`, and Netlify turns it into an HTTP response. The body has to be a string, which is why the data goes through `JSON.stringify()`.

If the API says no (a bad key, an unknown city), the function passes the status code on, so the front end's `response.ok` check still works. A `404` from OpenWeatherMap becomes a `404` from our function.

<details>
<summary>Rabbit hole: older tutorials use require('node-fetch')</summary>

Many guides (including the classroom version of this lesson) write the function like this:

```javascript
const fetch = require('node-fetch');

exports.handler = async function (event, context) {
  // ...the same as above
};
```

along with `npm install node-fetch@2`. That was needed when Node had no `fetch` of its own. Node has had a built-in `fetch` since version 18, and Netlify runs functions on current Node versions, so the extra package is no longer necessary.

The other difference is the module style. `require` and `exports.handler` are CommonJS, Node's original module system. `export const handler` is an ES module. Netlify accepts both, but don't mix them in one file. The `@2` in `node-fetch@2` was there for the same reason: version 3 is ES-module-only, and couldn't be loaded with `require`.

If you find an old tutorial with `node-fetch`, you can usually delete the `require` line and change `exports.handler =` to `export const handler =`, and it works the same.

</details>

<details>
<summary>Rabbit hole: the newer function style</summary>

Netlify's current documentation shows a different way of writing functions:

```javascript
export default async (request, context) => {
  const city = new URL(request.url).searchParams.get('city');
  // ...
  return Response.json(data);
};
```

Instead of a Lambda-style `event` and a `{ statusCode, body }` object, it receives a standard `Request` and returns a standard `Response`, the same objects `fetch` uses in the browser. Both styles work. We use the `handler` style here because it's what most existing examples use, so you'll recognise it when you see it. Netlify calls the older style "Lambda-compatible", after AWS Lambda, which it is based on.

</details>

### netlify.toml

`netlify.toml` tells Netlify where the functions are and which folder to publish:

```toml
[build]
  functions = "netlify/functions"
  publish = "."

[dev]
  functions = "netlify/functions"
  publish = "."
```

`[build]` is used when Netlify deploys your site, and `[dev]` when you run `netlify dev` locally. `publish = "."` means "serve the root of the repository": there is no build step, so the files are served as they are.

(`netlify/functions` happens to be Netlify's default, so the `functions` lines are not strictly needed. Writing them down makes it obvious where to look.)

### .gitignore and .env

`.gitignore` comes first, before the key is anywhere near the project:

```text
node_modules/
.env

# Local Netlify folder
.netlify
```

`.env` is where the key lives on your machine. `.netlify` is a folder the CLI creates for its own use. Neither belongs in Git.

Now create `.env` in the project root, with your real key in it:

```text
OPENWEATHER_API_KEY=your_actual_api_key_here
```

No quotes, no spaces around the `=`.

If you're used to keeping keys in `.env.local` (as Vite projects do), that works too: `netlify dev` reads both files. Just make sure the file you use is listed in `.gitignore`.

Then check what Git sees:

```bash
git status
```

`.env` should **not** be in the list. If it is, stop and fix `.gitignore` before you commit anything. A key that has been pushed to GitHub stays in the repository's history even if you delete the file later, and bots scan public repositories for keys all the time. If that ever happens, don't try to clean it up: create a new key and delete the old one.

## Run it locally

Start the Netlify development server from the project folder:

```bash
netlify dev
```

Among the first lines of output, you should see something like:

```text
Injected .env file env vars: OPENWEATHER_API_KEY
Loaded function fetch-weather
```

That's the CLI reading your `.env` file and handing the variable to the function, and finding the function in `netlify/functions`. Open `http://localhost:8888`, type a city and click the button.

This is where `netlify dev` earns its place. A normal static server (Live Server in VS Code, for example) only serves files, and there is no file called `/.netlify/functions/fetch-weather`. The request gets a `404`, and the app says `Error: 404`. `netlify dev` serves your files *and* runs your functions, the same way Netlify will once the site is live.

Stop the server with `Ctrl + C`. The `.env` file is only read when `netlify dev` starts, so restart it after changing the key.

### Check that the key is hidden

With the app running, open DevTools and go to the Network tab. Search for a city, and click the request that appears.

The request URL is `/.netlify/functions/fetch-weather?city=...`. No `appid`, no key, no `openweathermap.org`. Look at the Sources tab (Debugger in Firefox) too: `script.js` doesn't contain the key either. The browser has nothing to show you, because it was never given anything.

### Exercise 1

Predict first, then test. With `netlify dev` running, type this straight into the address bar of a new tab:

```text
http://localhost:8888/.netlify/functions/fetch-weather?city=Bergen
```

What do you get? And what happens if you leave out `?city=Bergen`?

<details>
<summary>Solution</summary>

You get the weather for Bergen as raw JSON: the whole response from OpenWeatherMap, with coordinates, wind, sunrise and more. Without `?city=Bergen`, you get the weather in New York, because of the fallback in the function. The key is nowhere in the response.

So far, so good. But notice what you just did: you used the weather API, on your key, without the app. Anyone who knows your site's address can do the same, from a browser, a script or a loop that runs all night. The function hides the key. It doesn't stop anyone from *using* it through your function. More on that after we deploy.

</details>

## Deploy to Netlify via GitHub

### Push to GitHub

Create a new, **empty** repository on GitHub, without a README, `.gitignore` or licence, since you already have a `.gitignore`. Then, from the project folder:

```bash
git add .
git status
```

Check the list one last time: `.env` must not be in it. Then:

```bash
git commit -m "Initial commit"
git branch -M main
git remote add origin <the repository URL GitHub shows you>
git push -u origin main
```

On GitHub, look at the files in the repository. You should see `index.html`, `script.js`, `netlify.toml`, `.gitignore` and the `netlify` folder, and no `.env`.

### Import the repository

1. In Netlify, add a new project, and choose to import an existing project from Git.
2. Pick GitHub, give Netlify access if it asks, and choose your repository.
3. Netlify reads `netlify.toml`, so the build settings should already be right: no build command, the root (`.`) as the publish directory, and `netlify/functions` as the functions directory. If any of the fields show something else, check that `netlify.toml` was pushed.

Netlify redesigns its dashboard now and then, so if a button is called something slightly different, trust the idea over the label.

### Add the key to Netlify

Your `.env` file never left your machine, so Netlify needs its own copy of the key. In the project's settings, find **Environment variables**, and add one:

- **Key:** `OPENWEATHER_API_KEY` - exactly the same name the function reads.
- **Value:** your API key.

If Netlify offers to mark the variable as containing a secret value, do so. Netlify then hides the value in the dashboard and logs, and scans your deploys to check that it doesn't turn up in the files you publish. With a function, it never does.

Some Netlify plans let you limit a variable to certain **scopes** (builds, functions, and so on). If yours does, the scope must include **Functions**, or the function won't see the variable.

### Deploy, and redeploy after changes

Deploy the site. After a short while you get a URL like `something-random.netlify.app`.

Netlify gives functions their environment variables when the site is deployed. If you added the key after the first deploy, or change it later, trigger a new deploy from the project's deploys page. Otherwise the function keeps running with the old value (or with no value at all).

From now on, every push to `main` deploys the site again, functions included.

### Check the live site

Open your live URL and search for a city. Then repeat the checks from before: the Network tab shows only the request to `/.netlify/functions/fetch-weather`, and nothing the browser downloaded contains the key.

If something doesn't work:

| Symptom | Likely cause |
| --- | --- |
| `Error: 401` | Netlify doesn't have the key, or has a different one. Check the variable name and value, and redeploy. A brand-new key may also not be active yet. |
| `Error: 404` for every city | The function wasn't found. Check that it's in `netlify/functions`, that it's called `fetch-weather.js`, and that the path in `script.js` matches. |
| `Error: 404` for one city | The function works. OpenWeatherMap just doesn't know that city. |

Why does a missing key give `401`? With no variable set, `process.env.OPENWEATHER_API_KEY` is `undefined`, the template string turns that into the text `undefined`, and OpenWeatherMap rejects `appid=undefined` like any other wrong key. The function passes the `401` on.

The function's own logs are under **Functions** in the project's dashboard. That's where an error inside the function shows up, since it never reaches the browser console.

And CORS? You won't see any CORS errors here. The page and the function are on the same site, so the browser treats the request as same-origin, and CORS rules don't apply. They would only come into play if a page on another domain tried to call your function.

## What the function doesn't protect

The key is now safe from anyone reading your front-end code. That was the goal, and it's a real improvement. But be clear about what you have, and what you don't:

- **The function itself is public.** As Exercise 1 showed, anyone can call it and use your quota through it. The key is secret, but using it isn't. For a free, rate-limited weather key, the worst case is that someone uses up your free calls - annoying, not dangerous. For a key that costs money or can change data, the function would also need to check *who* is asking (login, rate limits), which is a bigger topic.
- **The key is only as safe as the places it's stored.** That means your `.env` file, your Netlify account, and anyone who has access to its settings.
- **Don't send the key back by accident.** It's easy to `console.log(url)` while debugging, and the URL contains the key. In a function, that log ends up in Netlify's function logs. Returning the URL in an error message would send it straight to the browser.
- **The function trusts its input.** Whatever the browser sends ends up in a request that carries your key. That's what the next exercise is about.

### Exercise 2

Users type anything. Run the app and search for this exact text:

```text
Oslo&units=imperial
```

1. Predict: do you get the temperature in Celsius or Fahrenheit? Test it, and explain the result.
2. Fix `script.js` so the whole text reaches the function as one value. Then look at the URL the function builds. What has changed, and why is that worse?
3. Fix the function too.

Hint: `URLSearchParams` builds a query string and encodes every value for you.

<details>
<summary>Solution</summary>

**1. Celsius, for Oslo.** `script.js` pastes the text straight into the URL, so the browser requests:

```text
/.netlify/functions/fetch-weather?city=Oslo&units=imperial
```

In a query string, `&` separates parameters. Netlify reads this as two parameters, `{ city: 'Oslo', units: 'imperial' }`, and the function only looks at `city`. The rest of what the user typed silently disappeared.

**2. Fixing the front end** - let `URLSearchParams` build the query string:

```javascript
    // URLSearchParams encodes the city, so it arrives as one value
    const params = new URLSearchParams({ city });
    const response = await fetch(`/.netlify/functions/fetch-weather?${params}`);
```

Now the `&` and `=` are encoded (`city=Oslo%26units%3Dimperial`), and the function receives the whole text as the city. But the function also pastes the city straight into a URL, so the request it sends to OpenWeatherMap becomes:

```text
...weather?q=Oslo&units=imperial&appid=<your key>&units=metric
```

The user has added a parameter of their own to a request that carries *your* key. What OpenWeatherMap makes of two `units` parameters is up to OpenWeatherMap - that's the problem. The person typing should decide which city, and nothing else.

**3. Fixing the function** - the same tool, on the server:

```javascript
  const API_KEY = process.env.OPENWEATHER_API_KEY;
  const city = event.queryStringParameters.city || 'New York';

  // Every value is encoded, so the city can't add parameters of its own
  const params = new URLSearchParams({
    q: city,
    appid: API_KEY,
    units: 'metric',
  });
  const url = `https://api.openweathermap.org/data/2.5/weather?${params}`;
```

The function now sends `q=Oslo%26units%3Dimperial`: one odd city name, which OpenWeatherMap won't find. A `404` is the right answer to a city that doesn't exist.

Fixing only the front end isn't enough, because anyone can call the function directly (Exercise 1), with whatever query string they like. The rule: whatever code builds a URL is responsible for encoding the values in it.

`URLSearchParams` encodes spaces as `+`. That's fine: `New+York` arrives in the function as `New York`.

</details>

### Exercise 3

Send only what the page needs.

1. The function passes on OpenWeatherMap's entire response, but the page uses four values: the name, the country, the temperature and the description. Change the function so it sends only those four, as a flat object, and update `script.js` to match.
2. While you're in `script.js`: the data is put on the page with `innerHTML`. Change it so every line is added as text instead.

Why bother? Think about what happens to `script.js` if you one day switch to a different weather API.

<details>
<summary>Solution</summary>

**1. The function** builds a smaller object before returning it:

```javascript
    const data = await response.json();

    // Pick out the four values the page shows, and send only those
    const weather = {
      name: data.name,
      country: data.sys.country,
      temp: data.main.temp,
      description: data.weather[0].description,
    };

    return {
      statusCode: 200,
      body: JSON.stringify(weather),
    };
```

**2. `script.js`**, with both changes (and the encoding from Exercise 2):

```javascript
document.getElementById('getWeather').addEventListener('click', async () => {
  const city = document.getElementById('city').value.trim();
  const weatherDiv = document.getElementById('weather');

  // Put each line on the page as text, never as HTML
  function showLines(...lines) {
    const paragraphs = lines.map((line) => {
      const paragraph = document.createElement('p');
      paragraph.textContent = line;
      return paragraph;
    });
    weatherDiv.replaceChildren(...paragraphs);
  }

  if (!city) {
    showLines('Please enter a city name.');
    return;
  }

  try {
    const params = new URLSearchParams({ city });
    const response = await fetch(`/.netlify/functions/fetch-weather?${params}`);

    if (!response.ok) {
      showLines(`Error: ${response.status} ${response.statusText}`);
      return;
    }

    // The function sends exactly these four values now
    const { name, country, temp, description } = await response.json();

    showLines(
      `Location: ${name} (${country})`,
      `Temperature: ${temp}\u00B0C`,
      `Weather: ${description}`,
    );
  } catch (error) {
    console.error('Error fetching weather data:', error);
    showLines('Something went wrong. Please try again.');
  }
});
```

The nested destructuring is gone, because the function did that work. (`\u00B0` is the degree sign, written as an escape so the file stays plain ASCII.)

Why it's worth it:

- **The front end no longer knows which API you use.** It knows your function's shape - four flat values - and nothing about OpenWeatherMap's. Switch to another weather API, and only the function changes.
- **Less data over the network.** The full response is several times larger than the four values, and the browser downloaded all of it to use a few.
- **You decide what leaves the server.** There's nothing private in a weather report, but other APIs return things you don't want to pass on. Picking fields on purpose is a good habit.

And `textContent` instead of `innerHTML`: the text comes from someone else's server. With `textContent` it's always shown as text, never interpreted as HTML, so a strange city name can't add elements to your page.

</details>

## Summary

- An API key in front-end code is public, however it gets there. The browser has to send it, so the user can see it.
- A serverless function moves the call to the server. The browser asks your function, and the function adds the key and asks the API.
- The key lives in an environment variable: in `.env` locally (never committed), and in Netlify's settings when deployed. Redeploy after changing it.
- `netlify dev` runs your site and your functions locally, and reads `.env` for you. A plain static server can't run functions.
- The function is public. It hides the key, not the ability to use it, and it should treat its input with suspicion: encode values with `URLSearchParams`, and send back only what the page needs.

---

## Self-study: City Finder

A classmate has built a small app that looks up places by name and shows their coordinates, using OpenWeatherMap's [Geocoding API](https://openweathermap.org/api/geocoding-api). The same key you used for the weather app works here too. Their key is in `script.js`, and they want to put the app online.

Recreate the app from the files below, put your own key in `script.js`, and check that it works (any static server will do at this point, since there's no function yet). Search for `Bergen`. Then open the Network tab and find the key.

Your job: move the key into a Netlify Function called `find-city`, and deploy the app to Netlify from GitHub. When you're done:

1. Neither `script.js` nor any request the browser makes contains the key.
2. The function decides how many results to ask for (5). The browser can't change that.
3. The function doesn't pass on `local_names` - a long list of the place's name in other languages, which the page never uses.
4. A search with no place name gets a `400` from the function, without calling OpenWeatherMap at all.

`index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>City Finder</title>
</head>
<body>
  <h1>City Finder</h1>
  <input type="text" id="place" placeholder="Enter a place name">
  <button id="find">Find!</button>
  <ul id="results"></ul>

  <script src="script.js"></script>
</body>
</html>
```

`script.js`:

```javascript
// 1. Settings - the key is right here, in the browser
const API_KEY = 'paste_your_key_here';
const BASE_URL = 'https://api.openweathermap.org/geo/1.0/direct';

// 2. Elements
const placeInput = document.getElementById('place');
const findButton = document.getElementById('find');
const results = document.getElementById('results');

// 3. Show one line per match
function showResults(lines) {
  const items = lines.map((line) => {
    const item = document.createElement('li');
    item.textContent = line;
    return item;
  });
  results.replaceChildren(...items);
}

// 4. Search
findButton.addEventListener('click', async () => {
  const place = placeInput.value.trim();

  if (!place) {
    showResults(['Please enter a place name.']);
    return;
  }

  const params = new URLSearchParams({ q: place, limit: 5, appid: API_KEY });

  try {
    const response = await fetch(`${BASE_URL}?${params}`);

    if (!response.ok) {
      showResults([`Error: ${response.status}`]);
      return;
    }

    const places = await response.json();

    if (places.length === 0) {
      showResults([`No places called "${place}".`]);
      return;
    }

    showResults(places.map((match) => {
      const region = match.state ? `, ${match.state}` : '';
      return `${match.name}${region} (${match.country}): ` +
        `${match.lat.toFixed(2)}, ${match.lon.toFixed(2)}`;
    }));
  } catch (error) {
    console.error('Error finding places:', error);
    showResults(['Something went wrong. Please try again.']);
  }
});
```

Don't commit anything until the key is out of `script.js`. Once a key has been pushed, it's in the history for good.

<details>
<summary>Hint</summary>

The weather app is your template: the same `netlify.toml`, `.gitignore` and `.env`, and a function with the same overall shape. Most of `script.js` can stay as it is. Only the settings, and the parameters it sends, need to change.

For requirement 3, `map` with destructuring makes a short job of it.

</details>

<details>
<summary>Solution</summary>

**Project structure:**

```text
city-finder/
  netlify/
    functions/
      find-city.js
  index.html
  script.js
  netlify.toml
  .env
  .gitignore
```

`netlify.toml` and `.gitignore` are identical to the weather app's. `.env` holds the key under the same name:

```text
OPENWEATHER_API_KEY=your_actual_api_key_here
```

**`netlify/functions/find-city.js`:**

```javascript
// netlify/functions/find-city.js

export const handler = async function (event, context) {
  const API_KEY = process.env.OPENWEATHER_API_KEY;
  const place = event.queryStringParameters.q;

  // Nothing to look up - answer straight away, without spending a call
  if (!place) {
    return { statusCode: 400, body: 'Missing place name' };
  }

  // The function decides the limit, not the browser
  const params = new URLSearchParams({ q: place, limit: 5, appid: API_KEY });
  const url = `https://api.openweathermap.org/geo/1.0/direct?${params}`;

  try {
    const response = await fetch(url);
    if (!response.ok) {
      return { statusCode: response.status, body: response.statusText };
    }
    const places = await response.json();

    // Drop local_names (a long list of translations) - the page never uses it
    const slim = places.map(({ name, state, country, lat, lon }) => ({
      name, state, country, lat, lon,
    }));

    return {
      statusCode: 200,
      body: JSON.stringify(slim),
    };
  } catch (err) {
    return { statusCode: 500, body: err.toString() };
  }
};
```

**`script.js`** - only two parts change. The settings:

```javascript
// 1. Settings - no key here any more, only the address of our function
const BASE_URL = '/.netlify/functions/find-city';
```

And the parameters, which are now just the place name:

```javascript
  const params = new URLSearchParams({ q: place });
```

Everything else stays the same. The function sends back an array of places with the same field names as before, so the code that shows them doesn't need to know anything has changed.

**Checking each requirement:**

1. Run `netlify dev`, search, and look at the Network tab: the only request goes to `/.netlify/functions/find-city?q=Bergen`. Search `script.js` in the Sources tab for your key, and you won't find it.
2. The browser doesn't send a `limit` any more, and if someone adds `&limit=50` to the function's URL by hand, the function ignores it, since it only reads `q`.
3. Open the function's URL directly in the address bar (as in Exercise 1): each place has `name`, `country`, `lat`, `lon` and sometimes `state`, but no `local_names`. Places without a state just don't have the field, since `JSON.stringify` leaves out values that are `undefined`.
4. Open the function's URL without `?q=`: you get `400 Missing place name`. The `return` happens before `fetch`, so no call is made to OpenWeatherMap. (The page itself never sends an empty search, because `script.js` checks first. The function checks anyway, because anyone can call it directly.)

Then push to GitHub (check that `.env` isn't in the repository), import the repository in Netlify, add `OPENWEATHER_API_KEY` under Environment variables, deploy, and run the same checks on the live site.

**What changed, in one sentence:** the browser now knows *what* to look for, and only the function knows *how* - which API, which key, and how many results.

</details>
