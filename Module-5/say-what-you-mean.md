# Say What You Mean

Specifications, prompts, and the difference between plausible and correct.

**Prerequisites:** none beyond the rest of the course. Access to any conversational LLM.  
**Time:** about 60 minutes, plus the self-study task

---

## How this lesson works

Module 5 teaches project requirements (5.4) and LLMs as a development tool (5.3) as two separate lessons. They are one skill.

A requirement is a description of what you want built, written for a person. A prompt is a description of what you want built, written for a machine. Acceptance criteria - the conditions that have to be true before you can call it finished - are how you check the result, and the list is identical either way.

What makes this worth an hour rather than a slogan is the failure mode. Give a vague requirement to a developer and you get **questions back**. Give a vague prompt to a model and you get **an answer** - fluent, runnable, and full of decisions that nobody made on purpose.

This lesson takes one under-specified request and runs it twice: once as asked, once after writing down what "done" means. Two sections then follow on from that - where these tools are worth using, and why the code they produce is harder to review than it looks. Both are part of the lesson, not extras.

A few things are folded away in collapsible blocks: exercise solutions, and two asides that go deeper into a detail than you strictly need. Everything else is meant to be read.

**Note on running the prompts.** You will not get the output shown here. That is not a caveat, it is the subject of one of the sections below, and the solutions are written as *what to look for* rather than *what you will get*.

---

## The request

Same noticeboard as the previous lesson, though you do not need to have read it. Residents of a housing block post small ads. Someone asks for this:

> Can we make it so people can find things on the board?

That is a real request. It is roughly the level of detail you will be given, and it is about as specific as most user stories in a backlog.

---

## Round 1: build exactly what was asked

Paste that sentence into any LLM, ask for vanilla JavaScript, and you will get something close to this. Not identical - close.

```js
function searchListings(listings, query) {
  return listings.filter((listing) =>
    listing.title.toLowerCase().includes(query.toLowerCase())
  );
}

searchInput.addEventListener("input", () => {
  render(searchListings(listings, searchInput.value));
});
```

It works. It is correct JavaScript. You could demo it.

Now count the decisions in those eight lines that nobody made:

1. **Which fields?** Title only. Not the body, not the seller's name, not the price.
2. **Case.** Ignored, so "LAMP" matches "lamp". Reasonable, but a decision.
3. **Matching.** Substring, so "amp" matches "Lamp". Not prefix, not whole word.
4. **Multiple words.** "desk lamp" is treated as one literal string, so "lamp desk" finds nothing.
5. **Norwegian characters.** Untouched. More on this shortly, because it is the interesting one.
6. **Empty query.** Returns everything. It could as easily have returned nothing.
7. **No matches.** The board goes blank with no explanation.
8. **Sold listings.** Included, because nothing said otherwise.
9. **Where it runs.** Client-side over the whole array. Fine for 400 listings, useless for 40,000.
10. **The URL.** Unchanged, so a search result cannot be shared or reached with the back button.
11. **Frequency.** Runs on every keystroke.
12. **Scale.** No opinion about how big the list gets.

Twelve decisions. Some are fine. Several are wrong for this particular noticeboard. All of them were made silently.

**The model did not guess wrong. It guessed average.** It produced something statistically typical of the search functions in its training data, which is the correct response to a question that did not distinguish this search from every other search ever written. Average is the right answer to a question you did not ask.

### The one that actually bites

Look at point 5. Ask a model to handle accented characters in search and you will very often get this, because it is the popular snippet.

**One word first.** In text processing, *folding* means collapsing variants of a character so they compare as equal - `A` and `a` fold together under case folding, and `é` and `e` fold together under accent folding. It is the ordinary word for this job and you will meet it in the wild. It has nothing at all to do with `fold` as a synonym for `reduce`, which some languages use and which you met in Module 1. Different word, same spelling. I have named the function below for what it does, to keep the two apart.

```js
// The internet's favourite normalisation. Try it before you trust it.
const normaliseForSearch = (text) =>
  text.normalize("NFD").replace(/\p{Diacritic}/gu, "").toLowerCase();
```

Run it on Norwegian:

```js
normaliseForSearch("Lampe");     // "lampe"
normaliseForSearch("Ångstrøm");  // "angstrom"?  No: "angstrøm"
normaliseForSearch("Kjøkken");   // "kjøkken"
normaliseForSearch("Særlig");    // "særlig"
```

`å` collapses to `a`. `ø` and `æ` do not.

<details>
<summary><strong>Why, in more detail than you strictly need</strong></summary>

A **code point** is the number Unicode assigns to a character. `a` is 97. There is a single code point for `å` (229) and also a way to build the same visible letter out of two: `a` followed by a *combining ring above*, which is a mark that renders on top of the character before it.

**`normalize("NFD")`** rewrites a string into the second form wherever it can - the D stands for decomposed. So `å` becomes two code points, `a` plus a ring.

**`\p{Diacritic}`** is a regular expression pattern that matches "any character Unicode has flagged as a diacritical mark". The `u` flag is what enables that syntax. So the `replace` deletes the ring and leaves the `a`.

That whole chain only works on characters that *can* be decomposed. `ø` and `æ` are not an `o` or an `a` with something added; they are single code points that Unicode does not decompose into anything, so NFD leaves them alone and there is no diacritic for the regex to remove.

`é`, `ö` and `å` decompose. `ø` and `æ` do not. That is the entire bug, and there is no way to guess it from reading the code.

</details>

The snippet is not broken. It is doing exactly what it says, and what it says does not match what a Norwegian user expects. `Ångstrøm` becomes `angstrøm` - collapsed on one letter and not the other - so searching "angstrom" finds nothing.

And now the harder question, which is the actual point: **what should it do?** In Norwegian, `æ`, `ø` and `å` are letters in their own right at the end of the alphabet, not decorated vowels. Collapsing `kjøkken` to `kjokken` is arguably wrong. But residents typing on a foreign keyboard, or in a hurry, will type "kjokken" and expect a result.

There is no default correct answer. There is only a decision, and somebody has to make it. Left unspecified, it gets made by a system optimising for what is common across all languages at once.

---

## Why you cannot reproduce any of this

Run the same prompt twice and you get two different answers. This is worth understanding properly, because almost every bad habit around these tools traces back to misunderstanding it.

**A language model does not look up an answer and hand it to you. It writes one piece at a time, and at every step it is choosing.**

The pieces are called **tokens** - roughly a word, a bit of a word, or a piece of punctuation. At each step the model works out, for every token it knows, how likely that token is to come next. That produces a long list of candidates with a score against each: maybe `filter` at 61%, `find` at 12%, `for` at 9%, and a very long tail below that. Then it picks one and moves on.

**It does not always pick the highest score.** How strongly the choice favours the top of the list is a setting, usually called **temperature**. Low temperature means it almost always takes the favourite; high temperature means it wanders further down the list. Some wandering is deliberate, because always taking the single most likely token produces flat, repetitive text.

That choice happens for every token, and the effect compounds. Pick `find` instead of `filter` in line 2 and every subsequent step is now predicting the continuation of a different program. A single different choice early can produce a structurally different function by line 20.

So: two runs, two different walks through the same list of possibilities. Nothing went wrong.

**You might think turning the temperature to zero fixes this.** It helps a great deal, and it still does not make the thing reproducible, for four reasons that have nothing to do with the sampling:

- **The arithmetic is not perfectly repeatable.** Explained in the aside below, if you want it. The short version: those percentages are computed to many decimal places, and the last few can shift depending on what else the server happened to be doing. When two candidates are nearly tied, that is enough to flip which one wins.
- **The model behind a name changes.** "GPT-5" or "Claude Sonnet" is a product name, not a fixed artefact. Providers update the model, its hidden instructions and its safety layers without changing anything you type.
- **Your context is never identical.** Chat history, an attached file, a different folder open in your editor, documentation the tool fetched for itself. All of it is input, and all of it moves those percentages.
- **Training data has a cutoff.** The model's picture of a library's API is a snapshot, confidently presented, and it may predate the version you have installed.

<details>
<summary><strong>Aside: why the arithmetic is not perfectly repeatable</strong></summary>

Skip this if you like; nothing later depends on it.

Computers store decimals with limited precision, so adding them up is not quite the pure arithmetic you learnt at school. `(a + b) + c` and `a + (b + c)` can give very slightly different answers, because each addition rounds. With ordinary numbers this never matters. When you are adding billions of them to produce a score of 0.6100000001 versus 0.6099999998, it can.

On top of that, the servers running these models handle many people's requests at the same time, grouped together for speed. Which requests you happen to be grouped with can change the order the additions are done in, which changes where the rounding falls.

None of this changes the answer in any meaningful sense. It only changes which of two nearly-tied candidates comes out on top - and because of the compounding described above, one flipped coin near the start is enough to give you a different program.

</details>

### What follows from that, practically

**"It worked when I tried it" is evidence about one output. It is not evidence about the tool.** This is the habit to break. A student tries a prompt, gets good code, and concludes the prompt is good. What they have is one sample from a distribution they have not looked at.

If you want to know whether a prompt is good, run it three times and read all three. Cheap, fast, and much more informative than reading one.

**Anything you cannot reproduce, you have to be able to check.** If the output varies, then the only stable thing in the process is your criteria for accepting it. Which is the argument for the rest of this lesson: when you cannot pin down what you will get, you had better pin down what you will accept.

This is also why every solution in this lesson is written as a checklist rather than an answer key. Nobody can tell you what your model will produce this week. Everybody can tell you what a good result would have to contain.

---

## Round 2: write down what "done" means

Back to the request. Before touching a prompt, write three things.

**Functional requirements - what it does.**

- Residents can filter the visible board by typing a term.
- Search covers the title and the body of a listing.
- Listings marked sold are excluded from results.

**Non-functional requirements - how well it does it.**

- Results appear within 300 ms of the resident stopping typing.
- Works on up to 2,000 listings without the page becoming unresponsive.
- The current search term survives a page reload and a back-button press.

**Acceptance criteria - how you know.**

*Acceptance criteria* are the conditions that have to be true before you are allowed to call the feature finished. They are the part user stories usually leave out, and they are the part that does all the work. "As a resident I want to search listings so that I can find things" tells you what somebody wants. It does not tell you how anybody would know whether you had done it.

The format below is **Given / When / Then**, which comes from Behaviour-Driven Development and was developed by Daniel Terhorst-North and Chris Matts. Three parts:

- **Given** - the situation before anything happens. The setup.
- **When** - the thing the user does.
- **Then** - what must be true afterwards.

That is the whole format. It is worth using rather than free prose because those three parts are exactly what you need in order to check something, and writing them forces you to notice when you cannot fill one in. It is also close to how you would structure a unit test, which is not a coincidence: the same shape turns up as `describe` / `it` / `expect` in the Jest tests you wrote in Module 2, and there is a testing tool called Cucumber that runs Given/When/Then text as actual tests.

You do not need any of that tooling here. You are writing them in a document, in English, so that another person could read one and hand back a yes or a no:

```
Given the board has a listing titled "Kjøkkenbord"
When I search for "kjokkenbord"
Then that listing appears

Given the board has listings titled "Desk lamp" and "Lamp shade"
When I search for "lamp desk"
Then "Desk lamp" appears and "Lamp shade" does not

Given a search that matches nothing
When the results render
Then the board shows "No listings match that search"
And the board is not blank

Given an empty search box
When the results render
Then every unsold listing appears

Given a listing marked sold
When I search for a word in its title
Then it does not appear

Given I search for "lamp" and reload the page
When the page loads
Then the search box contains "lamp" and the results are filtered
```

Six criteria. Notice that the first one is a decision that has now been made on purpose: `ø` is collapsed, and a resident typing on any keyboard finds the kitchen table.

Notice also what happened to points 9 and 12 from the earlier list. "Up to 2,000 listings" is what decides whether you filter in the browser or ask the server, and it is a requirement, not a technical preference. The architecture fell out of the spec rather than being chosen first.

---

## The spec is the prompt

Now the prompt writes itself, because you have already done the hard part:

> Write a vanilla JavaScript function `searchListings(listings, query)` for a browser, no frameworks and no build step.
>
> Each listing is `{ id, title, body, sold }`. Return the listings that match.
>
> Requirements:
> - Match against title and body.
> - Exclude listings where `sold` is true.
> - Case-insensitive.
> - The query may contain several words. All of them must match, in any order.
> - Collapse Norwegian characters so that "kjokkenbord" matches "Kjøkkenbord". Note that `ø` and `æ` are single code points and are not handled by NFD normalisation, so map them explicitly.
> - An empty query returns all unsold listings.
>
> Do not write the DOM code. Just the function.

That is not a longer prompt because longer prompts are better. It is longer because it contains six decisions that were previously being made for you.

### The same list is the review checklist

Here is the join between the two lessons. Once you have acceptance criteria, you check the result against them - and **you check the same list whether a person or a model wrote the code.** Reviewing generated code is not a new skill that arrived with these tools. It is code review, applied to an author who cannot be asked what they were thinking.

Go through the criteria one at a time against whatever came back:

| Criterion | Check |
|---|---|
| Title and body | Does the filter touch both? |
| Excludes sold | Is there a `sold` check, and does it run before or after matching? |
| Multi-word, any order | Is the query split, or is it still one substring? |
| Norwegian characters | Try `kjokkenbord` against `Kjøkkenbord`. Do not read the code, run it |
| Empty query | Does it return everything or nothing? |
| No matches | Does the function's contract even cover this, or is it the caller's job? |

The last row is the one worth dwelling on. Two of the six criteria are about rendering, not filtering, so a correct `searchListings` cannot satisfy them alone. That is a gap in how the work was divided, and the checklist surfaced it. A checklist you write before you look at the answer finds things that reading the answer will not.

---

## Where to use one, and where not to

Most advice here is either "AI will replace developers" or "AI writes garbage". Both are useless. The useful version is narrower, and it is not really about the model at all.

### The heuristic: how fast can you check it?

Reach for a model where **verification is cheap**. Do it yourself where verification is expensive.

That single question predicts almost everything. It is not about how hard the task is, or how clever the model is. It is about how long it takes you to find out whether the answer is right.

**Cheap to verify, so a good fit:**

- **Boilerplate and format conversion.** A `fetch` wrapper, JSON to an interface, CSV to objects. Wrong answers fail immediately and loudly.
- **Test cases for a function you already understand.** You can read the assertions and know. It is also good at edge cases you did not think of, which is the actual value - not the typing.
- **Explaining unfamiliar code.** You can check the explanation against the code in front of you.
- **Regular expressions.** Notoriously hard to write, trivial to test. Almost the ideal case.
- **"What am I not thinking of?"** Asking for the edge cases in a spec you wrote is one of the highest-value uses there is, and it is the one people skip. You are not asking it to decide; you are asking it to enumerate.
- **Turning a vague requirement into testable criteria.** Give it "search should be fast" and ask for measurable versions. You pick which one is right. That is the division of labour this whole lesson is about.

**Expensive to verify, so do it yourself:**

- **Architectural and data-model decisions.** A plausible schema and a good schema look identical on Friday. The difference shows up in month three, when changing it is expensive. Model output is *most* fluent exactly here, which is the trap.
- **Anything depending on your project's private conventions.** It has never seen your codebase's rules. It will produce something idiomatic for the internet, not for you.
- **Recent library APIs.** Training cutoffs mean confident, out-of-date answers. This does not look like an error. It looks like working code that does not work.
- **Security judgement.** It will happily write `innerHTML = userInput` unless you ask it not to, because that is what most of the internet does. If you have read the previous lesson, you already know why "most of the internet" is not the standard you want.
- **"Is this a good idea?"** See below.

### It will agree with you

Ask "is this a good approach?" and you will usually be told yes. Ask "why is this a bad approach?" about the identical code and you will get a list of problems. Both answers are generated the same way: the model continues the conversation in the direction the conversation is pointing.

This matters most for the thing people reach for it hardest - "act as a senior developer and review my code". What that produces is a critique-shaped object. Sometimes it contains a real critique. You cannot tell which from reading it, because both look the same.

Two things that genuinely help:

- **Take your preference out of the prompt.** Not "I think we should filter client-side, is that right?" but "here is the requirement and here are two approaches; argue for each, then say which fails first and why."
- **Ask for the failure, not the verdict.** "Under what conditions does this break?" is answerable. "Is this good?" is not.

### Where it is stronger than people expect

Two things worth saying plainly, because the standard warnings undersell them.

**Enumeration under a constraint you supply.** "Here are my six acceptance criteria for a search feature. What have I not covered?" Genuinely useful, because breadth is exactly what a system trained on everything has, and because you are the one judging the list.

**Reading, not writing.** Pointing it at code you did not write and asking what it does is often the fastest way into an unfamiliar codebase. The check is immediate: does the explanation match what you see?

The pattern in both: you are using it where being *broadly typical* is the useful property, and you keep the judgement.

### One technique worth stealing: make it interview you

Everything above has you writing the spec and the model responding. There is an inversion that works better than it sounds: instead of writing the spec, ask the model to interview you until it has enough to write one.

> I want a search feature for a noticeboard of small ads. Do not write anything yet. Ask me one question at a time until you have enough to write a specification with acceptance criteria, then write it. Ask about the things I have not thought of.

The "one question at a time" instruction matters, and you will have to repeat it, because the default is to fire twenty questions at once and you will answer all twenty shallowly.

Why this works is the same reason the rest of the lesson works. Breadth is the thing the model has and you do not; judgement is the thing you have and it does not. Asking questions uses the first without needing the second. Every answer is still yours.

There is a second use of the same idea. Hand it a spec you have already written and ask it to interview *you* about whether the document is right. People are famously bad at reviewing a document by reading it - the eye slides over the gaps - and much better at answering questions about it.

---

## Why generated code is harder to review than it looks

Reading code is harder than writing it. Everyone knows this and nobody plans for it. Generated code adds four difficulties on top.

**1. There is nobody to ask.** Human code has an author with reasons. You can ask why the check is there, or read the commit message, or find the bug it was written for. Generated code has no reasons behind it, only patterns. "Why is this here?" has no answer, and the model's after-the-fact explanation of its own output is another generation, not a recollection.

**2. It is uniformly confident.** Human code signals its own weak points. A messy function, a `// not sure about this` comment, a variable named `tmp2`, an inconsistent bit where somebody was tired - all of it tells you where to look hardest. Generated code is smooth everywhere. The line that is subtly wrong looks exactly like the line that is right. You lose a signal you did not know you were using.

**3. Plausibility is the objective.** The model is not optimised to produce correct code. It is optimised to produce likely code. Most of the time those coincide, which is why it is useful. When they diverge, what you get is code shaped like working code, which is the hardest kind of wrong to spot.

**4. It invents things that do not exist.** A method with a sensible name that is not on that object. An option that was never in that library. Most of these fail loudly, which is fine. One category does not: **invented package names.** A model suggests `npm install some-plausible-helper`, and attackers have noticed that models hallucinate the same plausible names repeatedly, so they register them. Check that a package exists and is what it claims before installing it, particularly if the only place you have seen the name is a chat window.

### A word you will hear, and the distinction it hides

**Vibe coding** was coined by Andrej Karpathy in early 2025. In its original sense it means prompting a model for an application and *never looking at the code at all* - accepting every suggestion, pasting errors back in without reading them, and letting the program grow past the point where you could explain it. The defining feature is forgetting that the code exists.

The term escaped and now gets used for any coding with a model, including the careful kind where you read every diff. Those are two very different activities and it is worth keeping them apart in your own head, whatever other people call them.

Vibe coding in the strict sense is genuinely useful for something disposable that only you will run - a one-off script, a throwaway prototype, a thing you will delete on Friday. It is a bad fit for anything that will be maintained, used by other people, or touch information that matters. That covers everything you will submit on this course and most things anyone will pay you for.

This lesson is entirely about the other kind: the model writes it, you review it, you own it.

### What to do about it

**The ownership rule.** If you cannot explain a line, you cannot review it. If you cannot review it, do not commit it. Once it is in your repository it is yours, and "the AI wrote it" is not a thing you can say in a code review, a job interview, or a reflection document.

**Ask for the assumptions separately.** After you have the code, a second message - "what did you assume that I did not specify?" - is a different question and often a much more useful answer than the code was. It will not be complete. It is still faster than deriving the whole list yourself, and you can check each item.

**Run it before you read it.** Reading generated code carefully is expensive and the smoothness works against you. Running it against your acceptance criteria is cheap and does not care how confident it looks.

**One honest note for this course.** You will not be marked down for using these tools. You will fall over if you cannot explain what you submitted, and the assignment reflection asks you to be specific and honest about your process. "I generated this and checked it against my criteria, and here is what I had to fix" is a good answer. It is also, conveniently, true if you follow this lesson.

---

## Exercises

Solutions are folded under each. Write your answer first - and for the ones involving a model, run it before you open the solution, because the solution is a checklist and it works better if you have something to check.

### Exercise 1: the silent decisions

A resident asks for this:

> It would be good if you could see which things have already gone.

Do not write any code. List every decision an implementer would have to make in order to build it. Aim for eight or more.

<details>
<summary><strong>Solution</strong></summary>

There is no fixed correct list, but a good one covers these areas. If you found six or more, you are doing the thing this lesson is about.

**What "gone" means**
1. Sold, reserved, withdrawn by the seller, or expired? These are four different states and the request names none of them.
2. Who decides, and when? Does the seller mark it, or does it expire after a set time?
3. Can it be undone if a buyer pulls out?

**What the resident sees**
4. Are gone listings hidden, greyed out, moved to the bottom, or given a badge?
5. Are they included in search results?
6. Can you still open one and see the details, or read its contact link?

**Data and history**
7. Is the listing deleted or flagged? Deleting loses the record; flagging keeps it forever.
8. Does the seller still see their own gone listings somewhere?

**The bits nobody mentions until later**
9. What happens to a listing whose seller has left the building?
10. Does it show *when* it went? "Sold" and "sold in March" are different features.
11. Does anything need to change for people who had already loaded the page?

**The point.** The request was fourteen words. Ten decisions is not unusual, it is normal. This is what a model is doing on your behalf every time you hand it a sentence like this: making all ten, instantly, invisibly, in whichever direction is most common on the internet.

</details>

### Exercise 2: make it checkable

Rewrite each of these as acceptance criteria in Given / When / Then form. One statement may need more than one criterion.

1. "The board should load quickly."
2. "Users should be able to contact the seller."
3. "Don't show people rubbish listings."
4. "As a resident, I want to see new listings first, so I don't miss anything."

<details>
<summary><strong>Solution</strong></summary>

Yours will differ. What matters is that each one could be handed to another person who would return a yes or a no without asking you anything.

**1. "The board should load quickly."** Not checkable: no number, no condition, no definition of loaded.

```
Given a board of 400 listings on a 4G connection
When the page is loaded
Then the first listings are visible within 2 seconds
And the page does not shift layout after the images arrive
```

Two criteria, because "quickly" was hiding two different complaints. The second one is the thing that actually annoys people.

**2. "Users should be able to contact the seller."** Not checkable: contact by what means, and what happens when it is not possible?

```
Given a listing with a valid contact link
When I open the listing
Then a "Contact <seller>" link is shown and opens the seller's page

Given a listing whose contact link is missing or not an https URL
When I open the listing
Then the seller's name is shown as plain text with no link
```

The second criterion is the interesting one, and it is the one nobody writes. It is also, from the previous lesson, a security requirement wearing ordinary clothes.

**3. "Don't show people rubbish listings."** Barely a requirement. "Rubbish" needs a definition before anything can be built, and the honest first move is to go back and ask. If pressed:

```
Given a listing with no title or an empty body
When the board renders
Then that listing is not shown

Given a listing older than 90 days that has not been updated
When the board renders
Then that listing is not shown by default
And it is shown when "include old listings" is ticked
```

Both of those are guesses. Written down, they are guesses somebody can correct in ten seconds - which is exactly what a spec is for. Left in your head, they become behaviour that someone discovers in production.

**4. The user story.** It has a who, a what and a why, and it is still not checkable: "new" and "first" are both undefined.

```
Given listings posted on 1 March, 5 March and 5 March
When the board renders with no search term
Then they appear newest first
And the two listings from 5 March are ordered by time of posting

Given a listing that was edited after posting
When the board renders
Then it is ordered by its original posting date, not its edit date
```

That last criterion is a decision, not a detail. Ordering by edit date lets sellers bump their own listings to the top by making a trivial change, which is a feature you did not intend to build.

**What to notice across all four.** In each case, writing the criterion forced a decision that the original sentence had hidden. That is not extra work created by the format. The decision existed either way. The format is just what stops it being made by accident, by whoever writes the code - human or otherwise.

</details>

### Exercise 3: run the same prompt three times

This one needs a model. Any of them.

1. Take the vague request from Round 1: *"Can we make it so people can find things on the board?"* Add only "in vanilla JavaScript". Run it **three separate times, in three fresh conversations.**
2. For each of the three, work through the twelve silent decisions listed in Round 1 and record which way it went.
3. Now run the specified prompt from "The spec is the prompt" **twice**, and score both against the six acceptance criteria.
4. Answer: which decisions varied between the three vague runs, and which came out the same every time? What does that tell you?

<details>
<summary><strong>Solution: what to look for</strong></summary>

Nobody can tell you what you got. Here is what the results will show, and what each thing means.

**The vague runs will agree on some things and not others.**

Expect near-total agreement on: lower-casing the query, using `filter` with `includes`, and matching the title. These are so overwhelmingly common in the training data that sampling barely moves them.

Expect variation on: whether the body is searched, whether there is any debounce, whether an empty query is special-cased, whether it handles a no-results state, whether it is wrapped in a class or a plain function, and how much unrequested extra it invents.

**The reading.** Where the runs agree, you are seeing the strong consensus of published code - which is a reasonable default and is not the same thing as right for you. Where they disagree, you are watching a decision get made by a coin toss. Neither category was specified by you. The agreement is arguably the more dangerous of the two, because it is consistent enough to look deliberate.

**The specified runs should agree on nearly everything that you specified**, and may still vary in structure, naming, and whatever you left open. If a criterion you *did* specify came out differently across the two runs, that criterion is ambiguous - go and reword it. This is a genuinely good use of the non-determinism: **variation between runs is a map of what your spec failed to pin down.**

**Two things to check specifically:**

- **The Norwegian characters.** Do not read the code to decide whether it works. Run `searchListings` with "kjokkenbord" against a listing titled "Kjøkkenbord". A model that used the NFD snippet will produce code that looks completely correct and fails this. This is the whole lesson in one test case: plausible and correct came apart, and only running it told you.
- **The two rendering criteria.** A `searchListings` function cannot satisfy "the board shows a message" or "the search box still contains the term". If your scoring gave a mark for those, you scored something you did not ask for.

**If all three vague runs looked fine to you**, go back and check them against the six acceptance criteria rather than against your impression. "Looked fine" is the failure this lesson is about.

</details>

---

## Self-study task: specify, generate, grade

Not a build task. Roughly an hour.

### The feature

Residents want to be notified when something they are watching changes. That is all the detail you get, and it is deliberately about as vague as the request in Round 1.

### The task

1. **Write the spec.** Functional requirements, non-functional requirements, and at least six acceptance criteria in Given / When / Then form. Make every hidden decision explicitly, including the ones you are unsure about - a written guess can be corrected, an unwritten one cannot.
2. **Write two prompts.** One is the original vague sentence. The other is built from your spec.
3. **Run both.** Score each result against your own acceptance criteria. A criterion is met or not met; there is no partial credit, because "sort of works" is how features escape into production.
4. **Write the gap analysis.** One short section: which decisions did the vague run make for you, and which way did it go? Were any of them ones you would have chosen? Were any of them ones you had not realised were decisions until you saw them made?
5. **Run the specified prompt a second time** and note anything that differed. Each difference is a place your spec was ambiguous. Fix those criteria.

### What good looks like

- Your acceptance criteria could be handed to a stranger who could return a yes or a no on each.
- At least one criterion covers what happens when something is missing, empty, or fails.
- Your gap analysis names at least one decision you did not know you were making.
- You can explain every line you would keep.

<details>
<summary><strong>Solution: a worked spec and what the grading should surface</strong></summary>

One version of many. Yours should be a spec for your own reading of the feature, not a copy of this.

**The decisions hiding in "notified when something they are watching changes"**

Watching what - a single listing, a seller, or a search term? Changes how - price, availability, description, any edit? Notified how - a badge on the board, an email, a browser notification? When - immediately, or next time they open the page? What happens to a watch when the listing is deleted? Can you watch something that is already sold? How many things can you watch? Where is the watch list stored, and does it survive a different device?

Nine decisions, none of them in the sentence. If your spec answers fewer than five, the vague run will be making the rest for you.

**A narrowed spec**

*Functional*
- A resident can watch or unwatch any unsold listing.
- The board shows a count of watched listings that have changed since the resident last viewed them.
- A change means the price changed, or the listing became sold.

*Non-functional*
- The watch list survives a page reload.
- Checking for changes adds no more than 200 ms to page load for up to 50 watched listings.

*Acceptance criteria*

```
Given I am watching a listing priced at 150 kr
When the price changes to 100 kr and I load the board
Then the changed-listings count is 1

Given I am watching a listing and I have already viewed the change
When I load the board again
Then that listing is not counted again

Given I am watching a listing that becomes sold
When I load the board
Then it is counted as changed and marked sold

Given I am watching a listing that is deleted
When I load the board
Then no error is shown and the watch is removed

Given I have never watched anything
When I load the board
Then no count is shown at all, rather than a count of zero

Given I watch a listing and reload the page
When the board renders
Then the listing is still watched
```

**What the grading step should surface**

Three things reliably come out of the vague run, and they are the point of the exercise:

- **It will define "changes" as "any edit"**, because that is the most general reading, and it will therefore notify on a typo fix. Your spec narrowed this to price and availability. That is a decision with a real consequence: the general version trains residents to ignore the count.
- **It will not handle the deleted listing.** Absence and failure cases are consistently the weakest part of unspecified output, in the same way they are the weakest part of unspecified human work. Criterion four exists for exactly this reason.
- **It will probably use `localStorage`** without saying so or asking. That happens to be the right call here - a watch list is not sensitive - but nobody made the call. If the same silent default had been applied to a session token, the previous lesson explains why that is a conversation and not a default.

**The one worth writing up.** Whichever decision you had not realised was a decision. For most people on this feature it is the second one: "changed" felt like an obvious word until something had to be built from it. That is the transferable observation, and it is what the reflection in your assignment is asking for - not that you used a tool, but that you can see where your own thinking was underspecified.

**A closing note about the final project.** Everything above applies to it, and none of it is the same as doing it. When you start, the first thing to write is not code and it is not a prompt. It is the list of things you would have to be able to say yes or no to in order to call it finished.

</details>

---

## Further reading

### On requirements and text

- [User stories with examples and a template](https://www.atlassian.com/agile/project-management/user-stories) by Max Rehkopf, on Atlassian. The "As a... I want to... so that..." format the course uses, explained properly.
- [String.prototype.normalize()](https://developer.mozilla.org/en-US/docs/Web/API/String/normalize) on MDN, for what NFD actually does and does not do.
- [Intl.Collator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/Collator) on MDN. The proper tool for comparing and sorting text in a specific language, Norwegian included, once you outgrow the hand-rolled version in this lesson.

### On prompting, from the people who build the models

A word about these four before you click them. They are written by the companies selling the tools, so they are excellent on **technique** and thin on **limitations** - none of them leads with the reasons you might not want to use the product. Read them for the how. Come back to this lesson for the caveats. Noticing that a source has an interest in the answer is the same skill this whole lesson is about.

They are also the three vendors most likely to be behind whatever tool you end up using, and it is worth seeing that they broadly agree with each other, which is a mild sign that the advice is real rather than branding.

- [Prompt design strategies](https://ai.google.dev/gemini-api/docs/prompting-strategies), Google, for the Gemini API. The clearest of the four if you are starting from nothing.
- [Overview of prompting strategies](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-design-strategies), Google. Longer, and includes a "prompt health checklist" whose first entry is the underspecified task - handling edge cases and missing data rather than assuming your input is always well formed. That is this lesson's argument in Google's own words.
- [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices), Anthropic, for Claude. Notable for how much of it is about telling the model what *not* to do.
- [GPT-5.1 prompting guide](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide), OpenAI. Aimed at people building applications rather than chatting, so skim it - the sections on planning before coding are the relevant ones.

### On coding with these tools

- [Claude Code: best practices for agentic coding](https://www.anthropic.com/engineering/claude-code-best-practices), Anthropic. About their command-line tool specifically, but the general lesson transfers: most of the advice is about giving the tool written context up front, which is a spec by another name.
- [Prompt Engineering Guide](https://www.promptingguide.ai/). Not from a vendor. Broad and academic in places; worth skimming rather than reading.
