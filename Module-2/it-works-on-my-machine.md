# It works on my machine

You have built something. It fetches data, it renders nicely, and it runs perfectly on `localhost`. Then someone asks for a link, and you realise `localhost` means "this computer" - nobody else can see it.

This article is about closing that gap. We will build five small projects and put each of them on the web, using four different routes:

| # | Project | Build step | Deploys to | Route |
|---|---|---|---|---|
| 1 | Pokedex in plain JS | None | Netlify | Git |
| 2 | The same Pokedex in TS | `tsc` | Vercel | Git |
| 3 | Rick and Morty characters (Vite) | `vite build` | Netlify | CLI |
| 4 | Weather in Norway (Vite) | `vite build` | Vercel | CLI |
| 5 | Weather search with an API key (Vite) | `vite build` | Netlify | Git |

The projects get a little more complicated as we go, and each one adds one new idea. Projects 1 and 2 are the same app on purpose: the only difference between them is the compile step, so that is the one thing you need to watch.

A few terms before we start:

- **Host** - a service that stores your files and serves them to anyone who asks. Netlify and Vercel are both hosts with free tiers.
- **Static site** - a site made only of files (HTML, CSS, JS, images) that the host sends as they are. Everything in this article is a static site.
- **Deploy** - putting a version of your site on a host so it gets a public URL.
- **Build** - the step that turns your source code into the files a browser can actually use. Some projects need one, some don't.
- **Publish directory** (Netlify) / **output directory** (Vercel) - the folder the host should serve. Everything outside it stays private.
- **CLI** - Command Line Interface, a program you run in the terminal. Both hosts have one.

## Why not just use GitHub Pages?

You may already have used [GitHub Pages](https://pages.github.com/), which is also free and also serves static sites. Netlify and Vercel are alternatives to it, not replacements, and each has its place.

**GitHub Pages** is the simplest option when there is nothing to build. There's no extra account or service - it's a setting in the repository you already have, and projects like project 1 work straight away. Its limits show up once a build is involved. Pages only serves files. To run `tsc` or `vite build`, you have to write a GitHub Actions workflow yourself, and a project site lives at `username.github.io/repo-name/`, a sub-folder that Vite has to be told about (with the `base` option) or links to assets will break. There's also no place to run code on a server, which you'd need to keep an API key truly secret (more on that in project 5).

**Netlify and Vercel** run the build for you as part of every deploy. They also give you separate preview URLs to test changes before they go live, a settings page for environment variables, a CLI, and serverless functions for when you do need code on a server. The downsides: it's one more account, the free tiers have usage limits, and they are commercial products whose dashboards and plans change more often than GitHub's. Vercel's free Hobby plan, for example, is meant for personal, non-commercial projects.

A rough rule: plain HTML, CSS and JS with no build step - GitHub Pages is fine. Anything that needs a build or a secret - Netlify or Vercel will save you work.

And if you ever need a real back end - a Node/Express server that runs all the time, or a database - look at [Render](https://render.com/). It hosts static sites too, but its strength is long-running servers, which Netlify and Vercel aren't built for. We won't use it here. One thing to know if you try it: free servers there go to sleep when nobody uses them, so the first request after a break can take a while.

## Before you start

You need:

1. **Node.js 22.13 or newer.** The Netlify CLI refuses to install on anything older. Check with `node -v`.
2. **A GitHub account**, and Git set up on your machine.
3. **Free accounts on [Netlify](https://www.netlify.com/) and [Vercel](https://vercel.com/).** Sign up with your GitHub account rather than with an email address. Three of the five projects deploy straight from GitHub, so both hosts need to talk to GitHub anyway, and this links them from the start. It also means no extra passwords, and the CLI logins in projects 3 and 4 become a single click when you're already logged in to GitHub. You'll still be asked which repositories each host may read the first time you import one - that's a separate permission, and a good one to keep narrow.
4. **A free [OpenWeatherMap](https://openweathermap.org/) API key.** Do this now, even though we won't need it until project 5. New keys can take a few hours to start working, and a key that isn't active yet gives you a `401` error that looks exactly like a mistake in your code.

A note on screenshots and button labels: both hosts redesign their dashboards regularly. The steps below describe what to look for rather than the exact wording, so if a button is called something slightly different, trust the idea over the label.

---

## Project 1: Pokedex in plain JS

**Goal:** search for a Pokemon by name or number and show its picture and types. Data comes from [PokeAPI](https://pokeapi.co/), which needs no key.

### The files

```text
p1-pokedex-js/
  .gitignore
  package.json
  index.html
  style.css
  js/
    api.js
    main.js
```

`index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pokedex</title>
    <link rel="stylesheet" href="style.css">
    <script type="module" src="js/main.js"></script>
  </head>
  <body>
    <main>
      <h1>Pokedex</h1>
      <form id="search-form">
        <label for="search">Name or number</label>
        <input id="search" name="search" required>
        <button>Search</button>
      </form>
      <div id="output"></div>
    </main>
  </body>
</html>
```

`style.css`:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
  padding: 0 1rem;
}

#output img {
  width: 12rem;
  image-rendering: pixelated;
}
```

`js/api.js`:

```javascript
// 1. Settings
const BASE_URL = 'https://pokeapi.co/api/v2';

// 2. Fetch one Pokemon by name or number
export async function getPokemon(nameOrNumber) {
  const query = String(nameOrNumber).trim().toLowerCase();

  // Only spaces would fetch the whole list instead of one Pokemon
  if (!query) {
    throw new Error('Type a name or a number first');
  }

  const response = await fetch(`${BASE_URL}/pokemon/${query}`);

  if (!response.ok) {
    throw new Error(`Could not find "${query}" (${response.status})`);
  }

  return response.json();
}
```

`js/main.js`:

```javascript
import { getPokemon } from './api.js';

// 1. Elements
const form = document.querySelector('#search-form');
const input = document.querySelector('#search');
const output = document.querySelector('#output');

// 2. Rendering
function renderMessage(message) {
  const paragraph = document.createElement('p');
  paragraph.textContent = message;
  output.replaceChildren(paragraph);
}

function renderPokemon(pokemon) {
  const card = document.createElement('article');
  const heading = document.createElement('h2');
  const image = document.createElement('img');
  const types = document.createElement('p');

  heading.textContent = `#${pokemon.id} ${pokemon.name}`;
  image.src = pokemon.sprites.front_default;           // Can be null!
  image.alt = pokemon.name;
  types.textContent = pokemon.types
    .map((entry) => entry.type.name)
    .join(', ');

  card.append(heading, image, types);
  output.replaceChildren(card);
}

// 3. Search
form.addEventListener('submit', async (event) => {
  event.preventDefault();
  renderMessage('Loading...');

  try {
    const pokemon = await getPokemon(input.value);
    renderPokemon(pokemon);
  } catch (error) {
    renderMessage(error.message);
  }
});
```

Notice we build the card with `createElement` and `textContent` rather than `innerHTML`. The data comes from someone else's server, so we never let it be interpreted as HTML.

### Run it locally

Double-clicking `index.html` won't work. The page loads, but the script doesn't, and the console complains about CORS. You need a small local web server instead.

You probably use the Live Server extension in VS Code for this. Here we'll use [live-server](https://www.npmjs.com/package/live-server) from npm instead - the same idea, but installed as part of the project. Anyone who clones the repo gets the same server with `npm install`, and it works in any editor.

From the project folder:

```bash
npm init -y
npm install -D live-server
```

`-D` saves it as a **devDependency**: something you need while developing, but not something the finished site needs. Netlify serves your files without it.

Then add a `dev` script, so `package.json` looks like this. `npm init -y` also adds fields like `version`, `main` and `license` - you can leave them or delete them, they don't matter here:

```json
{
  "name": "p1-pokedex-js",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "live-server"
  },
  "devDependencies": {
    "live-server": "^1.2.2"
  }
}
```

And a `.gitignore`, so the installed packages stay out of Git:

```text
node_modules
```

Now start it:

```bash
npm run dev
```

Your browser opens `http://127.0.0.1:8080`, and the page reloads by itself every time you save a file. Search for `pikachu`. Stop the server with `Ctrl + C`.

Expect `npm install` to print a handful of deprecation warnings and a list of "vulnerabilities" (10 at the time of writing). That's because live-server is no longer maintained: the last release was in 2022, and many of the packages it depends on are older still. The same goes for the Live Server extension in VS Code, which is built on the same package.

For now, that's acceptable, for two reasons. live-server only runs on your own machine while you develop, and it never becomes part of the deployed site. And it still works fine on current Node versions. But keep the warnings in mind: an unmaintained tool can stop working after a Node or operating system update, and nobody will be there to fix it.

<details>
<summary>Rabbit hole: what if live-server stops working?</summary>

The closest replacement is [Five Server](https://github.com/yandeu/five-server), a maintained rewrite of live-server. Like live-server, it comes both as an npm package (`five-server`) and as a VS Code extension, so you can still use the same tool in both places. It even uses port 5500 by default, like the Live Server extension.

Switching is a small change: `npm install -D five-server`, and `"dev": "five-server"` instead of `"dev": "live-server"`.

</details>

<details>
<summary>Rabbit hole: why won't modules load from a double-click?</summary>

When you double-click, the browser opens the file with a `file://` URL. Module scripts (`type="module"`) are always fetched using CORS rules, and a page loaded from `file://` has no proper origin - browsers treat it as `null`. A module request from a `null` origin is refused.

Classic scripts without `type="module"` don't have this rule, which is why older tutorials got away with double-clicking. live-server gives the page a real origin (`http://127.0.0.1:8080`), and the problem disappears. So will it on Netlify, since that is a real web server too.

</details>

### Deploy: Netlify via Git

1. Create a new repository on GitHub, and push the project to it (see the reminder below if you need it).
2. In Netlify, add a new project and choose to import an existing project from Git.
3. Pick GitHub, give Netlify access if it asks, and choose your repository.
4. On the settings screen, leave the **build command** empty, and leave the **publish directory** empty (or set it to `.`, which means "the root of the repo"). Netlify will still run `npm install`, because the project has a `package.json`. That's harmless: it installs live-server, and nothing uses it.
5. Deploy. After a few seconds you get a URL like `something-random.netlify.app`. You can rename it under the project's domain settings.

<details>
<summary>Reminder: pushing a new project to GitHub</summary>

Create an **empty** repository on GitHub - don't let GitHub add a README or `.gitignore`. You already have those locally, and an extra commit on GitHub makes your first push fail. Then, from the project folder:

```bash
git init
git add .
git status
```

Check the list `git status` shows: `node_modules` should **not** be in it. If it is, your `.gitignore` is missing or misspelled - fix that before you go on. Then:

```bash
git commit -m "First commit"
git branch -M main
git remote add origin https://github.com/your-username/your-repo.git
git push -u origin main
```

Use the repository URL GitHub shows you instead of the example. The same steps work for every Git deploy in this article.

</details>

From now on, every push to your main branch triggers a new deploy on its own. That is the big advantage of the Git route: you never deploy by hand again.

**What just happened?** Netlify copied your files from GitHub and served them exactly as they are. There was nothing to build: the browser gets `js/main.js`, and `js/main.js` is already JavaScript. Hold on to that thought - it stops being true in the next project.

### Exercise 1

PokeAPI also returns a Pokemon's `height` and `weight`. Show them on the card in metres and kilograms. Then commit, push, and watch Netlify deploy the change without you doing anything else.

Hint: the API does not use metres and kilograms.

<details>
<summary>Solution</summary>

PokeAPI gives height in decimetres and weight in hectograms, so both need dividing by 10. Pikachu has `height: 4` and `weight: 60`, which is 0.4 m and 6 kg.

In `renderPokemon`, add one more paragraph:

```javascript
  const types = document.createElement('p');
  const size = document.createElement('p');

  // ...the existing lines...

  // PokeAPI uses decimetres and hectograms - divide by 10
  size.textContent =
    `${pokemon.height / 10} m, ${pokemon.weight / 10} kg`;

  card.append(heading, image, types, size);
```

After `git push`, the deploy list in Netlify shows a new entry within a few seconds, and the live site updates when it finishes. No build was needed, so the deploy is quick.

</details>

---

## Why TypeScript has to be compiled

Before we convert the Pokedex to TypeScript, let's see what happens if we try to skip the compile step. Rename `js/main.js` to `js/main.ts`, update the `<script>` tag to point at it, and reload.

The page does nothing, and the console shows an error. Which error depends on the browser, but the cause is the same, and it has two layers.

**Layer 1: the server doesn't know what a `.ts` file is.** Web servers decide the `Content-Type` of a file from its extension. For `.ts`, most of them say `video/mp2t` - an MPEG video format that happened to claim the extension first. You can check this yourself:

```bash
curl -I http://127.0.0.1:8080/js/main.ts
```

Among the headers you'll see `Content-Type: video/mp2t`. A browser won't run a video as a script, so it refuses before even looking inside.

(When you've seen the error, rename the file back to `main.js` and fix the script tag again, so project 1 keeps working.)

**Layer 2: even if it did look inside, it's not JavaScript.** A line like this is a syntax error to a JavaScript engine:

```typescript
function renderMessage(message: string): void {
```

`: string` and `: void` mean nothing in JavaScript. Browsers only understand JavaScript, so something has to remove the TypeScript parts before a browser sees the code.

That "something" does two separate jobs, and it helps to keep them apart:

1. **Type checking** - reading the code and reporting type errors. This is what `tsc` (the TypeScript compiler) is really for. It produces error messages, not files a browser needs.
2. **Transpiling** - producing JavaScript files with the types removed. `tsc` can do this as well. Tools like Vite use faster, separate tools for this part, which strip the types without checking them.

Both jobs have to happen before the site goes live. The only question is **where**:

- **On your machine** - you run the build and upload the finished files.
- **On the host** - you give the host your source code and a build command, and the host runs the build.

Git deploys normally use the second option: GitHub only has what you committed, and build output is normally not committed. With a CLI deploy, you can choose - and the two CLIs make different choices by default, as you'll see in projects 3 and 4.

<details>
<summary>Rabbit hole: but Node can run .ts files now?</summary>

Yes - recent versions of Node run many `.ts` files directly (this was tested on Node 22.22). But Node doesn't compile them. It strips the types out, just blanking out the TypeScript parts of the text and running what is left.

That only works for TypeScript features that are pure type annotations. Something like an `enum` generates real code, so it can't simply be blanked out, and Node throws a syntax error. This is also why the current Vite `tsconfig.json` includes `"erasableSyntaxOnly": true` - it makes the compiler warn you about exactly those features.

None of this helps in the browser: browsers do no type stripping at all.

</details>

---

## Project 2: The same Pokedex in TypeScript

**Goal:** the same app, now in TypeScript, compiled with plain `tsc` - no Vite. We deploy it to Vercel via Git, and this time the host has to run a build.

### The files

```text
p2-pokedex-ts/
  .gitignore
  package.json
  tsconfig.json
  vercel.json
  public/
    index.html
    style.css
    js/            <- created by tsc, never committed
  src/
    api.ts
    main.ts
```

The idea is a clean split. `src/` is what you write. `public/` is what the host serves. `tsc` reads from `src/` and writes into `public/js/`.

Start with a new, empty folder. Copy `index.html` and `style.css` from project 1 into `public/` - they don't change at all. The script tag still says `js/main.js`, because that is where the compiled file will end up.

Create `package.json` and install TypeScript and live-server:

```bash
npm init -y
npm install -D typescript live-server
```

Then edit `package.json` so it has these scripts and devDependencies (your version numbers may differ, and any extra fields from `npm init -y` can stay):

```json
{
  "name": "p2-pokedex-ts",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "live-server public",
    "build": "tsc",
    "watch": "tsc --watch"
  },
  "devDependencies": {
    "live-server": "^1.2.2",
    "typescript": "^7.0.2"
  }
}
```

Both packages are devDependencies: TypeScript is needed to build the site, and live-server to look at it locally, but the finished site in `public/` needs neither. `live-server public` serves the `public` folder instead of the project root, since that's what the host will serve too.

That version number is worth a comment. At the time of writing (September 2026), `npm install -D typescript` gives you **TypeScript 7**, while a new Vite project still installs **TypeScript 6**. Everything in this article works with both, but don't be surprised when your projects don't match.

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "es2023",
    "module": "esnext",
    "moduleResolution": "bundler",
    "lib": ["ES2023", "DOM"],
    "strict": true,
    "rootDir": "src",
    "outDir": "public/js"
  },
  "include": ["src"]
}
```

The two lines that matter most for deploying:

- `"outDir": "public/js"` - where the compiled `.js` files go.
- `"rootDir": "src"` - where the source files live. Since TypeScript 6 you have to set this explicitly. If you leave it out, `tsc` stops with an error, but it still writes the files - into `public/js/src/` instead, where `index.html` won't find them.

`.gitignore`:

```text
node_modules
public/js
```

Ignoring `public/js` is deliberate. Compiled files are generated from `src/`, so committing them would mean two copies of the same code that can drift apart. It also means the host must build the project itself - which is the whole point of this project.

`src/api.ts`:

```typescript
// 1. Types - only the parts of the response we actually use
export interface Pokemon {
  id: number;
  name: string;
  sprites: { front_default: string | null };
  types: { type: { name: string } }[];
}

// 2. Settings
const BASE_URL = 'https://pokeapi.co/api/v2';

// 3. Fetch one Pokemon by name or number
export async function getPokemon(
  nameOrNumber: string,
): Promise<Pokemon> {
  const query = nameOrNumber.trim().toLowerCase();

  // Only spaces would fetch the whole list instead of one Pokemon
  if (!query) {
    throw new Error('Type a name or a number first');
  }

  const response = await fetch(`${BASE_URL}/pokemon/${query}`);

  if (!response.ok) {
    throw new Error(`Could not find "${query}" (${response.status})`);
  }

  return response.json();
}
```

`src/main.ts`:

```typescript
import { getPokemon, type Pokemon } from './api.js';

// 1. Elements - the generic says what we expect, the ! says "trust me"
const form = document.querySelector<HTMLFormElement>('#search-form')!;
const input = document.querySelector<HTMLInputElement>('#search')!;
const output = document.querySelector<HTMLDivElement>('#output')!;

// 2. Rendering
function renderMessage(message: string): void {
  const paragraph = document.createElement('p');
  paragraph.textContent = message;
  output.replaceChildren(paragraph);
}

function renderPokemon(pokemon: Pokemon): void {
  const card = document.createElement('article');
  const heading = document.createElement('h2');
  const image = document.createElement('img');
  const types = document.createElement('p');

  heading.textContent = `#${pokemon.id} ${pokemon.name}`;
  image.src = pokemon.sprites.front_default ?? '';     // null -> empty
  image.alt = pokemon.name;
  types.textContent = pokemon.types
    .map((entry) => entry.type.name)
    .join(', ');

  card.append(heading, image, types);
  output.replaceChildren(card);
}

// 3. Search
form.addEventListener('submit', async (event) => {
  event.preventDefault();
  renderMessage('Loading...');

  try {
    const pokemon = await getPokemon(input.value);
    renderPokemon(pokemon);
  } catch (error) {
    // In strict mode, error is "unknown" - check before using it
    const message =
      error instanceof Error ? error.message : 'Unknown error';
    renderMessage(message);
  }
});
```

Compare this with project 1. The logic is identical. TypeScript made us deal with two things the JS version silently ignored: `front_default` can be `null` (the JS version had a comment warning about it, the TS version won't compile without handling it), and `error` in a `catch` could be anything, not necessarily an `Error`.

And look at the import: `'./api.js'`, not `'./api.ts'`. That looks wrong, but it's right. `tsc` does not change import paths when it compiles. The browser will be loading `public/js/main.js`, which asks for `./api.js` - so that is what the import has to say. TypeScript understands that `./api.js` refers to `api.ts` while checking types.

### Run it locally

You need two terminals: one compiling, one serving.

```bash
# Terminal 1 - recompile every time you save a .ts file
npm run watch

# Terminal 2 - serve the public folder
npm run dev
```

The two work nicely together: when you save a `.ts` file, `tsc` writes a new `.js` file into `public/js`, live-server notices that something in `public` changed, and the browser reloads.

Search for `pikachu` again. Then look in `public/js/` - you'll find `main.js` and `api.js`, with all the types gone.

### Deploy: Vercel via Git

`vercel.json` tells Vercel how to build and what to serve:

```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "public"
}
```

You can type the same values into Vercel's dashboard instead, but a file in the repo is easier to share and harder to forget.

1. Push the project to a new GitHub repository.
2. In Vercel, add a new project and import the repository from GitHub.
3. Vercel reads `vercel.json` and shows your build command and output directory in the settings. Leave the framework preset as "Other".
4. Deploy. You get a URL like `something.vercel.app`.

Look at the build log while it runs. Vercel runs `npm install` (which installs TypeScript on Vercel's machine), then `npm run build` (which runs `tsc`), then serves `public/` - which now contains the `js/` folder that doesn't exist in your repo.

**What would happen without the build?** The host would serve `public/` without `js/`. The browser asks for `js/main.js`, gets a 404, and the page loads with no JavaScript at all. Clicking Search then does what a form does without JavaScript: it reloads the page with `?search=pikachu` in the URL. If you ever see that on a deployed site, check the build before you check your code.

<details>
<summary>Rabbit hole: can I write './api.ts' in my imports instead?</summary>

Yes, with one extra compiler option (available since TypeScript 5.7):

```json
"rewriteRelativeImportExtensions": true
```

With that option on, you can write `import { getPokemon } from './api.ts';`, and `tsc` rewrites it to `'./api.js'` in the output. Without it, `'./api.ts'` is a compile error.

We stick with `.js` in this article because it shows what is actually going on: the browser needs the name of the file that will exist after compiling. Vite projects write `.ts` for a different reason - Vite bundles everything into one file, so the import paths in the final output don't matter.

</details>

### Exercise 2

Predict first, then test. For each change below, will `tsc` complain, will the browser complain, or both? Undo each change before trying the next one.

1. Change the import in `main.ts` to `'./api'` (no extension).
2. Change it to `'./api.ts'`.
3. Remove `"rootDir": "src"` from `tsconfig.json`, delete `public/js`, and run `npm run build`.

<details>
<summary>Solution</summary>

1. **`'./api'` - only the browser complains.** With `"moduleResolution": "bundler"`, TypeScript happily finds `api.ts` without an extension, so `tsc` compiles with no errors. But the compiled `main.js` still says `'./api'`, and there is no file with that name. The browser gets a 404 for `/js/api`, and the module fails to load. This is the nastiest of the three: the compiler said everything was fine.
2. **`'./api.ts'` - `tsc` complains** with error TS5097: an import path can only end in `.ts` when `allowImportingTsExtensions` is enabled. Nothing reaches the browser. (See the rabbit hole above for the option that makes this work.)
3. **Missing `rootDir` - `tsc` complains, and the browser too.** TypeScript 6 and 7 report error TS5011 and ask you to set `rootDir`. But it still writes the output, into `public/js/src/main.js`. The browser asks for `public/js/main.js`, gets a 404, and nothing works.

The lesson: a successful compile does not mean the site works. The only real test is loading the page.

</details>

---

## Projects 3 and 4: Enter Vite

Doing this by hand with `tsc` works, but you have seen the fiddly bits: import extensions, `rootDir`, two terminals. Vite takes care of all of those. It still compiles TypeScript - it just hides the step.

### Scaffold and clean up

Do this once for each of projects 3 and 4 (and again for project 5 later):

```bash
npm create vite@latest p3-rick-and-morty -- --template vanilla-ts --no-immediate
cd p3-rick-and-morty
npm install
```

The `--template vanilla-ts` part picks Vanilla + TypeScript without asking. `--no-immediate` stops Vite from asking "Install with npm and start now?" at the end. If you leave it out and answer yes, it installs and starts the demo page straight away - stop it with `Ctrl + C` and carry on from `cd`. The starter project is a demo page with a counter, logos and links. Remove everything that belongs to the demo:

```bash
rm -r src/counter.ts src/assets public/icons.svg
```

Use bash (not PowerShell).

Keep `public/favicon.svg` if you like it, or replace it with your own. Then replace the contents of `index.html`, `src/main.ts` and `src/style.css` with the project code below. (Until you do, `npm run dev` shows errors, because the old `main.ts` still imports the files you just deleted.)

What's left is small: `index.html`, `package.json`, `tsconfig.json`, `.gitignore`, `src/main.ts` and `src/style.css`.

If you have done the Vite lessons in Module 1, a couple of things may look different:

- There is **no `src/vite-env.d.ts`** any more. The same job is now done by `"types": ["vite/client"]` inside `tsconfig.json`.
- There is **no `"strict": true`** in `tsconfig.json`. You still get strict mode: since TypeScript 6, strict is the default. Try adding a function parameter without a type - `tsc` will complain.

Tools change faster than course material. When the two disagree, what's on your screen wins - and it's worth figuring out why.

### How Vite handles TypeScript

Open `package.json`:

```json
"scripts": {
  "dev": "vite",
  "build": "tsc && vite build",
  "preview": "vite preview"
}
```

- `npm run dev` starts a development server. When the browser asks for `/src/main.ts`, Vite strips the types on the spot and sends back JavaScript. That is why `index.html` can point at a `.ts` file during development. There is still a compile step - it just happens one file at a time, as the browser asks.
- `npm run build` does the two jobs from earlier, in order. `tsc` checks the types. It writes nothing, because `tsconfig.json` has `"noEmit": true`. Then `vite build` strips the types (without checking them), bundles everything, and writes the finished site to `dist/`. If `tsc` finds an error, the `&&` stops `vite build` from running at all.
- `npm run preview` serves `dist/` locally, so you can test the real production version before you deploy it.

`dist/` is in `.gitignore`, just like `public/js` was in project 2. Same reason, same consequence: someone has to run the build before the site can go live.

---

## Project 3: Rick and Morty characters

**Goal:** a paged grid of characters from the [Rick and Morty API](https://rickandmortyapi.com/), with Previous and Next buttons. Deployed with the Netlify CLI.

### The files

`index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" type="image/svg+xml" href="/favicon.svg">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rick and Morty characters</title>
    <script type="module" src="/src/main.ts"></script>
  </head>
  <body>
    <main>
      <h1>Rick and Morty characters</h1>
      <nav>
        <button id="previous" type="button" disabled>Previous</button>
        <span id="page-info"></span>
        <button id="next" type="button">Next</button>
      </nav>
      <ul id="characters"></ul>
    </main>
  </body>
</html>
```

`src/style.css`:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 60rem;
  margin: 2rem auto;
  padding: 0 1rem;
}

nav {
  display: flex;
  gap: 1rem;
  align-items: center;
}

#characters {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(10rem, 1fr));
  gap: 1rem;
  padding: 0;
  list-style: none;
}

#characters img {
  width: 100%;
}
```

`src/api.ts`:

```typescript
// 1. Types - only the parts of the response we actually use
export interface Character {
  id: number;
  name: string;
  status: 'Alive' | 'Dead' | 'unknown';
  species: string;
  image: string;
}

export interface CharacterPage {
  info: { pages: number; next: string | null; prev: string | null };
  results: Character[];
}

// 2. Settings
const BASE_URL = 'https://rickandmortyapi.com/api';

// 3. Fetch one page of characters
export async function getCharacters(
  page: number,
): Promise<CharacterPage> {
  const response = await fetch(`${BASE_URL}/character?page=${page}`);

  if (!response.ok) {
    throw new Error(`Could not load page ${page} (${response.status})`);
  }

  return response.json();
}
```

`src/main.ts`:

```typescript
import './style.css';
import { getCharacters, type Character } from './api.ts';

// 1. Elements
const list = document.querySelector<HTMLUListElement>('#characters')!;
const pageInfo = document.querySelector<HTMLSpanElement>('#page-info')!;
const previous =
  document.querySelector<HTMLButtonElement>('#previous')!;
const next = document.querySelector<HTMLButtonElement>('#next')!;

// 2. State
let currentPage = 1;

// 3. Rendering
function renderCharacter(character: Character): HTMLLIElement {
  const item = document.createElement('li');
  const image = document.createElement('img');
  const name = document.createElement('h2');
  const details = document.createElement('p');

  image.src = character.image;
  image.alt = character.name;
  image.loading = 'lazy';                 // Only load when visible
  name.textContent = character.name;
  details.textContent = `${character.species} - ${character.status}`;

  item.append(image, name, details);
  return item;
}

// 4. Load and show a page
async function showPage(page: number): Promise<void> {
  pageInfo.textContent = 'Loading...';

  try {
    const data = await getCharacters(page);
    currentPage = page;
    list.replaceChildren(...data.results.map(renderCharacter));
    pageInfo.textContent = `Page ${page} of ${data.info.pages}`;
    previous.disabled = data.info.prev === null;
    next.disabled = data.info.next === null;
  } catch (error) {
    pageInfo.textContent =
      error instanceof Error ? error.message : 'Unknown error';
  }
}

// 5. Events
previous.addEventListener('click', () => showPage(currentPage - 1));
next.addEventListener('click', () => showPage(currentPage + 1));

showPage(1);
```

Two Vite details to notice. The CSS is imported from JavaScript (`import './style.css'`) - Vite picks it up and turns it into a proper stylesheet in the build. And the imports end in `.ts`, the opposite of project 2, because Vite bundles everything into one file, so the import paths never reach the browser.

Run it with `npm run dev` and click through a few pages.

### Deploy: Netlify CLI

Add a `netlify.toml` to the project root, so the CLI knows how to build and what to upload:

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

Install the CLI once, and log in (a browser window opens so you can approve it):

```bash
npm install -g netlify-cli
netlify login
```

If the install fails with a permission error (`EACCES`), don't reach for `sudo`. Skip the global install and put `npx` in front instead: `npx netlify-cli login`, `npx netlify-cli deploy`, and so on. It does the same thing, it's just more to type.

Then, from the project folder:

```bash
netlify deploy
```

The first time, the CLI asks whether to link this folder to an existing Netlify project or create a new one - create a new one. Then it:

1. runs `npm run build` **on your machine**,
2. uploads the contents of `dist/`,
3. prints a **draft URL**.

A draft deploy is a preview with its own unique URL. It isn't secret - anyone with the link can open it - but it doesn't change your live site. Check that everything works, then publish it for real:

```bash
netlify deploy --prod
```

This builds again and deploys to your project's main URL.

**What just happened?** The build ran on your computer, and only the finished files went to Netlify. No GitHub repo was involved. That's handy for quick experiments, but it also means nothing redeploys when you change the code - you run `netlify deploy --prod` again every time.

You don't need to build locally on purpose: `netlify deploy` builds by default. If you ever want to upload a `dist/` you already built, use `netlify deploy --prod --no-build`.

### Exercise 3

The API can filter by status: `?page=1&status=dead` only returns dead characters. Add a dropdown with All, Alive, Dead and Unknown. Changing it should jump back to page 1. Deploy the new version to production.

<details>
<summary>Solution</summary>

In `index.html`, above the `<nav>`:

```html
      <label for="status">Status</label>
      <select id="status">
        <option value="">All</option>
        <option value="alive">Alive</option>
        <option value="dead">Dead</option>
        <option value="unknown">Unknown</option>
      </select>
```

In `src/api.ts`, give `getCharacters` an optional second parameter and build the query string with `URLSearchParams`, so the status is only added when one is picked:

```typescript
// 3. Fetch one page of characters, optionally filtered by status
export async function getCharacters(
  page: number,
  status = '',
): Promise<CharacterPage> {
  const params = new URLSearchParams({ page: String(page) });

  if (status) {
    params.set('status', status);           // Only when chosen
  }

  const response = await fetch(`${BASE_URL}/character?${params}`);

  if (!response.ok) {
    throw new Error(`Could not load page ${page} (${response.status})`);
  }

  return response.json();
}
```

In `src/main.ts`, grab the dropdown, pass its value along, and listen for changes:

```typescript
const status = document.querySelector<HTMLSelectElement>('#status')!;

// in showPage:
    const data = await getCharacters(page, status.value);

// with the other events:
status.addEventListener('change', () => showPage(1)); // Restart
```

Going back to page 1 matters: there are far fewer dead characters than characters in total, so if you were on page 30, it might not exist in the filtered list.

Then `netlify deploy --prod`. Since there is no Git link, this is the only way the change gets online.

</details>

---

## Project 4: Weather in Norway

**Goal:** pick a Norwegian city and see the current temperature and wind, plus a 7 day forecast. Data from [Open-Meteo](https://open-meteo.com/), which needs no key for non-commercial use. Deployed with the Vercel CLI.

Scaffold and clean up exactly as before, with the name `p4-weather`.

### The files

`index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" type="image/svg+xml" href="/favicon.svg">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather in Norway</title>
    <script type="module" src="/src/main.ts"></script>
  </head>
  <body>
    <main>
      <h1>Weather in Norway</h1>
      <label for="city">City</label>
      <select id="city"></select>
      <p id="current"></p>
      <table>
        <thead>
          <tr><th>Day</th><th>Min</th><th>Max</th></tr>
        </thead>
        <tbody id="forecast"></tbody>
      </table>
    </main>
  </body>
</html>
```

`src/style.css`:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
  padding: 0 1rem;
}

td,
th {
  padding: 0.25rem 1rem;
  text-align: left;
}
```

`src/cities.ts` - Open-Meteo works with coordinates, not city names, so we keep our own list:

```typescript
export interface City {
  name: string;
  latitude: number;
  longitude: number;
}

export const cities: City[] = [
  { name: 'Oslo', latitude: 59.91, longitude: 10.75 },
  { name: 'Bergen', latitude: 60.39, longitude: 5.32 },
  { name: 'Trondheim', latitude: 63.43, longitude: 10.39 },
  { name: 'Stavanger', latitude: 58.97, longitude: 5.73 },
  { name: 'Tromsø', latitude: 69.65, longitude: 18.96 },
];
```

`src/api.ts`:

```typescript
import type { City } from './cities.ts';

// 1. Types - Open-Meteo returns the daily values as parallel arrays
export interface Forecast {
  current_units: { temperature_2m: string; wind_speed_10m: string };
  current: { temperature_2m: number; wind_speed_10m: number };
  daily: {
    time: string[];
    temperature_2m_min: number[];
    temperature_2m_max: number[];
  };
}

// 2. Settings
const BASE_URL = 'https://api.open-meteo.com/v1/forecast';

// 3. Fetch current weather and a 7 day forecast for one city
export async function getForecast(city: City): Promise<Forecast> {
  const params = new URLSearchParams({
    latitude: String(city.latitude),
    longitude: String(city.longitude),
    current: 'temperature_2m,wind_speed_10m',
    daily: 'temperature_2m_min,temperature_2m_max',
    wind_speed_unit: 'ms',                             // m/s, not km/h
    timezone: 'auto',
  });
  const response = await fetch(`${BASE_URL}?${params}`);

  if (!response.ok) {
    throw new Error(`Could not load weather (${response.status})`);
  }

  return response.json();
}
```

`src/main.ts`:

```typescript
import './style.css';
import { cities } from './cities.ts';
import { getForecast, type Forecast } from './api.ts';

// 1. Elements
const select = document.querySelector<HTMLSelectElement>('#city')!;
const current =
  document.querySelector<HTMLParagraphElement>('#current')!;
const forecast =
  document.querySelector<HTMLTableSectionElement>('#forecast')!;

// 2. Fill the dropdown - the option value is the index in the array
cities.forEach((city, index) => {
  select.append(new Option(city.name, String(index)));
});

// 3. Rendering
function renderRow(date: string, min: string, max: string) {
  const row = document.createElement('tr');
  const day = new Date(date).toLocaleDateString('en-GB', {
    weekday: 'long',
    timeZone: 'UTC',                  // Dates only, no times
  });

  for (const text of [day, min, max]) {
    const cell = document.createElement('td');
    cell.textContent = text;
    row.append(cell);
  }

  return row;
}

function renderForecast(data: Forecast): void {
  const units = data.current_units;
  const { temperature_2m, wind_speed_10m } = data.current;
  const { time, temperature_2m_min, temperature_2m_max } = data.daily;

  current.textContent =
    `Now: ${temperature_2m} ${units.temperature_2m}, ` +
    `wind ${wind_speed_10m} ${units.wind_speed_10m}`;

  const rows = time.map((date, i) =>
    renderRow(
      date,
      `${temperature_2m_min[i]} ${units.temperature_2m}`,
      `${temperature_2m_max[i]} ${units.temperature_2m}`,
    ),
  );
  forecast.replaceChildren(...rows);
}

// 4. Load weather for the selected city
async function showWeather(): Promise<void> {
  const chosen = select.value;
  current.textContent = 'Loading...';

  try {
    const data = await getForecast(cities[Number(chosen)]);

    // Ignore the answer if another city was picked while we waited
    if (chosen === select.value) {
      renderForecast(data);
    }
  } catch (error) {
    if (chosen === select.value) {
      current.textContent =
        error instanceof Error ? error.message : 'Unknown error';
    }
  }
}

select.addEventListener('change', showWeather);
showWeather();
```

Three small things worth a look. The units (degrees Celsius and metres per second) come from the API response in `current_units` rather than being typed into our code. The `timeZone: 'UTC'` option is there because `new Date('2026-09-25')` means midnight UTC - without it, anyone west of Greenwich would see every day shifted back by one. And `showWeather` remembers which city it was asked for. If you scroll through the dropdown with the arrow keys, several requests are sent at once, and they don't always come back in order. Without the check, a slow answer for Oslo could arrive last and overwrite the weather for the city you actually picked.

Run it with `npm run dev` and switch between cities.

### Deploy: Vercel CLI

No config file this time. Vercel recognises a Vite project by itself.

```bash
npm install -g vercel
vercel login
```

The same trick works here if the install is refused: `npx vercel login`, `npx vercel`, `npx vercel --prod`.

Then, from the project folder:

```bash
vercel
```

The first run asks a few questions: whether to set up and deploy the folder (yes), which account to use, whether to link to an existing project (no), the project name, and which folder the code is in (`./`). Then it shows the settings it detected - the framework should say Vite, with `npm run build` and `dist`. Accept them.

Then it uploads your **source code**, builds it **on Vercel's machines**, and prints a **preview URL**. When you're happy:

```bash
vercel --prod
```

**What just happened?** Compare this with project 3. Both are CLI deploys from your own folder, but the Netlify CLI built on your machine and uploaded `dist/`, while the Vercel CLI uploaded your source and built in the cloud. Watch the output: you can see the Vercel build log scroll past, running `npm install` and `npm run build` on a machine that isn't yours.

In practice, both work. The difference shows up when something goes wrong. If a build works locally but fails on Vercel, the problem is usually in something that only exists on your machine - a file you never committed, or a different Node version.

<details>
<summary>Rabbit hole: building locally with the Vercel CLI</summary>

You can make Vercel work the way the Netlify CLI does:

```bash
vercel build --prod
vercel deploy --prebuilt --prod
```

`vercel build` runs the build on your machine and saves the result in a `.vercel/output` folder. `--prebuilt` uploads that folder instead of your source code. This is mostly useful in automated pipelines where the build has already run somewhere else, but it shows that "where does the build happen?" is a choice, not a fixed rule.

</details>

### Exercise 4

Add a **Rain** column to the forecast, showing the day's total precipitation with its unit, and add one more city of your choice. Deploy the new version to production.

Hint: look in the [Open-Meteo docs](https://open-meteo.com/en/docs) for the daily variable, and remember the units come back in their own object.

<details>
<summary>Solution</summary>

The daily variable is `precipitation_sum`, and its unit (`mm`) comes back in `daily_units`.

`src/cities.ts` - add a line, for example:

```typescript
  { name: 'Bodø', latitude: 67.28, longitude: 14.4 },
```

`index.html` - one more header cell:

```html
<tr><th>Day</th><th>Min</th><th>Max</th><th>Rain</th></tr>
```

`src/api.ts` - ask for the new variable, and add it to the type:

```typescript
export interface Forecast {
  current_units: { temperature_2m: string; wind_speed_10m: string };
  current: { temperature_2m: number; wind_speed_10m: number };
  daily_units: { precipitation_sum: string };
  daily: {
    time: string[];
    precipitation_sum: number[];
    temperature_2m_min: number[];
    temperature_2m_max: number[];
  };
}

// in getForecast:
    daily: 'temperature_2m_min,temperature_2m_max,precipitation_sum',
```

`src/main.ts` - let `renderRow` take any number of values, instead of adding a fourth parameter (a rest parameter collects them into an array):

```typescript
function renderRow(date: string, ...values: string[]) {
  // ...
  for (const text of [day, ...values]) {
```

And pass the rain value in `renderForecast`:

```typescript
  const rainUnit = data.daily_units.precipitation_sum;
  const {
    time,
    temperature_2m_min,
    temperature_2m_max,
    precipitation_sum,
  } = data.daily;

  // ...

  const rows = time.map((date, i) =>
    renderRow(
      date,
      `${temperature_2m_min[i]} ${units.temperature_2m}`,
      `${temperature_2m_max[i]} ${units.temperature_2m}`,
      `${precipitation_sum[i]} ${rainUnit}`,
    ),
  );
```

Finally `vercel --prod`. Like project 3, there is no Git link, so nothing happens until you deploy by hand.

</details>

---

## Project 5: Weather search with an API key

**Goal:** type any city name and get the current weather from [OpenWeatherMap](https://openweathermap.org/current). Unlike Open-Meteo, this API needs a key - the one you registered at the start. Deployed to Netlify via Git.

This project is about the key more than the weather, so read this part first.

### Why keys go in environment variables

An **environment variable** is a named value that comes from outside your code - from the machine or service running it. Instead of writing the key into a `.ts` file, you write it in a separate file (or a host's settings page), and your code only refers to it by name.

This is standard practice for good reasons:

- **Keys stay out of Git.** Anything you commit stays in the repository's history, even if you delete it later. Bots scan public GitHub repos for keys all day, every day.
- **Different environments, different values.** Your laptop, a test site and the live site can each use their own key without any change to the code.
- **Keys can be changed without touching code.** If a key leaks, you create a new one and update one setting.

### Why this is a problem with Vite

Here's the catch. Environment variables were invented for servers, where the code stays on the server. A front-end build has no server: everything ends up in files that are sent to every visitor's browser.

Vite deals with this very literally. Anything you read with `import.meta.env.VITE_SOMETHING` is **replaced with its actual value at build time**. The finished JavaScript in `dist/` doesn't contain "read the key from somewhere" - it contains the key itself, as a plain string.

That gives us three practical consequences:

- **The `VITE_` prefix does not mean "secret".** It means the opposite. Vite only exposes variables with this prefix *because* they end up public, so you have to opt in on purpose.
- **Changing the value requires a new build.** The value is baked in. Changing it on the host does nothing until the site is built again.
- **Environment variables keep the key out of your repository, not out of your site.** Anyone can open DevTools and read it.

So is project 5 a bad idea? For a free, rate-limited weather key, the worst case is that somebody uses up your free quota - annoying, not dangerous. That's an acceptable trade-off for a learning project. **Never do this with a key that costs money, can change data, or gives access to anything private.** The proper fix is to keep the key on a server and let the front end ask the server instead. That's a topic for later.

### Scaffold and set up the key

Scaffold and clean up as before, with the name `p5-weather-key`. Then create two files in the project root.

`.env.example` - committed to Git, so others know which variables the project needs:

```text
VITE_OPENWEATHER_API_KEY=paste_your_key_here
```

`.env.local` - your real key, **never committed**:

```text
VITE_OPENWEATHER_API_KEY=your_actual_key
```

Why `.env.local` and not just `.env`? Look at Vite's `.gitignore`: it contains `*.local`, but not `.env`. A key in `.env` gets committed the next time you run `git add .` - exactly what we're trying to avoid. Vite reads both files, so use the one Git ignores.

Run `git status` to check. `.env.example` should show up, `.env.local` should not.

`src/env.d.ts` - this tells TypeScript which `VITE_` variables exist, so you get autocompletion, and the key gets the type `string` instead of `any`. It won't catch a misspelled name, though: Vite allows any `VITE_` name, so a typo just gives you `undefined` at runtime. That's what the "No API key found" check in `api.ts` is for:

```typescript
// Tell TypeScript which VITE_ variables exist
interface ImportMetaEnv {
  readonly VITE_OPENWEATHER_API_KEY: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### The files

`index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" type="image/svg+xml" href="/favicon.svg">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Weather search</title>
    <script type="module" src="/src/main.ts"></script>
  </head>
  <body>
    <main>
      <h1>Weather search</h1>
      <form id="search-form">
        <label for="city">City</label>
        <input id="city" name="city" required>
        <button>Search</button>
      </form>
      <p id="output"></p>
    </main>
  </body>
</html>
```

`src/style.css`:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
  padding: 0 1rem;
}
```

`src/api.ts`:

```typescript
// 1. Types - only the parts of the response we actually use
export interface Weather {
  name: string;
  main: { temp: number; feels_like: number };
  weather: { description: string }[];
}

// 2. Settings - the key comes from .env.local, not from the code
const BASE_URL = 'https://api.openweathermap.org/data/2.5/weather';
const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;

// 3. Fetch the current weather for a city
export async function getWeather(city: string): Promise<Weather> {
  if (!API_KEY) {
    throw new Error('No API key found - see .env.example');
  }

  const params = new URLSearchParams({
    q: city.trim(),
    units: 'metric',
    appid: API_KEY,
  });
  const response = await fetch(`${BASE_URL}?${params}`);

  // 401 means the key was refused - new keys take a while to activate
  if (response.status === 401) {
    throw new Error('API key not accepted (401) - is it active yet?');
  }

  if (!response.ok) {
    throw new Error(`No weather for "${city}" (${response.status})`);
  }

  return response.json();
}
```

`src/main.ts`:

```typescript
import './style.css';
import { getWeather } from './api.ts';

// 1. Settings - an escape keeps the degree sign out of the source file
const DEGREES = '\u00B0C';

// 2. Elements
const form = document.querySelector<HTMLFormElement>('#search-form')!;
const input = document.querySelector<HTMLInputElement>('#city')!;
const output = document.querySelector<HTMLParagraphElement>('#output')!;

// 3. Search
form.addEventListener('submit', async (event) => {
  event.preventDefault();
  output.textContent = 'Loading...';

  try {
    const data = await getWeather(input.value);
    const temp = Math.round(data.main.temp);
    const feelsLike = Math.round(data.main.feels_like);
    const description = data.weather[0]?.description ?? '';

    output.textContent =
      `${data.name}: ${temp} ${DEGREES} ` +
      `(feels like ${feelsLike} ${DEGREES}), ${description}`;
  } catch (error) {
    output.textContent =
      error instanceof Error ? error.message : 'Unknown error';
  }
});
```

Run it with `npm run dev`. If you get `401`, your key isn't active yet (or there's a typo in `.env.local`). If you get `404`, OpenWeatherMap doesn't know that city. Vite only reads `.env.local` when it starts, so restart `npm run dev` after changing the file.

### See the problem for yourself

Build the project and search the output for your key:

```bash
npm run build
grep -r "your_actual_key" dist
```

(Use your real key in the command, and bash, not PowerShell.)

There it is, in plain text, in a file you're about to put on the internet.

### Deploy: Netlify via Git

Add the same `netlify.toml` as in project 3:

```toml
[build]
  command = "npm run build"
  publish = "dist"
```

1. Push to a new GitHub repository. Check on GitHub that `.env.local` is **not** there.
2. In Netlify, import the repository as in project 1. The build settings are read from `netlify.toml`.
3. Before (or right after) the first deploy, open the project's **environment variables** settings and add `VITE_OPENWEATHER_API_KEY` with your key as its value. If Netlify offers to mark it as containing a secret value, do so - it is one.
4. Trigger a new deploy from the project's deploys page, so the build runs with the variable available. Adding or changing a variable does nothing to a site that's already built - Vite baked the old value (or nothing) into it.

Your `.env.local` never left your machine, so the host needs its own copy of the key. That's the point: the repository works for anyone, with their own key, and only the host holds yours.

**Don't be surprised if the build fails.** Netlify scans the build output for the values of variables marked as secret, and with Vite, your key is right there in `dist/`. When it finds it, it stops the deploy and points at the file. The host is telling you what the previous section said: this key is not secret any more.

For this project, you have decided to accept that (see the reasoning above), so you can tell Netlify it's intentional. Add this to `netlify.toml`, commit and push:

```toml
[build.environment]
  SECRETS_SCAN_OMIT_KEYS = "VITE_OPENWEATHER_API_KEY"
```

This switches off the scan for that one variable only. Everything else is still checked. Make that decision consciously each time - never add it just to make a red build go green.

### Exercise 5

Find your key on the live site. Use only the browser - no terminal, no access to the repository.

<details>
<summary>Solution</summary>

There are at least two ways:

1. **Network tab.** Open DevTools, go to the Network tab, and search for a city. Click the request to `api.openweathermap.org`: the key is right there in the URL, as the `appid` parameter.
2. **Sources (Debugger in Firefox).** Open the JavaScript file under `assets/` and search it for `appid` or for part of your key. Minified or not, the value is in the bundle as a plain string.

The first one shows that hiding it in the bundle wouldn't help anyway: to use the key, the browser has to send it, and anything the browser sends is visible to the person using the browser. That's why the real fix has to involve a server.

</details>

---

## Summary

| # | Build | Where the build runs | Redeploys on push? |
|---|---|---|---|
| 1 | None | Nowhere - files are served as they are | Yes |
| 2 | `tsc` | On Vercel | Yes |
| 3 | `tsc && vite build` | On your machine, then `dist/` is uploaded | No - run `netlify deploy --prod` |
| 4 | `tsc && vite build` | On Vercel, from uploaded source | No - run `vercel --prod` |
| 5 | `tsc && vite build` | On Netlify, with the key from its settings | Yes |

The one question to ask for any project: **what does the browser need, and who produces it?** If the answer involves a build, make sure the build runs somewhere, and that the host serves the folder it writes to.

---

## Self-study: It works on my machine

A classmate built a tiny TypeScript app that shows a random dog picture from the [Dog CEO API](https://dog.ceo/dog-api/). It works perfectly on their laptop. They pushed it to GitHub and connected it to Netlify, leaving all build settings in the dashboard empty so `netlify.toml` would decide.

The deployed site loads, but it's unstyled, and clicking the button does nothing.

Recreate the project from the files below, and check that it works locally (`npm install`, `npm run build`, `npm run dev`). Then deploy it to Netlify via Git and find out why it breaks. There are **two** separate problems.

When you import the repository, Netlify may fill in a build command for you. Clear the field, so it's empty like your classmate's - otherwise Netlify quietly fixes one of the problems before you find it.

```text
dog-button/
  .gitignore
  netlify.toml
  package.json
  tsconfig.json
  public/
    index.html
    style.css
  src/
    api.ts
    main.ts
```

`.gitignore`:

```text
node_modules
public/js
```

`netlify.toml`:

```toml
[build]
  publish = "public"
```

`package.json`:

```json
{
  "name": "dog-button",
  "private": true,
  "type": "module",
  "scripts": {
    "dev": "live-server public",
    "build": "tsc",
    "watch": "tsc --watch"
  },
  "devDependencies": {
    "live-server": "^1.2.2",
    "typescript": "^7.0.2"
  }
}
```

`tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "es2023",
    "module": "esnext",
    "moduleResolution": "bundler",
    "lib": ["ES2023", "DOM"],
    "strict": true,
    "rootDir": "src",
    "outDir": "public/js"
  },
  "include": ["src"]
}
```

`public/index.html`:

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dog button</title>
    <link rel="stylesheet" href="Style.css">
    <script type="module" src="js/main.js"></script>
  </head>
  <body>
    <main>
      <h1>Dog button</h1>
      <button id="fetch-dog" type="button">Show me a dog</button>
      <img id="dog" alt="">
    </main>
  </body>
</html>
```

`public/style.css`:

```css
body {
  font-family: system-ui, sans-serif;
  max-width: 40rem;
  margin: 2rem auto;
  padding: 0 1rem;
  background: #fdf6e3;
}

#dog {
  display: block;
  max-width: 100%;
  margin-top: 1rem;
}
```

`src/api.ts`:

```typescript
// 1. Types and settings
interface DogResponse {
  message: string;                  // The image URL
  status: string;
}

const API_URL = 'https://dog.ceo/api/breeds/image/random';

// 2. Fetch a random dog image URL
export async function getRandomDog(): Promise<string> {
  const response = await fetch(API_URL);

  if (!response.ok) {
    throw new Error(`No dog today (${response.status})`);
  }

  const data: DogResponse = await response.json();
  return data.message;
}
```

`src/main.ts`:

```typescript
import { getRandomDog } from './api.js';

// 1. Elements
const button = document.querySelector<HTMLButtonElement>('#fetch-dog')!;
const image = document.querySelector<HTMLImageElement>('#dog')!;

// 2. Show a new dog on every click
button.addEventListener('click', async () => {
  button.disabled = true;

  try {
    image.src = await getRandomDog();
    image.alt = 'A random dog';
  } catch (error) {
    image.alt =
      error instanceof Error ? error.message : 'Unknown error';
  } finally {
    button.disabled = false;
  }
});
```

Start with the deploy log in Netlify and the Network tab on the live site. For each problem, explain why it didn't show up locally, then fix it and deploy again.

<details>
<summary>Hint</summary>

Look at every 404 in the Network tab. For each one, ask: does this file exist in the repository? Does it exist in what Netlify served? And is the name *exactly* the same?

</details>

<details>
<summary>Solution</summary>

**Problem 1: nothing builds the TypeScript.** `netlify.toml` tells Netlify what to publish, but not how to build it, and the dashboard settings were left empty. So Netlify publishes `public/` as it is in the repository - and `public/js` is in `.gitignore`, so it doesn't exist there. The browser asks for `js/main.js` and gets a 404, which is why the button does nothing. The deploy log gives it away: there is no build step in it at all.

Locally it worked because your classmate had run `npm run build`, so `public/js` existed on their machine. It just never left it.

Fix - tell Netlify how to build:

```toml
[build]
  command = "npm run build"
  publish = "public"
```

Netlify installs dev dependencies by default, so TypeScript is available for the build.

**Problem 2: `Style.css` is not `style.css`.** The link says `Style.css` with a capital S, but the file is `style.css`. On macOS and Windows, the file system ignores case by default, so live-server found the file anyway. Netlify's servers run Linux, where `Style.css` and `style.css` are two different names. One of them doesn't exist, so the stylesheet is a 404 and the page is unstyled.

Fix - make the link match the file name exactly:

```html
<link rel="stylesheet" href="style.css">
```

A good habit: use lowercase for all file names, always. Then this whole category of bugs can't happen.

**Why both only showed up after deploying:** in each case, your machine was more forgiving than the host - one had leftover build output, the other ignored letter case. That's what "it works on my machine" usually means: the machine is part of the reason it works.

</details>
