# Testing JavaScript with Jest: A Short, Practical Introduction

Modern JavaScript development moves fast, and bugs often slip in where you least expect them. Automated tests help you catch those mistakes early and give you the confidence to change code without fear of breaking something else. Jest is one of the most popular tools for this: lightweight, fast, and easy to learn.

This short lesson walks you through the essentials of testing JavaScript with Jest. Each section uses real examples, and along the way you’ll see how testing naturally leads to cleaner, more maintainable code.

## Installing and Running Jest
You can add Jest to any JavaScript project — whether you’re starting fresh or upgrading an existing codebase.

**1. Install Jest**
```js
npm install --save-dev jest
```
This adds Jest as a development dependency. It works the same way whether your project is brand new or has been around for years.

**2. Add a Test Script**
Open your package.json and add a test command:
```json
{
  "scripts": {
    "test": "jest"
  }
}
```
If the file already has a scripts section, just add the test line.

**3. Create Your First Test File**
Jest automatically runs any file that ends with `.test.js` or `.spec.js`.

You can create a folder called `tests/` or keep test files next to the code — both approaches work.

Example:
```md
add.js
add.test.js
```
**4. Run Jest**
```sh
npm test
```
Jest will find all test files, run them, and show you which tests passed or failed.

That’s the entire setup. From here, you can start adding test files one by one, whether the project is brand new or already full of code.

## Your First Test: Start Small with a Pure Function
A great way to learn testing is to begin with a tiny function that has no side effects.
```js
// add.js
export function add(a, b) {
  return a + b;
}
```
Now create the matching test file:
```js
// add.test.js
import { add } from './add.js';

test('adds two numbers', () => {
  const result = add(2, 3);
  expect(result).toBe(5);
});
```
Run the test and you should see a green result.

This simple pattern shows the bare essentials:
* Clear test names describe behaviour.
* Organise tests using Arrange (set up), Act (call function), Assert (check outcome).
* Small, predictable functions lead to simple, reliable tests.

## Covering Real Behaviour: Think in Cases, Not Just Happy Paths
Most functions have rules, exceptions, and edge cases. Tests should reflect that.

Consider a password validator:
```js
// password.js
export function isValidPassword(str) {
  // Valid if it's a string, at least 6 chars, and contains a digit.
  return typeof str === 'string' &&
         str.length >= 6 &&
         /\d/.test(str);
}
```
A useful test suite covers both valid and invalid cases:
```js
// password.test.js
import { isValidPassword } from './password.js';

describe('isValidPassword', () => {
  test('accepts a long, mixed password', () => {
    expect(isValidPassword('hello123')).toBe(true);
  });

  test('rejects passwords that are too short', () => {
    expect(isValidPassword('a1')).toBe(false);
  });

  test('rejects empty input', () => {
    expect(isValidPassword('')).toBe(false);
  });
});
```
Grouping related tests with describe improves readability, especially as your project grows.

A simple rule helps here:  
If you can phrase a rule in a sentence, you can write a test for it.

## Making Code Testable by Separating Logic from Side Effects
One advantage of testing is that it exposes code that is difficult to reason about. A common culprit is mixing logic with side effects such as DOM updates, storage operations, or network calls.

Here’s an example of code that looks fine but is hard to test:
```js
function showTotal() {
  const cart = JSON.parse(localStorage.getItem('cart'));
  let total = 0;

  cart.forEach(item => {
    total += item.price * item.quantity;
  });

  document.querySelector('#total').textContent = total + ' kr';
}
```

This function reads storage, calculates a total, and updates the DOM. Testing it would require mocking all of that.

A small refactor makes a big difference:
```js
// cart.js
export function calculateCartTotal(cart) {
  return cart.reduce((sum, item) => sum + item.price * item.quantity, 0);
}
```
```js
// ui.js
import { calculateCartTotal } from './cart.js';

export function showTotal() {
  const cart = JSON.parse(localStorage.getItem('cart'));
  const total = calculateCartTotal(cart);
  document.querySelector('#total').textContent = total + ' kr';
}
```
Now the pure logic is isolated and easy to test:
```js
// cart.test.js
import { calculateCartTotal } from './cart.js';

test('calculates total from price and quantity', () => {
  const cart = [
    { price: 100, quantity: 2 },
    { price: 50, quantity: 1 }
  ];

  expect(calculateCartTotal(cart)).toBe(250);
});
```

This pattern is at the heart of good testable design:
* Keep logic and side effects apart.
* Test the pure logic.
* Keep the side-effect code thin and simple.

## Testing Async Code: Mocking Fetch
Real applications often fetch data from APIs. Unit tests shouldn’t actually hit those APIs, so you mock them.

Given this function:
```js
// api.js
export async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) throw new Error('Network error');
  return response.json();
}
```
Here is a test that simulates both success and failure:
```js
// api.test.js
import { fetchUser } from './api.js';

global.fetch = jest.fn();

test('returns user data when the request succeeds', async () => {
  const mockUser = { id: 1, name: 'Alice' };

  fetch.mockResolvedValue({
    ok: true,
    json: () => Promise.resolve(mockUser)
  });

  const result = await fetchUser(1);
  expect(result).toEqual(mockUser);
});
```
And a test for error handling:
```js
test('throws an error when the request fails', async () => {
  fetch.mockResolvedValue({ ok: false });

  await expect(fetchUser(1)).rejects.toThrow('Network error');
});
```
This demonstrates three key points:
* Don’t call real APIs in unit tests.
* Always test both the success path and the failure path.
* Use mocks to simulate behaviour instead of relying on external systems.

## How Much Should You Test?
You don’t have to test everything. Instead, focus on parts of your code that matter most:
* Logic with branches and rules.
* Code that regularly breaks.
* Code you plan to refactor.
* Functions with important business behaviour.

Test coverage tools can help you spot gaps, but coverage is not a goal on its own. Aim for tests that are fast, clear, and stable.

## Quick Checklist for Writing Good Tests
* One behaviour per test.
* Clear, descriptive test names.
* Small, predictable functions are easier to test.
* Keep side effects separate from logic.
* Mock external systems.
* Avoid randomness and large setups.
* Prefer pure functions where possible.

## Wrap-up and a Small Challenge
Testing isn’t just about catching mistakes. It has a shaping effect on your code: when something is hard to test, it often means the design can be improved. Jest makes it simple to create small, focused tests that both document and protect your code.

Try this:
* Pick any project you’ve already built.
* Write tests for three functions:
    1. a pure utility function
    2. a function with edge cases
    3. an async function that fetches data
* If any of them feel hard to test, refactor them until they’re easy.

This kind of practice builds confidence quickly, and the quality of your code improves as a side effect.
