# Express

[**Express**](https://expressjs.com/) is a minimal and flexible Node.js web application framework that provides a robust set of features for building web and mobile applications. It simplifies the process of setting up a web server, handling routing, and managing middleware in a Node.js environment.

### Pros of Using Express:
* **Easy to set up**: Express is minimal and easy to learn, making it suitable for beginners and experienced developers alike.
* **Middleware support**: It allows you to use middleware to handle requests, responses, and perform additional operations (like authentication, logging, etc.).
* **Routing**: Express provides a powerful and simple routing mechanism.
* **Large ecosystem**: It has a huge ecosystem of plugins, libraries, and tools to extend functionality.
* **Non-blocking**: Built on top of Node.js, which is non-blocking and asynchronous, making it ideal for handling multiple requests efficiently.

### Cons of Using Express:
* **No structure out of the box**: It doesn’t enforce any specific structure, which can lead to disorganized projects for beginners.
* **Manual configuration**: Unlike some other frameworks, many features (like security, templating, etc.) need to be manually added, which can increase complexity.
* **No built-in database integration**: You need to handle the database connection separately, as Express doesn't come with built-in database support.

## Getting started with Express

Assuming Node is installed (see [Node.js](./nodejs.md)).

### Step 1: Set Up Project

Make a new project folder in your `Repos/` folder, preferably a git repo.

Open your Terminal in the new project folder.

Initialize a New Project:

```bash
npm init -y
```

This will create a `package.json` file to manage dependencies.

### Step 2: Setting Up Express

Install Express:
```bash
npm install express
```

**Note**: We are NOT using the ` --save-dev` flag here.  
We want to install Express under `"dependencies"`, not `"devDependencies"`

Add "start" to "scripts" in `package.json` (remove the "test" script): 

```js
  "scripts": {
    "start": "node app.js"
  },
```

Now you can run the script with `npm run start` and not (only) `node app.js`.

---

### Step 3: Create a Basic Express Application

Your project should look like this:

```bash
express-multipage-app/
│
├── app.js            # Main Express app file
├── routes/
│   ├── home.js       # Home route module
│   ├── about.js      # About route module
│   └── contact.js    # Contact route module
└── package.json      # Project dependencies and metadata
```

Tip: Use Terminal and `touch` and `mkdir`:

```bash
touch app.js
mkdir routes
touch routes/home.js routes/about.js routes/contact.js
```

### Step 4: Add the Code

#### `app.js` (Main Express App): 

```js
const express = require('express');
const app = express();
const PORT = 3000;

// Importing route modules
const homeRoute = require('./routes/home');
const aboutRoute = require('./routes/about');
const contactRoute = require('./routes/contact');

// Middleware to parse request bodies (if needed)
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Route handling
app.use('/', homeRoute); // Homepage route
app.use('/about', aboutRoute); // About page route
app.use('/contact', contactRoute); // Contact page route

// Start the server
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

#### `routes/home.js` (Home Route)
```js
const express = require('express');
const router = express.Router();

// Define the route for the homepage
router.get('/', (req, res) => {
    res.send(`
        <nav>
            <a href="/about">Go to About</a><br>
            <a href="/contact">Go to Contact</a><br>
            <a href="/users">Go to Users</a>
        </nav>
        <h1>Welcome to the Home Page</h1>
    `);
});

module.exports = router;
```

#### `routes/about.js` (About Route)

```js
const express = require('express');
const router = express.Router();

// Define the route for the about page
router.get('/', (req, res) => {
    res.send(`
        <nav>
            <a href="/">Go back to Home</a><br>
            <a href="/contact">Go to Contact</a><br>
            <a href="/users">Go to Users</a>
        </nav>    
        <h1>About Us</h1>
        <p>This is the about page of our Express app.</p>
        <br>
    `);
});

module.exports = router;
```

#### `routes/contact.js` (Contact Route)

```js
const express = require('express');
const router = express.Router();

// Define the route for the contact page
router.get('/', (req, res) => {
    res.send(`
        <nav>
            <a href="/">Go back to Home</a><br>
            <a href="/about">Go to About</a><br>
            <a href="/users">Go to Users</a>
        </nav>
        <h1>Contact Us</h1>
        <form action="/contact" method="post">
            <input type="text" name="name" placeholder="Your Name" required><br>
            <input type="email" name="email" placeholder="Your Email" required><br>
            <button type="submit">Submit</button>
        </form>
    `);
});

// Handle form submission (POST request)
router.post('/', (req, res) => {
    const { name, email } = req.body;
    res.send(`
        <h1>Thank You, ${name}!</h1>
        <p>We will contact you at ${email} soon.</p>
        <a href="/">Go back to Home</a>
    `);
});

module.exports = router;
```

### Step 5: Run the Application

In your terminal:
```bash
npm run start
```

You will possibly get an error: 

```bash
node:events:498
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE: address already in use :::3000
//...
```

If so, this is probably because we already used port 3000 during the [Node.js](./nodejs.md) article.

We then need to change the server settings in the top of `app.js`, use port 3001 instead of 3000:

```js
const PORT = 3001;
```

Now re-run the project with

```bash
npm run start
```

And you should get a message: 

```bash
Server running on http://localhost:3001
```

Click (or copy/paste) the link and open it in your browser.

![](../media/node-express.png)

Test the app:
* Navigate to **About**
* Navigate to **Contact**, and fill the form and submit
* Navigate to **Users**, and you'll get an error (because it's not set up, yet):  
`Cannot GET /users`

Notice, we are using: 
* **Routing**: We used Express's Router to separate route logic for different pages (home, about, contact). Each page is handled in its own module (`home.js`, `about.js`, `contact.js`).
* **Multipage Handling**: Each route serves its own HTML-like content. You can extend this by using template engines like Pug or EJS for dynamic HTML rendering.
* **Form Handling**: In the contact route (`contact.js`), the form data is posted back to the server, which processes and displays a response.

Tip: You exit the server running with ctrl-C (`^C`)

##### Pros of Using Express in this Example:

* **Simplified routing**: Routing is cleanly separated into individual modules, making the app more modular and easier to maintain.
* **Form handling**: The `express.urlencoded()` middleware helps in handling form data in POST requests.

> **Note**: In this example, `express.json()` isn’t necessary unless you intend to handle JSON requests in the future. You only need `express.urlencoded()` for the form submission.

### Step 6: Install `node-fetch` to Handle API Requests

We'll use `node-fetch` (a once popular package for making HTTP requests) to fetch data from the API.

Install it by running:

```bash
npm install node-fetch
```

Note: Again we are not using the `--save-dev` flag.

> This example uses `node-fetch` because it provides a consistent Fetch API in older Node.js versions and some legacy environments. In current versions of Node (v18+), you should just use the built-in `fetch` instead, since it is now available globally and is the recommended choice going forward.

### Step 7: Add Users-page

Let's extend the Express app by adding another page that fetches data from an external API and uses middleware to parse the response before serving it to the client.

We'll follow these steps:

1. **Create a new route**: We'll create a new page (/users) that fetches data from an API (we'll use the Random User API).
2. **Use middleware**: We'll use middleware to handle the API request and format the data before sending it to the user.
3. **Serve the data on the new page**: Display the fetched and parsed data on the new route.

**Update the Project Structure**
Here's the updated structure after adding a new route module for `/users`.

```bash
express-multipage-app/
│
├── app.js              # Main Express app file
├── routes/
│   ├── home.js         # Home route module
│   ├── about.js        # About route module
│   ├── contact.js      # Contact route module
│   └── users.js        # New Users route module (fetches API data)
└── package.json        # Project dependencies and metadata
```

Tip: Use `touch routes/users.js` to create the file.

Then add the code to `routes/users.js`: 

```js
const express = require('express');
const router = express.Router();

// Middleware to fetch and parse API data
async function fetchUserData(req, res, next) {
    try {
        const fetch = await import('node-fetch'); // Dynamically import node-fetch
        const response = await fetch.default('https://randomuser.me/api/');
        const data = await response.json();
        req.userData = data.results[0]; // Attach user data to the request object
        next(); // Proceed to the route handler
    } catch (error) {
        next(error); // Handle errors
    }
}

// Apply the middleware to fetch data for the /users route
router.get('/', fetchUserData, (req, res) => {
    const { name, email, location } = req.userData;
    res.send(`
        <nav>
            <a href="/">Go back to Home</a><br>
            <a href="/about">Go to About</a><br>
            <a href="/contact">Go to Contact</a><br>
        </nav>
        <h1>Random User Information</h1>
        <p><strong>Name:</strong> ${name.first} ${name.last}</p>
        <p><strong>Email:</strong> ${email}</p>
        <p><strong>Location:</strong> ${location.city}, ${location.country}</p>
    `);
});

module.exports = router;
```

#### Sidenote:

If you are on a current version of Node (v18+), you can replace:
```js
const fetch = await import('node-fetch');
const response = await fetch.default('https://randomuser.me/api/');
```
with:
```js
const response = await fetch('https://randomuser.me/api/');
```
This removes the need to install `node-fetch` (step 6), since `fetch` is built into modern Node.js.

### Step 8: Update the Main Express App (`app.js`)
Now, integrate the new route into the main app, both under ***Importing route modules***:

```js
// Importing route modules
const homeRoute = require('./routes/home');
const aboutRoute = require('./routes/about');
const contactRoute = require('./routes/contact');
const usersRoute = require('./routes/users'); // Import the new users route
```

And under ***Route handling***: 

```js
// Route handling
app.use('/', homeRoute); // Homepage route
app.use('/about', aboutRoute); // About page route
app.use('/contact', contactRoute); // Contact page route
app.use('/users', usersRoute); // Users page route (fetches data from API)
```

#### Explanation of the New Route and Middleware:
1. **API Request**: The `/users` route uses `node-fetch` to make a GET request to the Random User API (https://randomuser.me/api/).

2. **Middleware**:
    * The middleware function `fetchUserData` fetches the data from the API and attaches the parsed result (`req.userData`) to the request object.
    * This middleware is applied to the `/users` route.
    * After fetching the data, the middleware calls next() to proceed to the final route handler.

3. **Route Handler**: Once the data is fetched and parsed by the middleware, the route handler formats and sends the data as an HTML response to the client.

### Step 9: Re-Run the app

```bash
node run start
```

Open the app in your Browser and navigate to `/users`, you will see something along the lines of: 

```html
Random User Information
Name: John Doe
Email: john.doe@example.com
Location: New York, United States
```

#### Reloads

Install `nodemon` and add a run script: 
```json
"start-dev": "nodemon app.js"
```
Or, use the simpler and faster (but less configurable) `--watch` flag, built into current versions of Node (again v18+): 
```json
"start-dev": "node --watch app.js"
```

Then re-run:

```bash
node run start-dev
```