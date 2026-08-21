# Unpacking Data: Destructuring and the Array Toolkit

**Time:** roughly 90 minutes of reading and coding, plus the self-study task  
**Prerequisites:** objects, arrays, functions, arrow functions, `map` / `filter` / `find` from JS1

---

## How this lesson works

Most lessons on this topic show you a syntax, then an example of that syntax. This one runs the other way round. We start with **one messy dataset and a list of jobs to do with it**. Each job is written first in the plain way you already know, then rewritten with the new syntax. You will see the pain before you see the cure, which is the only reliable way to remember why a feature exists.

**The main thread is vanilla JavaScript.** Open a `.js` file, paste the dataset in, and run each snippet as you read. Do not skip the running part.

TypeScript appears only inside folded blocks marked *TypeScript version*, and it is optional. You can read the whole lesson, do every exercise and finish the self-study task without opening a single one. They are there because your course runs on TypeScript and you will want them on the second pass, not the first. They also do not simply repeat the example with types bolted on. Each one covers a whole section at once and ends with a short list of what actually changes, because in practice roughly four things change and the rest is identical. Learn the pattern first; add the types when the pattern is automatic.

One practical warning if you paste the TypeScript snippets into a Vite `vanilla-ts` project: that scaffold enables `noUnusedLocals` and `noUnusedParameters`, so any snippet that creates a variable it never logs will be underlined in red. That is a tidiness rule, not a type error. Add a `console.log`, or switch those two options off in `tsconfig.json` while you are experimenting.

---

## The dataset

You volunteer for a local ornithology club. Their field app exports observations as JSON. Somebody has to turn that export into a readable summary page, and that somebody is you.

```js
// The raw export from the club's field app
const outings = [
  {
    id: 'out-118',
    site: { name: 'Utsira', county: 'Rogaland', position: [59.3072, 4.8836] },
    observer: { first_name: 'Ingrid', years_active: 12 },
    conditions: { wind_ms: 6 },
    species: [
      { name: 'Gannet', count: 42, ringed: 0 },
      { name: 'Eider', count: 18, ringed: 3 },
      { name: 'Turnstone', count: 5, ringed: 5 },
    ],
  },
  {
    id: 'out-119',
    site: { name: 'Lista', county: 'Agder', position: [58.1094, 6.5686] },
    observer: { first_name: 'Bjorn', years_active: 3 },
    conditions: { wind_ms: 2, temperature_c: 9 },
    species: [
      { name: 'Dunlin', count: 60, ringed: 12 },
      { name: 'Sanderling', count: 9, ringed: 1 },
    ],
  },
  {
    id: 'out-120',
    site: { name: 'Jaerstranda', county: 'Rogaland', position: [58.7663, 5.5343] },
    observer: { first_name: 'Marit', years_active: 7 },
    conditions: { wind_ms: 11, temperature_c: 4 },
    species: [
      { name: 'Oystercatcher', count: 14, ringed: 2 },
      { name: 'Gannet', count: 7, ringed: 0 },
      { name: 'Eider', count: 21, ringed: 0 },
    ],
  },
];
```

Three things about this data are deliberately annoying, and all three are things you will meet in real APIs:

1. The values you want are buried two or three levels down.
2. `first_name` is snake_case, which clashes with the camelCase you write everywhere else.
3. The first outing has no `temperature_c`. The field is simply absent, not `null`.

<details>
<summary><strong>TypeScript version: typing the dataset</strong></summary>

```ts
interface SpeciesRecord {
  name: string;
  count: number;
  ringed: number;
}

interface Site {
  name: string;
  county: string;
  position: [number, number]; // tuple: exactly two numbers, latitude then longitude
}

interface Observer {
  first_name: string;
  years_active: number;
}

interface Conditions {
  wind_ms: number;
  temperature_c?: number; // optional - genuinely absent on out-118
}

interface Outing {
  id: string;
  site: Site;
  observer: Observer;
  conditions: Conditions;
  species: SpeciesRecord[];
}

const outings: Outing[] = [
  // ...exactly the same data as above
];
```

**How this differs from JS**

- The data is identical. Only the shape description is new, and it is erased before the code runs.
- `temperature_c?: number` is the honest way to model a field that may be missing. Its type becomes `number | undefined`, so TypeScript will not let you do arithmetic on it until you supply a default or check it.
- `position: [number, number]` is a **tuple**, not `number[]`. A tuple fixes the length and the meaning of each slot, which is exactly right for coordinates and is what makes array destructuring type-safe further down.
- Writing `const outings: Outing[]` means a typo in the data itself (`temp_c` instead of `temperature_c`) is caught the moment you save the file, not when the page renders blank.

</details>

---

## Job 1: a one-line headline per outing

The club wants a line like `Utsira (Rogaland), observed by Ingrid, 6 m/s, 12 C`.

**The plain way.** You already know how to do this:

```js
const outing = outings[0];

const siteName = outing.site.name;
const county = outing.site.county;
const observerName = outing.observer.first_name;
const wind = outing.conditions.wind_ms;
let temperature = outing.conditions.temperature_c;
if (temperature === undefined) {
  temperature = 12;
}
```

Six lines, the word `outing` written five times, and a manual `if` for the missing field. It works. It is just noisy, and noise hides bugs.

**The same thing, destructured:**

```js
const {
  site: { name: siteName, county },
  observer: { first_name: observerName },
  conditions: { wind_ms: wind, temperature_c: temperature = 12 },
} = outing;

console.log(`${siteName} (${county}), observed by ${observerName}, ${wind} m/s, ${temperature} C`);
// Utsira (Rogaland), observed by Ingrid, 6 m/s, 12 C
```

One statement, and it does three separate jobs at once.

### The mental model: the left side is a picture, not a path

This is the single idea that makes destructuring click.

`outing.site.name` is a **path**: go here, then here, then here. Destructuring is not a path. The left-hand side is a **picture of the shape you expect**, with variable names written where the values sit. Compare:

```js
// The data
{ site: { name: 'Utsira' } }

// The pattern
{ site: { name: siteName } }
```

The pattern has the same brackets in the same places. Wherever the data has a value, the pattern has a variable name.

Three tools fit inside that picture:

| Tool | Syntax | Reads as |
|---|---|---|
| Rename | `first_name: observerName` | "take `first_name`, call it `observerName`" |
| Default | `temperature_c = 12` | "if it is `undefined`, use 12 instead" |
| Both | `temperature_c: temp = 12` | "take `temperature_c`, call it `temp`, default 12" |

Note the order in the combined form: **source, colon, new name, equals, default**. That order never changes.

### Traps worth knowing now

**Drilling does not give you the container.** This surprises nearly everyone:

```js
const { site: { name } } = outing;
console.log(name);  // 'Utsira'
console.log(site);  // ReferenceError: site is not defined
```

The colon means "go inside". It does not create a `site` variable. If you want both, ask for both: `const { site, site: { name } } = outing;`

**Renaming removes the old name.** After `const { first_name: observerName } = outing.observer`, there is no `first_name` variable. Only `observerName`.

**Defaults only fire on `undefined`.** Not `null`, not `0`, not `''`, not `false`.

```js
const { temperature_c = 12 } = { temperature_c: null };
console.log(temperature_c); // null, not 12
```

If a field can arrive as `null`, a default will not save you. Use `??` after the fact: `const temp = temperature_c ?? 12;`

**Destructuring `undefined` throws.** This is the one that will actually break your page:

```js
const { name } = undefined; // TypeError: Cannot destructure property 'name' of 'undefined'
```

The fix is a default on the whole pattern, which is why you will constantly see `= {}` in function parameters:

```js
/**
 * Builds a short label for a site.
 * @param {object} [site] - The site object, may be missing entirely.
 * @returns {string} A human-readable label.
 */
function siteLabel({ name = 'Unknown site', county = 'Unknown county' } = {}) {
  return `${name} (${county})`;
}

console.log(siteLabel(outings[0].site)); // 'Utsira (Rogaland)'
console.log(siteLabel());                // 'Unknown site (Unknown county)'
```

<details>
<summary><strong>TypeScript version: Job 1</strong></summary>

```ts
const outing: Outing = outings[0];

// Identical to the JavaScript. No annotations belong inside the pattern.
const {
  site: { name: siteName, county },
  observer: { first_name: observerName },
  conditions: { wind_ms: wind, temperature_c: temperature = 12 },
} = outing;

// siteName: string, county: string, wind: number
// temperature: number - NOT number | undefined, because the default removes it

/**
 * Builds a short label for a site.
 * @param site - Any object with an optional name and county.
 */
function siteLabel({ name = 'Unknown site', county = 'Unknown county' }: Partial<Site> = {}): string {
  return `${name} (${county})`;
}

console.log(siteLabel(outings[0].site)); // 'Utsira (Rogaland)'
console.log(siteLabel());                // 'Unknown site (Unknown county)'
```

**How this differs from JS**

- **Do not annotate inside a destructuring pattern.** `const { name: string } = site;` is not a type annotation - it renames `name` to a variable called `string`. The colon already means "rename". If you must annotate, annotate the whole pattern from outside: `const { a, b }: { a: number; b: string } = obj;`
- In a parameter, the order is **pattern, then annotation, then default**: `function f({ a }: Options = {}) {}`. Getting that order wrong is the most common TypeScript syntax error in this lesson.
- `Partial<Site>` is a utility type that makes every property of `Site` optional, which is exactly what "this object might be missing bits" means. A full `Site` is still accepted.
- Defaults narrow the type. Because of `= 12`, `temperature` is `number`, and TypeScript stops nagging you about `undefined`.
- The `null` trap is visible at compile time. If a field is typed `number | null`, the default does **not** strip `null` from the type, so TypeScript makes you deal with it. This is one of the clearest cases where the type system catches a runtime bug for you.
- The crash from destructuring `undefined` also moves to compile time. Given `let maybe: Site | undefined`, writing `const { name } = maybe;` is an error before the page ever loads.

</details>

---

## Job 2: split the coordinates

`position` is `[59.3072, 4.8836]`. Latitude first, longitude second, by convention. Positional data like this is exactly what array destructuring is for.

```js
const { site: { position: [latitude, longitude] } } = outings[0];
console.log(latitude, longitude); // 59.3072 4.8836
```

Objects match **by name**. Arrays match **by position**. That is the whole difference. Which means with arrays you sometimes need to skip:

```js
// Imagine the club later adds altitude in the middle: [lat, altitude, lon]
const [lat, , lon] = [59.3072, 12, 4.8836];
console.log(lat, lon); // 59.3072 4.8836
```

The lonely comma is a hole. One comma without a name skips one element. Use it sparingly, because `[, , , x]` is unreadable and a comment will not save it.

### Rest: "and everything else"

The `...` operator, on the left of an `=`, gathers whatever is left over.

```js
const [firstSpecies, ...otherSpecies] = outings[0].species;
console.log(firstSpecies.name);     // 'Gannet'
console.log(otherSpecies.length);   // 2
```

It works in objects too, and there it is genuinely useful. Say the summary page must never show the `ringed` numbers:

```js
const { ringed, ...publicRecord } = outings[0].species[1];
console.log(publicRecord); // { name: 'Eider', count: 18 }
```

You have created a copy without one field, without writing a loop. Two rules: rest must come **last**, and there can only be **one** of it.

### `at()`: the tidy way to reach the end

```js
const lastSpecies = outings[0].species.at(-1);
console.log(lastSpecies.name); // 'Turnstone'
```

`at(-1)` replaces `arr[arr.length - 1]`. Negative indices count backwards. Positive ones work exactly like brackets.

<details>
<summary><strong>TypeScript version: Job 2</strong></summary>

```ts
// position is a tuple, so both variables are number, not number | undefined
const { site: { position: [latitude, longitude] } } = outings[0];

// Skipping works the same. The tuple type documents what the hole is.
const withAltitude: [number, number, number] = [59.3072, 12, 4.8836];
const [lat, , lon] = withAltitude;

// Array rest: SpeciesRecord, then SpeciesRecord[]
const [firstSpecies, ...otherSpecies] = outings[0].species;

// Object rest: TypeScript works out the leftover shape by itself.
// publicRecord is { name: string; count: number }, i.e. Omit<SpeciesRecord, 'ringed'>
const { ringed, ...publicRecord } = outings[0].species[1];

// at() is honest about the end of the array
const lastSpecies: SpeciesRecord | undefined = outings[0].species.at(-1);
console.log(lastSpecies?.name); // 'Turnstone'
```

**How this differs from JS**

- **Tuples make array destructuring safe.** With `position: [number, number]`, `latitude` and `longitude` are plain `number`. Had you typed it `number[]`, the length is unknown, and if your `tsconfig.json` has `noUncheckedIndexedAccess` enabled you would get `number | undefined` instead. Use a tuple whenever position carries meaning.
- **Object rest is a free `Omit`.** You do not annotate `publicRecord`; TypeScript computes the remaining shape. That is the same result as `Omit<SpeciesRecord, 'ringed'>` from the utility types lesson, just derived automatically.
- **`at()` returns `T | undefined`**, always, even with a tuple. TypeScript forces the `?.` or a check. In plain JavaScript, the same code silently produces `undefined` and blows up one line later.
- `at()` needs `"lib"` to include `ES2022` in `tsconfig.json`. The default Vite scaffold already does.

</details>

---

## Job 3: a row of text per species

Now we need the array methods, and this is where destructuring stops being a party trick and starts earning its keep.

### Refresher: choose the method by what you want back

Everything from JS1, in one table:

| You want... | Method | You get back |
|---|---|---|
| every item, changed | `map` | a **new array, same length** |
| only the items that qualify | `filter` | a **new array, same or shorter** |
| the first item that qualifies | `find` | **the item itself**, or `undefined` |
| where that item is | `findIndex` | a **number**, or `-1` |
| a yes or no answer | `some` / `every` | a **boolean** |
| anything else at all | `reduce` | **whatever you build** |

Two mistakes to unlearn:

- `filter(...)[0]` when you mean `find(...)`. `filter` walks the whole array and builds a throwaway array. `find` stops at the first hit and hands you the object.
- `map` when you mean `forEach`. If you are not using the returned array, you have built one for nothing. `map` is for transforming, `forEach` is for side effects.

### Destructuring in the callback parameter

A callback receives one item at a time. That item is an object. So the parameter slot is a pattern slot:

```js
const species = outings[0].species;

// Without destructuring
const rowsA = species.map((s) => `${s.name}: ${s.count}`);

// With destructuring in the parameter
const rowsB = species.map(({ name, count }) => `${name}: ${count}`);

console.log(rowsB); // ['Gannet: 42', 'Eider: 18', 'Turnstone: 5']
```

Read that parameter list out loud: "for each item, I need its `name` and its `count`". The signature now documents exactly what the function touches. On a five-field object where you use two fields, that is a real gain in clarity, and it is why you will see this style in essentially every modern codebase.

The same trick works everywhere a callback appears:

```js
// filter: only the busy sightings
const busy = species.filter(({ count }) => count >= 10);
console.log(busy.length); // 2

// find: the first species that had any birds ringed
const firstRinged = species.find(({ ringed }) => ringed > 0);
console.log(firstRinged.name); // 'Eider'

// find returns undefined when nothing matches - always guard
const missing = species.find(({ name }) => name === 'Puffin');
console.log(missing?.name ?? 'not seen'); // 'not seen'
```

Note the guard on the last one. `find` returning `undefined` is the most common source of "Cannot read properties of undefined" in beginner code.

<details>
<summary><strong>TypeScript version: Job 3</strong></summary>

```ts
const species: SpeciesRecord[] = outings[0].species;

// No annotation on the callback parameter. TypeScript already knows the element type,
// so it knows name is a string and count is a number.
const rows: string[] = species.map(({ name, count }) => `${name}: ${count}`);

const busy: SpeciesRecord[] = species.filter(({ count }) => count >= 10);

// find is typed SpeciesRecord | undefined, so the guard is compulsory
const firstRinged = species.find(({ ringed }) => ringed > 0);
console.log(firstRinged?.name ?? 'none ringed');

// A typo in the pattern is caught immediately:
// const rows = species.map(({ nmae }) => nmae);
// Error: Property 'nmae' does not exist on type 'SpeciesRecord'.
```

**How this differs from JS**

- **Callback parameters are inferred, so leave them bare.** `species.map(({ name, count }: SpeciesRecord) => ...)` is legal but redundant noise. Annotate the array, not the callback.
- **Destructuring becomes spell-checked.** In JavaScript, `({ nmae })` quietly gives you `undefined` and you see `undefined: 42` in the output. In TypeScript it is a compile error with the correct name suggested.
- **`find` forces the guard.** Its return type is `SpeciesRecord | undefined`, so reading `.name` off it without `?.` or a check will not compile. This is the single biggest day-to-day benefit of TypeScript for beginners.
- One thing TypeScript does **not** do for free: `filter` does not narrow types. `list.filter((x) => x !== undefined)` still gives you `(T | undefined)[]`. Fixing that needs a type predicate (`(x): x is T => ...`), which belongs to the type-guards lesson.

</details>

---

## Job 4: the totals

### `flatMap`: one level of nesting, removed

Each outing has an array of species. We want a single flat list of every sighting across every outing.

```js
// map gives you an array of arrays, which is rarely what you want
const nested = outings.map(({ species }) => species);
console.log(nested.length); // 3 (three arrays)

// flatMap does map and then flattens one level
const allSightings = outings.flatMap(({ species }) => species);
console.log(allSightings.length); // 8 (eight species records)
```

`flatMap` is exactly `map` followed by `.flat()`, in one pass. Reach for it whenever a callback returns an array and you want the contents, not the container.

### `reduce`: build anything from a list

`reduce` is the general-purpose tool. It carries an accumulator from one item to the next, and you decide what that accumulator is. A number, a string, an array, an object, anything.

```js
const totalBirds = allSightings.reduce((total, { count }) => total + count, 0);
console.log(totalBirds); // 176
```

Three parts, and it is worth naming them:

- `(total, { count }) => total + count` is the reducer. Whatever it returns becomes `total` on the next round.
- `0` is the starting value. **Always pass one.** Without it, `reduce` uses the first element as the seed, which breaks on empty arrays and on arrays of objects.
- The result is a single value, so `reduce` ends a chain. Nothing can follow it.

The accumulator does not have to be a number. Here it is an object, which is how you group data:

```js
// Total birds per county
const byCounty = outings.reduce((totals, { site: { county }, species }) => {
  const outingTotal = species.reduce((sum, { count }) => sum + count, 0);
  totals[county] = (totals[county] ?? 0) + outingTotal;
  return totals;
}, {});

console.log(byCounty); // { Rogaland: 107, Agder: 69 }
```

Notice the destructuring in the reducer parameter, pulling `county` out of a nested object and `species` from the top level in the same breath. That is Job 1 and Job 3 combined.

The single most common `reduce` bug: forgetting `return totals`. The accumulator becomes `undefined` on round two and everything collapses.

### Chaining: a pipeline that reads like a sentence

Because `map` and `filter` return arrays, you can keep going:

```js
const heavyRingingSites = outings
  .filter(({ species }) => species.some(({ ringed }) => ringed > 0))
  .map(({ site: { name } }) => name);

console.log(heavyRingingSites); // ['Utsira', 'Lista', 'Jaerstranda']
```

Read top to bottom: keep the outings where at least one species was ringed, then take the site name of each. One step per line, one job per step.

### `some` and `every`: questions, not lists

```js
const anyGannets = allSightings.some(({ name }) => name === 'Gannet');
const allCounted = allSightings.every(({ count }) => count > 0);
console.log(anyGannets, allCounted); // true true
```

`some` stops at the first `true`. `every` stops at the first `false`. Both return a boolean, which makes them ideal inside an `if`. And `includes` is the simpler cousin for plain values: `['Utsira', 'Lista'].includes('Lista')` is `true`.

<details>
<summary><strong>TypeScript version: Job 4</strong></summary>

```ts
// flatMap infers SpeciesRecord[] from the callback's return type
const allSightings: SpeciesRecord[] = outings.flatMap(({ species }) => species);

// The seed 0 tells TypeScript the accumulator is a number. Nothing else needed.
const totalBirds: number = allSightings.reduce((total, { count }) => total + count, 0);

// Grouping needs a hand, because an empty object seed infers as {}
type CountyTotals = Record<string, number>;

const byCounty = outings.reduce<CountyTotals>((totals, { site: { county }, species }) => {
  const outingTotal = species.reduce((sum, { count }) => sum + count, 0);
  totals[county] = (totals[county] ?? 0) + outingTotal;
  return totals;
}, {});

console.log(byCounty); // { Rogaland: 107, Agder: 69 }

const heavyRingingSites: string[] = outings
  .filter(({ species }) => species.some(({ ringed }) => ringed > 0))
  .map(({ site: { name } }) => name);

const anyGannets: boolean = allSightings.some(({ name }) => name === 'Gannet');
```

**How this differs from JS**

- **`reduce` with an object accumulator is the one place you must help TypeScript.** Given the seed `{}`, it infers the literal type `{}`, which has no properties, so `totals[county] = ...` is an error. Two fixes, both fine: the type argument `reduce<CountyTotals>(...)`, or annotating the seed as `{} as CountyTotals`. The type argument is clearer.
- `Record<string, number>` means "an object with string keys and number values". If the keys were a known union such as `type County = 'Rogaland' | 'Agder'`, prefer `Partial<Record<County, number>>`, because the object starts empty and is filled in gradually.
- **`reduce` for numbers needs nothing extra.** The seed `0` is enough for the accumulator to be inferred as `number`, and forgetting `return` inside the reducer becomes a compile error rather than a mystery `undefined`.
- `flatMap` infers its result from what the callback returns, so `SpeciesRecord[][]` collapses to `SpeciesRecord[]` in the type as well as at runtime.

</details>

---

## Job 5: correcting a record without wrecking the original

The last piece. Someone miscounted the Eiders at Utsira: it was 19, not 18.

**The tempting way is the wrong way:**

```js
outings[0].species[1].count = 19; // the original data is now gone
```

Once you overwrite it, there is no undo, no comparison against the old value, and any other part of the app holding a reference to that object silently changes too. Copy instead of edit.

**Spread (`...`) on the right-hand side is the copy tool.** Rest gathers; spread scatters. Same three dots, opposite direction, and which one you have depends entirely on which side of the `=` it sits.

```js
const original = outings[0].species[1];
const corrected = { ...original, count: 19 };

console.log(original.count);  // 18, untouched
console.log(corrected.count); // 19
```

The order matters: spread first, then your overrides. Later keys win.

For arrays, `with()` does the same job in one call:

```js
const speciesList = outings[0].species;
const updatedList = speciesList.with(1, corrected);

console.log(speciesList[1].count);  // 18
console.log(updatedList[1].count);  // 19
```

### One warning about `sort`

`sort` and `splice` change the array in place and return, respectively, the same array and the removed items. This catches people out constantly:

```js
const counts = [18, 42, 5];
const sorted = counts.sort((a, b) => b - a);
console.log(sorted); // [42, 18, 5]
console.log(counts); // [42, 18, 5] - the ORIGINAL was reordered too
console.log(sorted === counts); // true - same array, not a copy
```

`sorted` and `counts` are the same array. Use `toSorted()` when you want a copy, or spread first:

```js
const safelySorted = [...counts].sort((a, b) => b - a);
```

Also remember that a bare `sort()` compares items **as strings**, so `[10, 9, 100].sort()` gives `[10, 100, 9]`. Always pass a comparator for numbers.

<details>
<summary><strong>TypeScript version: Job 5</strong></summary>

```ts
const original: SpeciesRecord = outings[0].species[1];

// Annotating the result turns a typo into a compile error
const corrected: SpeciesRecord = { ...original, count: 19 };
// const wrong: SpeciesRecord = { ...original, cont: 19 };
// Error: Object literal may only specify known properties. Did you mean 'count'?

const updatedList: SpeciesRecord[] = outings[0].species.with(1, corrected);

// readonly makes mutation itself a compile error
const counts: readonly number[] = [18, 42, 5];
// counts.sort((a, b) => b - a);
// Error: Property 'sort' does not exist on type 'readonly number[]'.

const safelySorted: number[] = [...counts].sort((a, b) => b - a);
const alsoFine: number[] = counts.toSorted((a, b) => b - a);
```

**How this differs from JS**

- **Annotate the result of a spread.** Without `: SpeciesRecord`, the object `{ ...original, cont: 19 }` is perfectly valid TypeScript - it just infers a new shape with an extra `cont` property. The annotation is what turns the typo into an error, and it costs you nine characters.
- **`readonly` is how you enforce immutability at compile time.** A `readonly T[]` simply has no `sort`, `splice`, `push` or `reverse` on it. Marking your source data `readonly` means the compiler, not your discipline, stops you mutating it. `readonly` also works on interface properties: `readonly species: readonly SpeciesRecord[]`.
- `with()` and `toSorted()` are ES2023. If your editor underlines them, add `ES2023` to the `lib` array in `tsconfig.json`. The Vite vanilla-ts scaffold ships with `ES2022`, so this is a real thing you will hit.
- `structuredClone` needs `DOM` in `lib`, which the Vite scaffold does include.

</details>

---

## Summary card

```js
// PATTERNS (left of the =)
const { a, b: renamed, c = 'default' } = obj;   // objects match by name
const [first, , third, ...rest] = arr;          // arrays match by position
const { x, ...others } = obj;                   // rest gathers leftovers
function f({ id, name } = {}) {}                // patterns work as parameters

// SPREAD (right of the =)
const copy = { ...obj, count: 19 };             // copy plus override
const merged = [...a, ...b];                    // join arrays

// CHOOSING AN ARRAY METHOD
map      -> same length, transformed
filter   -> fewer items, same shape
find     -> one item or undefined
some     -> boolean, stops at first true
every    -> boolean, stops at first false
flatMap  -> map, then flatten one level
reduce   -> anything you like, always seed it, always return the accumulator
sort     -> MUTATES; use toSorted() or [...arr].sort()
```

---

## Exercises

Work in a plain `.js` file. Solutions are not submitted or assessed - these exist so you get the syntax into your fingers.

### Exercise 1: unpack a station reading (destructuring only)

```js
const reading = {
  station_id: 'SN39040',
  location: {
    place: 'Kjevik',
    position: [58.2042, 8.0853],
  },
  values: {
    air_temp: 14.2,
    wind: { speed_ms: 3.5, direction: 'NW' },
  },
};
```

Write **one** `const` statement that produces exactly these five variables:

- `stationId` - from `station_id` (rename)
- `place` - from the nested `location` (nesting)
- `latitude` - the first element of `position` (array pattern inside an object pattern)
- `windSpeed` - from `values.wind.speed_ms` (deep nesting plus rename)
- `humidity` - does not exist in the data, so default it to `50`

Log all five. Then answer in a comment: after your statement runs, is there a variable called `location`? Why not?

<details>
<summary><strong>Solution: Exercise 1</strong></summary>

```js
// JavaScript
const {
  station_id: stationId,
  location: {
    place,
    position: [latitude],
  },
  values: {
    wind: { speed_ms: windSpeed },
  },
  humidity = 50,
} = reading;

console.log(stationId); // 'SN39040'
console.log(place);     // 'Kjevik'
console.log(latitude);  // 58.2042
console.log(windSpeed); // 3.5
console.log(humidity);  // 50

// No, there is no variable called `location`. The colon after `location` means
// "descend into this object", not "give me this object". Nesting drills past the
// container without ever binding it. Ask for both if you need both:
// const { location, location: { place } } = reading;
```

```ts
// TypeScript
interface StationReading {
  station_id: string;
  location: {
    place: string;
    position: [number, number];
  };
  values: {
    air_temp: number;
    wind: { speed_ms: number; direction: string };
  };
  humidity?: number;
}

const reading: StationReading = {
  station_id: 'SN39040',
  location: { place: 'Kjevik', position: [58.2042, 8.0853] },
  values: { air_temp: 14.2, wind: { speed_ms: 3.5, direction: 'NW' } },
};

// The destructuring statement itself is byte-for-byte the same as the JavaScript
const {
  station_id: stationId,
  location: {
    place,
    position: [latitude],
  },
  values: {
    wind: { speed_ms: windSpeed },
  },
  humidity = 50,
} = reading;
```

**Notes**

- `humidity` sits at the top level of the pattern because that is where the property would live if it existed. A default works for a property that is absent from a type only if the type declares it optional - hence `humidity?: number` in the interface. Without that, TypeScript rejects the whole pattern with "Property 'humidity' does not exist".
- After the default, `humidity` is `number`, not `number | undefined`.
- `position` is a tuple, so `latitude` is `number`. The unused longitude simply is not named.

</details>

### Exercise 2: the relay team (array destructuring)

```js
const legTimes = ['4x100m final', 12.4, 11.8, 12.1];
```

The first element is a label, not a time.

1. In one statement, create `eventName` (the label), `openingLeg` (12.4), and `remainingLegs` (an array of the other two).
2. Create `anchorLeg` using `at()` so it holds the final time, without counting positions by hand.
3. The timing sheet has legs two and three swapped. Given `let legTwo = 11.8;` and `let legThree = 12.1;`, swap them **without** a temporary variable. Hint: it is one line, involving an array on both sides. Careful with the semicolon on the line above.
4. A leg that did not run comes through as `undefined`. Given `const partial = [13.0, undefined];`, destructure it so the second variable falls back to `0`. Then try it with `null` in place of `undefined` and explain the difference in a comment.

<details>
<summary><strong>Solution: Exercise 2</strong></summary>

```js
// JavaScript
const legTimes = ['4x100m final', 12.4, 11.8, 12.1];

// 1. Label, first time, and the rest
const [eventName, openingLeg, ...remainingLegs] = legTimes;
console.log(eventName);      // '4x100m final'
console.log(openingLeg);     // 12.4
console.log(remainingLegs);  // [11.8, 12.1]

// 2. The final time, without counting
const anchorLeg = legTimes.at(-1);
console.log(anchorLeg); // 12.1

// 3. Swapping without a temporary variable
let legTwo = 11.8;
let legThree = 12.1;

[legTwo, legThree] = [legThree, legTwo];
console.log(legTwo, legThree); // 12.1 11.8

// The right-hand side builds a throwaway array, and the left-hand pattern
// immediately unpacks it. Both reads happen before either write.
// The semicolon warning: if the previous line does not end in one, JavaScript
// reads `something\n[legTwo, legThree]` as an index lookup on `something`.
// Either end every statement with a semicolon, or start this line with one:
// ;[legTwo, legThree] = [legThree, legTwo];

// 4. Defaults on a short or gappy array
const partial = [13.0, undefined];
const [first, second = 0] = partial;
console.log(first, second); // 13 0

const withNull = [13.0, null];
const [firstAgain, secondAgain = 0] = withNull;
console.log(secondAgain); // null - the default did NOT fire

// A default only fires on `undefined`. `null` is a real value, deliberately set,
// so JavaScript hands it over untouched. Use `?? 0` if you need to catch both.
```

```ts
// TypeScript
// Typed as a tuple, each slot keeps its own type
const legTimes: [string, number, number, number] = ['4x100m final', 12.4, 11.8, 12.1];

const [eventName, openingLeg, ...remainingLegs] = legTimes;
// eventName: string, openingLeg: number, remainingLegs: [number, number]

// at() does not track tuple positions: the type is the union of EVERY slot
const anchorLeg: string | number | undefined = legTimes.at(-1);

// If you need it typed as a number, index the tuple directly
const anchorLegAsNumber: number = legTimes[3];

let legTwo = 11.8;
let legThree = 12.1;
[legTwo, legThree] = [legThree, legTwo];

// A leg that did not run
const partial: [number, number | undefined] = [13.0, undefined];
const [first, second = 0] = partial;
// second: number - the default removed undefined from the type

const withNull: [number, number | null] = [13.0, null];
const [firstAgain, secondAgain = 0] = withNull;
// secondAgain: number | null - the default did NOT remove null,
// so TypeScript keeps reminding you that null is still possible
```

**Notes**

- Without the tuple annotation, TypeScript infers `legTimes` as `(string | number)[]`, and every destructured variable becomes `string | number` - useless for arithmetic. Positional data deserves a tuple.
- The rest element of a tuple is itself a tuple: `remainingLegs` is `[number, number]`, so `remainingLegs.length` is known to be 2.
- **`at()` throws away tuple positions.** On `[string, number, number, number]`, `at(-1)` is typed `string | number | undefined`, because TypeScript does not evaluate `-1` into "the fourth slot". Annotating it `number | undefined` is a compile error. Use `legTimes[3]` when you need the precise type, and `at(-1)` when readability matters more.
- The `null` case is the clearest demonstration of what types buy you. In JavaScript the difference between `undefined` and `null` is invisible until something breaks at runtime; in TypeScript it is right there in the hover text.

</details>

### Exercise 3: the lending desk (array methods plus destructuring)

```js
const loans = [
  { id: 'L-01', borrower: { name: 'Aud' }, items: [{ title: 'Kon-Tiki', days: 14 }, { title: 'Sult', days: 7 }], returned: false },
  { id: 'L-02', borrower: { name: 'Kim' }, items: [{ title: 'Naiv. Super', days: 21 }], returned: true },
  { id: 'L-03', borrower: { name: 'Per' }, items: [{ title: 'Sult', days: 30 }, { title: 'Markens Grode', days: 10 }], returned: false },
];
```

Using destructuring in every callback parameter:

1. `borrowerNames` - an array of the three borrower names.
2. `overdue` - the loans not yet returned, mapped to strings like `'L-01 (Aud)'`.
3. `firstLongLoan` - the first loan containing an item borrowed for more than 20 days. Handle the case where none exists.
4. `allTitles` - a flat array of every title across every loan. (`flatMap`)
5. `totalDays` - the sum of `days` across every item everywhere. (`flatMap` then `reduce`) Expected: 82.
6. `titleCounts` - an object like `{ 'Sult': 2, ... }` counting how many times each title appears. (`reduce` with an object accumulator)
7. `everyLoanHasItems` - a boolean. (`every`)
8. Mark loan `L-01` as returned **without mutating** `loans`. Produce a new `updatedLoans` array, then log both to prove the original is unchanged. (`with` plus spread, or `map` plus spread)

---

<details>
<summary><strong>Solution: Exercise 3</strong></summary>

```js
// JavaScript

// 1. Every borrower's name
const borrowerNames = loans.map(({ borrower: { name } }) => name);
console.log(borrowerNames); // ['Aud', 'Kim', 'Per']

// 2. Outstanding loans as labels
const overdue = loans
  .filter(({ returned }) => !returned)
  .map(({ id, borrower: { name } }) => `${id} (${name})`);
console.log(overdue); // ['L-01 (Aud)', 'L-03 (Per)']

// 3. First loan with a long borrowing period
// `some` inside `find`: the outer question is "which loan", the inner is "any item?"
const firstLongLoan = loans.find(({ items }) => items.some(({ days }) => days > 20));
console.log(firstLongLoan?.id ?? 'none'); // 'L-02'

// 4. Every title, flattened
const allTitles = loans.flatMap(({ items }) => items.map(({ title }) => title));
console.log(allTitles);
// ['Kon-Tiki', 'Sult', 'Naiv. Super', 'Sult', 'Markens Grode']

// 5. Total borrowing days across everything
const totalDays = loans
  .flatMap(({ items }) => items)
  .reduce((sum, { days }) => sum + days, 0);
console.log(totalDays); // 82

// 6. How often each title appears
const titleCounts = allTitles.reduce((counts, title) => {
  counts[title] = (counts[title] ?? 0) + 1;
  return counts;
}, {});
console.log(titleCounts);
// { 'Kon-Tiki': 1, 'Sult': 2, 'Naiv. Super': 1, 'Markens Grode': 1 }

// 7. A yes or no question
const everyLoanHasItems = loans.every(({ items }) => items.length > 0);
console.log(everyLoanHasItems); // true

// 8. Marking L-01 returned without touching the original
const updatedLoans = loans.map((loan) =>
  loan.id === 'L-01' ? { ...loan, returned: true } : loan
);

console.log(loans[0].returned);        // false - original intact
console.log(updatedLoans[0].returned); // true

// `with` is the alternative when you already know the index:
// const index = loans.findIndex(({ id }) => id === 'L-01');
// const updatedLoans = loans.with(index, { ...loans[index], returned: true });
```

```ts
// TypeScript
interface LoanItem {
  title: string;
  days: number;
}

interface Loan {
  id: string;
  borrower: { name: string };
  items: LoanItem[];
  returned: boolean;
}

const loans: Loan[] = [
  // ...same data
];

const borrowerNames: string[] = loans.map(({ borrower: { name } }) => name);

const overdue: string[] = loans
  .filter(({ returned }) => !returned)
  .map(({ id, borrower: { name } }) => `${id} (${name})`);

// Loan | undefined, so the optional chain is compulsory
const firstLongLoan = loans.find(({ items }) => items.some(({ days }) => days > 20));
console.log(firstLongLoan?.id ?? 'none');

const allTitles: string[] = loans.flatMap(({ items }) => items.map(({ title }) => title));

const totalDays: number = loans
  .flatMap(({ items }) => items)
  .reduce((sum, { days }) => sum + days, 0);

// The one call that needs a type argument
const titleCounts = allTitles.reduce<Record<string, number>>((counts, title) => {
  counts[title] = (counts[title] ?? 0) + 1;
  return counts;
}, {});

const everyLoanHasItems: boolean = loans.every(({ items }) => items.length > 0);

const updatedLoans: Loan[] = loans.map((loan) =>
  loan.id === 'L-01' ? { ...loan, returned: true } : loan
);
```

**Notes**

- Task 4 needs `flatMap` **and** an inner `map`, because the callback must return an array of titles, not an array of item objects. `loans.flatMap(({ items }) => items)` would give you the item objects instead - which is exactly what task 5 wants, so compare the two.
- Task 6 is the only place TypeScript needs help. `reduce<Record<string, number>>` tells it the accumulator is a lookup object; without it the empty seed infers as `{}` and the assignment fails to compile.
- Task 8: `map` plus a conditional handles "update the one that matches" without knowing where it is. Everything not matching is returned by reference, unchanged, which is correct and cheap - you only copy what you change.
- Annotating `updatedLoans: Loan[]` is what makes a typo such as `retunred: true` an error. Spread without an annotation would happily invent a new shape.

</details>

## Self-study task: Festival line-up summariser

Build a single-page tool, in plain HTML, CSS and JavaScript. No framework, no build step, no fetching - the data is hard-coded in your file. The point is not the styling. The point is that **every piece of state you display is derived from one array, and that array is never mutated**.

### Starting data

```js
const stages = [
  {
    id: 'stage-main',
    name: 'Main Stage',
    capacity: 8000,
    acts: [
      { id: 'a1', name: 'Nordlys', genre: 'electronic', minutes: 75, headliner: true },
      { id: 'a2', name: 'Salt & Stein', genre: 'folk', minutes: 45, headliner: false },
      { id: 'a3', name: 'Kobber', genre: 'rock', minutes: 60, headliner: false },
    ],
  },
  {
    id: 'stage-tent',
    name: 'The Tent',
    capacity: 1200,
    acts: [
      { id: 'a4', name: 'Vals for Ingen', genre: 'folk', minutes: 40, headliner: false },
      { id: 'a5', name: 'Blaatimen', genre: 'electronic', minutes: 90, headliner: true },
    ],
  },
  {
    id: 'stage-club',
    name: 'Club Room',
    capacity: 300,
    acts: [
      { id: 'a6', name: 'Hos Ruth', genre: 'jazz', minutes: 55, headliner: false },
    ],
  },
];
```

### Level 1 process

1. **Render the stage cards.** For each stage, show its name, capacity, and how many acts it holds. Use destructuring in the `map` callback so the callback signature names exactly the fields it uses.
2. **Render the acts under each stage.** Each row shows the act name, genre, and duration. Headliners get a visible marker in the text, such as `[HEADLINER]`.
3. **Show a summary bar** above everything with three figures, each derived at render time rather than stored:
   - total number of acts across all stages (`flatMap`)
   - total stage time in hours and minutes (`flatMap` then `reduce`, expected total is 365 minutes)
   - a breakdown of acts per genre, for example `folk 2, electronic 2, rock 1, jazz 1` (`reduce` with an object accumulator)
4. **Add a genre filter.** A `<select>` listing every genre that actually appears in the data, built with `flatMap` and `Set`, plus an "all" option. Changing it re-renders and filters the act rows. The summary bar must reflect what is currently shown.
5. **Add a "lengthen by 5 minutes" button** on each act row. Clicking it must produce a **new** `stages` array with that one act's `minutes` increased by 5. Do not write `act.minutes += 5`. Use `map` and spread, or `with`, all the way down through stage and act. Then re-render from the new array.
6. **Prove immutability.** Keep a `const originalStages = structuredClone(stages);` at the top. Add a "check original" button that logs whether the original data still matches its starting values. If your update logic is correct, it always will.

### Level 2 process (optional)

7. Sort the act rows by duration, longest first, using `toSorted` so the source array is untouched. Add a toggle for ascending and descending.
8. Add a "clashes" warning: if any stage has more than 150 total minutes of programme, show a warning on that stage card. Use `some` on the derived totals.
9. Extract your render logic into small functions - `renderStage`, `renderAct`, `renderSummary` - each taking a destructured object parameter with a sensible `= {}` fallback, and each returning an HTML string rather than touching the DOM directly. Only one function at the end should write to the DOM.

### What to check before you call it done

- No callback parameter is a vague single letter where a destructured pattern would be clearer.
- No `reduce` without a starting value.
- No `sort` on an array you did not create yourself in that same expression.
- Every `find` result is guarded before you read a property off it.
- Clicking "lengthen" twenty times and then "check original" still reports the original as unchanged.

---

<details>
<summary><strong>Solution: self-study task (Level 1 reference implementation)</strong></summary>

This is one correct answer, not the only one. Compare structure rather than character-by-character.

**index.html**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Festival line-up summariser</title>
    <style>
      body { font-family: system-ui, sans-serif; margin: 2rem; max-width: 48rem; }
      .stage { border: 1px solid #ccc; padding: 1rem; margin-block: 1rem; }
      .act { display: flex; justify-content: space-between; gap: 1rem; padding: 0.25rem 0; }
      .warn { color: #a33; font-weight: bold; }
      ul { list-style: none; padding: 0; }
    </style>
  </head>
  <body>
    <h1>Festival line-up</h1>

    <label for="genre-filter">Genre</label>
    <select id="genre-filter"></select>

    <div id="summary"></div>
    <div id="stages"></div>

    <button id="check-original" type="button">Check original</button>
    <p id="check-result"></p>

    <script type="module" src="./main.js"></script>
  </body>
</html>
```

**main.js**

```js
// Data
const initialStages = [
  // ...the starting data from the brief
];

// A snapshot taken before anything can touch the data
const originalStages = structuredClone(initialStages);

// State - only these two lines are allowed to change
let stages = initialStages;
let activeGenre = 'all';

// Derived values - none of this is stored, all of it is recalculated on render

/**
 * Flattens every act across every stage into one list.
 * @param {Array} stageList - The stages to read.
 * @returns {Array} Every act, in stage order.
 */
function allActs(stageList) {
  return stageList.flatMap(({ acts }) => acts);
}

function totalMinutes(acts) {
  return acts.reduce((sum, { minutes }) => sum + minutes, 0);
}

function countByGenre(acts) {
  return acts.reduce((counts, { genre }) => {
    counts[genre] = (counts[genre] ?? 0) + 1;
    return counts;
  }, {});
}

function formatDuration(minutes) {
  return `${Math.floor(minutes / 60)}h ${minutes % 60}m`;
}

function visibleActs(acts) {
  return activeGenre === 'all' ? acts : acts.filter(({ genre }) => genre === activeGenre);
}

// Rendering - every function returns a string, none of them touch the DOM

function renderAct({ id, name, genre, minutes, headliner = false } = {}) {
  const marker = headliner ? ' [HEADLINER]' : '';
  return `
    <li class="act">
      <span>${name}${marker} - ${genre} - ${minutes} min</span>
      <button type="button" data-act-id="${id}">+5 min</button>
    </li>`;
}

function renderStage({ name, capacity, acts = [] } = {}) {
  const shown = visibleActs(acts);
  const warning =
    totalMinutes(acts) > 150 ? '<p class="warn">Long programme on this stage</p>' : '';

  return `
    <section class="stage">
      <h2>${name}</h2>
      <p>Capacity ${capacity} - ${acts.length} acts</p>
      ${warning}
      <ul>${shown.map((act) => renderAct(act)).join('')}</ul>
    </section>`;
}

function renderSummary(acts = []) {
  // Object.entries gives [key, value] pairs, so the map callback destructures an array
  const genreLine = Object.entries(countByGenre(acts))
    .map(([genre, count]) => `${genre} ${count}`)
    .join(', ');

  return `
    <p><strong>${acts.length}</strong> acts, ${formatDuration(totalMinutes(acts))} of music</p>
    <p>${genreLine}</p>`;
}

function renderGenreOptions(acts) {
  const genres = ['all', ...new Set(acts.map(({ genre }) => genre))];
  return genres
    .map((genre) => `<option value="${genre}"${genre === activeGenre ? ' selected' : ''}>${genre}</option>`)
    .join('');
}

// The only function that writes to the DOM
function render() {
  const acts = allActs(stages);
  const shown = visibleActs(acts);

  document.querySelector('#genre-filter').innerHTML = renderGenreOptions(acts);
  document.querySelector('#summary').innerHTML = renderSummary(shown);
  document.querySelector('#stages').innerHTML = stages.map((stage) => renderStage(stage)).join('');
}

// Updates - build a new array, never edit the old one

function lengthenAct(actId) {
  stages = stages.map((stage) => {
    const index = stage.acts.findIndex(({ id }) => id === actId);
    if (index === -1) {
      return stage; // untouched stages are reused as-is, no copy needed
    }

    const act = stage.acts[index];
    return {
      ...stage,
      acts: stage.acts.with(index, { ...act, minutes: act.minutes + 5 }),
    };
  });
}

// Events

document.querySelector('#stages').addEventListener('click', (event) => {
  const button = event.target.closest('button[data-act-id]');
  if (!button) {
    return;
  }
  lengthenAct(button.dataset.actId);
  render();
});

document.querySelector('#genre-filter').addEventListener('change', (event) => {
  activeGenre = event.target.value;
  render();
});

document.querySelector('#check-original').addEventListener('click', () => {
  const unchanged = JSON.stringify(initialStages) === JSON.stringify(originalStages);
  document.querySelector('#check-result').textContent = unchanged
    ? 'Original data is unchanged.'
    : 'Original data has been mutated.';
});

render();
```

**Why it is built this way**

- `stages` is reassigned, never edited. `lengthenAct` rebuilds the one stage and the one act that changed and reuses every other object by reference. That is the whole immutability pattern in eight lines.
- Nothing derived is stored. Totals, genre counts and the filtered list are recalculated inside `render`, so they can never disagree with the data.
- One DOM write, in `render`. Every other function is a pure string builder you could unit-test without a browser.
- Event delegation on `#stages` means the buttons keep working after `innerHTML` replaces them. Attaching listeners to each button inside `render` would break on the first re-render.

**TypeScript version**

The derive and render functions are unchanged apart from annotations. What genuinely differs is the type model and the DOM code.

```ts
type Genre = 'electronic' | 'folk' | 'rock' | 'jazz';

interface Act {
  id: string;
  name: string;
  genre: Genre;
  minutes: number;
  headliner: boolean;
}

interface Stage {
  id: string;
  name: string;
  capacity: number;
  acts: Act[];
}

const initialStages: Stage[] = [
  // ...same data
];

const originalStages: Stage[] = structuredClone(initialStages);

let stages: Stage[] = initialStages;
let activeGenre: Genre | 'all' = 'all';

// Partial<Record<...>> because the object starts empty and fills up gradually
type GenreCounts = Partial<Record<Genre, number>>;

function countByGenre(acts: Act[]): GenreCounts {
  return acts.reduce<GenreCounts>((counts, { genre }) => {
    counts[genre] = (counts[genre] ?? 0) + 1;
    return counts;
  }, {});
}

function renderAct({ id, name, genre, minutes, headliner = false }: Partial<Act> = {}): string {
  const marker = headliner ? ' [HEADLINER]' : '';
  return `
    <li class="act">
      <span>${name}${marker} - ${genre} - ${minutes} min</span>
      <button type="button" data-act-id="${id}">+5 min</button>
    </li>`;
}

// querySelector returns Element | null, so say what you expect or check for null
function render(): void {
  const summaryEl = document.querySelector<HTMLElement>('#summary');
  const stagesEl = document.querySelector<HTMLElement>('#stages');
  const filterEl = document.querySelector<HTMLSelectElement>('#genre-filter');

  if (!summaryEl || !stagesEl || !filterEl) {
    throw new Error('Required elements are missing from the page');
  }

  const acts = allActs(stages);
  filterEl.innerHTML = renderGenreOptions(acts);
  summaryEl.innerHTML = renderSummary(visibleActs(acts));
  stagesEl.innerHTML = stages.map((stage) => renderStage(stage)).join('');
}

// event.target is EventTarget | null, so narrow it before using DOM methods
document.querySelector('#stages')?.addEventListener('click', (event: Event) => {
  const { target } = event;
  if (!(target instanceof HTMLElement)) {
    return;
  }

  const button = target.closest<HTMLButtonElement>('button[data-act-id]');
  const actId = button?.dataset.actId;
  if (!actId) {
    return;
  }

  lengthenAct(actId);
  render();
});

// The change listener needs one more step than the JavaScript version.
// A <select> hands you a plain string, which is not a Genre.
const GENRES: Genre[] = ['electronic', 'folk', 'rock', 'jazz'];

function isGenreFilter(value: string): value is Genre | 'all' {
  return value === 'all' || GENRES.includes(value as Genre);
}

document.querySelector('#genre-filter')?.addEventListener('change', (event: Event) => {
  const { target } = event;
  if (!(target instanceof HTMLSelectElement) || !isGenreFilter(target.value)) {
    return;
  }

  activeGenre = target.value; // now narrowed to Genre | 'all'
  render();
});
```

**How the TypeScript version differs**

- **`querySelector` returns `Element | null`.** Either pass the element type as a generic (`querySelector<HTMLSelectElement>`) and check for `null`, or use `?.`. This is the single biggest change when moving DOM code to TypeScript, and it catches a real bug class: a typo in a selector.
- **`event.target` is `EventTarget | null`** and has no `closest`. Narrow it with `instanceof HTMLElement` first. That is a type guard, straight out of the earlier lesson.
- **`dataset.actId` is `string | undefined`**, always, because the attribute may be absent. Guard before use.
- **`genre: Genre`** as a union rather than `string` means a typo such as `'foolk'` in the data is caught immediately, and `Record<Genre, number>` knows all four keys. Because the counting object starts empty, wrap it in `Partial<...>`, otherwise TypeScript insists all four genres exist from the first line.
- **A `<select>` value is always `string`.** With `activeGenre` typed `Genre | 'all'`, the line `activeGenre = target.value` does not compile. The honest fix is the `isGenreFilter` type predicate above, which is the same type-guard pattern from the earlier lesson. The lazy fix is `activeGenre = target.value as Genre`, which silences the compiler and tells it a lie. If this feels like too much ceremony, type `genre` as `string` throughout and the problem disappears along with the safety.
- **`Partial<Act>` on `renderAct`** is what makes the `= {}` fallback legal. Without `Partial`, an empty object does not satisfy `Act`.
- `stages.with(...)` again needs `ES2023` in the `lib` array of `tsconfig.json`.

</details>

## Further reading

**Destructuring, rest and spread**

- [Destructuring](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring) - MDN. The reference for everything in Jobs 1 and 2. Note the distinction it draws between binding patterns and assignment patterns, which is why the swap in Exercise 2 needs its semicolon.
- [Spread syntax (`...`)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) - MDN. Includes the point that all spread copies are shallow, which matters the moment your data nests.
- [Rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) - MDN. The `...` in a function signature, as opposed to in a pattern.

**Array methods**

- [`Array`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) - MDN. Worth opening for one thing in particular: the table pairing each mutating method with its copying alternative (`sort` with `toSorted`, `splice` with `toSpliced`, and so on). That table is Job 5 in a single screen.
- [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) - MDN.
- [`flatMap()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap) - MDN.
- [`at()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/at) - MDN.
- [`with()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/with) - MDN.
- [`toSorted()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/toSorted) - MDN.

**If you opened the TypeScript blocks**

- [Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) - TypeScript Handbook. Covers interfaces, optional properties and the object types used throughout this lesson.
- [TSConfig reference: `lib`](https://www.typescriptlang.org/tsconfig/lib.html) - the setting to change when `with()` and `toSorted()` are underlined in red.
