# Building a Single Page Application with Vanilla JavaScript

<div style="color: red; background: beige;">
NOTE! This is a work in progress and will not work properly at the moment!
</div>

## Introduction

A Single Page Application (SPA) is a web application that loads a single HTML page and dynamically updates content as the user interacts with the app, without requiring full page reloads. This creates a smoother, more app-like experience.

In this tutorial, we'll build a simple SPA with three pages: Home, About, and Contact. We'll cover routing, dynamic content loading, and browser history management.

---

## Step 1: Set Up the HTML Structure

First, create an `index.html` file with a basic structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vanilla JS SPA</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <nav>
        <a href="/" data-link>Home</a>
        <a href="/about" data-link>About</a>
        <a href="/contact" data-link>Contact</a>
    </nav>
    
    <div id="app"></div>
    
    <script src="app.js"></script>
</body>
</html>
```

**What's happening here:**
- The `<nav>` contains links with a `data-link` attribute. This custom attribute helps us identify which links should be handled by our SPA router rather than causing a full page reload.
- The `<div id="app">` is our content container where different views will be rendered.
- We load our JavaScript at the bottom so the DOM is ready when our script runs.

---

## Step 2: Create Basic CSS

Create a `styles.css` file for basic styling:

```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    padding: 20px;
}

nav {
    background-color: #333;
    padding: 15px;
    margin-bottom: 30px;
    border-radius: 5px;
}

nav a {
    color: white;
    text-decoration: none;
    margin-right: 20px;
    padding: 8px 15px;
    border-radius: 3px;
    transition: background-color 0.3s;
}

nav a:hover {
    background-color: #555;
}

#app {
    max-width: 800px;
    margin: 0 auto;
}
```

---

## Step 3: Create View Classes

Now, let's create our JavaScript. We'll start by defining classes for each view/page. Create an `app.js` file:

```javascript
// Abstract View class that all pages will extend
class AbstractView {
    constructor() {
        // Constructor can be used for initialization
    }
    
    setTitle(title) {
        document.title = title;
    }
    
    async getHtml() {
        // This method will be overridden by child classes
        return "";
    }
}
```

**What's happening:**
- `AbstractView` is a base class that defines the structure all our views will follow.
- `setTitle()` updates the browser tab title.
- `getHtml()` is an async method that returns the HTML content for the view. We make it async to support future enhancements like loading data from APIs.

Now create specific view classes:

```javascript
// Home View
class Home extends AbstractView {
    constructor() {
        super();
        this.setTitle("Home");
    }
    
    async getHtml() {
        return `
            <h1>Welcome Home!</h1>
            <p>This is the home page of our Single Page Application.</p>
            <p>Navigate using the links above - notice how the page doesn't reload!</p>
        `;
    }
}

// About View
class About extends AbstractView {
    constructor() {
        super();
        this.setTitle("About");
    }
    
    async getHtml() {
        return `
            <h1>About Us</h1>
            <p>This SPA was built with vanilla JavaScript.</p>
            <p>No frameworks, no libraries - just pure JS!</p>
        `;
    }
}

// Contact View
class Contact extends AbstractView {
    constructor() {
        super();
        this.setTitle("Contact");
    }
    
    async getHtml() {
        return `
            <h1>Contact</h1>
            <p>Get in touch with us!</p>
            <form id="contact-form">
                <input type="email" placeholder="Your email" required>
                <textarea placeholder="Your message" required></textarea>
                <button type="submit">Send</button>
            </form>
        `;
    }
}

// 404 Not Found View
class NotFound extends AbstractView {
    constructor() {
        super();
        this.setTitle("404 - Page Not Found");
    }
    
    async getHtml() {
        return `
            <h1>404 - Page Not Found</h1>
            <p>The page you're looking for doesn't exist.</p>
            <p><a href="/" data-link>Return to Home</a></p>
        `;
    }
}
```

**What's happening:**
- Each view extends `AbstractView` and implements its own `getHtml()` method.
- The constructor calls `super()` to run the parent class constructor, then sets a unique title.
- Each view returns different HTML content as a template literal.

---

## Step 4: Implement the Router

The router is the heart of our SPA. It matches URLs to views:

```javascript
// Define route patterns and their corresponding views
const pathToRegex = path => new RegExp("^" + path.replace(/\//g, "\\/").replace(/:\w+/g, "(.+)") + "$");

const getParams = match => {
    const values = match.result.slice(1);
    const keys = Array.from(match.route.path.matchAll(/:(\w+)/g)).map(result => result[1]);
    
    return Object.fromEntries(keys.map((key, i) => {
        return [key, values[i]];
    }));
};

const navigateTo = url => {
    history.pushState(null, null, url);
    router();
};

const router = async () => {
    // Define our routes
    const routes = [
        { path: "/", view: Home },
        { path: "/about", view: About },
        { path: "/contact", view: Contact }
    ];
    
    // Test each route for potential match
    const potentialMatches = routes.map(route => {
        return {
            route: route,
            result: location.pathname.match(pathToRegex(route.path))
        };
    });
    
    let match = potentialMatches.find(potentialMatch => potentialMatch.result !== null);
    
    // If no match found, show 404 page
    if (!match) {
        match = {
            route: { path: location.pathname, view: NotFound },
            result: [location.pathname]
        };
    }
    
    // Create instance of the view and get its HTML
    const view = new match.route.view();
    document.querySelector("#app").innerHTML = await view.getHtml();
};
```

**What's happening:**

1. **`pathToRegex()`**: Converts a path string like "/about" into a regular expression that can match URLs. This also supports dynamic parameters like "/user/:id".

2. **`getParams()`**: Extracts URL parameters (useful for dynamic routes like "/user/123").

3. **`navigateTo()`**: This function does two things:
   - Uses `history.pushState()` to add a new entry to the browser's history without reloading the page
   - Calls `router()` to update the view

4. **`router()`**: The main routing function:
   - Defines all available routes and their corresponding view classes
   - Compares the current URL (`location.pathname`) against each route
   - Finds the matching route or shows the 404 NotFound view if no match exists
   - Creates an instance of the matched view class
   - Renders the view's HTML into the `#app` container

---

## Step 5: Handle Navigation Events

Now we need to intercept link clicks and back/forward button presses:

```javascript
// Run router when page loads
document.addEventListener("DOMContentLoaded", () => {
    // Handle navigation links
    document.body.addEventListener("click", e => {
        if (e.target.matches("[data-link]")) {
            e.preventDefault();
            navigateTo(e.target.href);
        }
    });
    
    // Initial route
    router();
});

// Handle browser back/forward buttons
window.addEventListener("popstate", router);
```

**What's happening:**

1. **DOMContentLoaded listener**: Runs when the HTML is fully loaded.
   - Uses event delegation on `document.body` to catch clicks on any link with `[data-link]`
   - When such a link is clicked, prevents the default behavior (page reload) and calls `navigateTo()`
   - Calls `router()` initially to load the current page

2. **popstate listener**: Fires when the user clicks the browser's back or forward buttons
   - Calls `router()` to update the view to match the new URL
   - This is crucial for making the back/forward buttons work in an SPA

---

## Step 6: Add Dynamic Parameters with User Pages

Now let's implement dynamic routing with a real example: a users list and individual user profile pages.

First, create a `users.json` file in your project directory:

```json
[
    {
        "id": 1,
        "name": "Alice Johnson",
        "email": "alice@example.com",
        "role": "Developer"
    },
    {
        "id": 2,
        "name": "Bob Smith",
        "email": "bob@example.com",
        "role": "Designer"
    },
    {
        "id": 3,
        "name": "Carol Williams",
        "email": "carol@example.com",
        "role": "Product Manager"
    },
    {
        "id": 4,
        "name": "David Brown",
        "email": "david@example.com",
        "role": "Engineer"
    }
]
```

Now add the Users list view to your `app.js`:

```javascript
class Users extends AbstractView {
    constructor() {
        super();
        this.setTitle("Users");
    }
    
    async getHtml() {
        try {
            const response = await fetch('/users.json');
            const users = await response.json();
            
            const usersList = users.map(user => `
                <li>
                    <a href="/user/${user.id}" data-link>
                        <strong>${user.name}</strong> - ${user.role}
                    </a>
                </li>
            `).join('');
            
            return `
                <h1>Users</h1>
                <p>Click on a user to view their profile:</p>
                <ul class="users-list">
                    ${usersList}
                </ul>
            `;
        } catch (error) {
            return `
                <h1>Users</h1>
                <p>Error loading users. Please try again.</p>
            `;
        }
    }
}
```

**What's happening:**
- We use `fetch()` to load the JSON file asynchronously
- `await response.json()` parses the JSON data
- `map()` transforms each user object into an HTML list item with a link
- Each link uses `/user/${user.id}` to create dynamic URLs
- The `data-link` attribute ensures these links work with our SPA router
- We include error handling in case the file can't be loaded

Now add the UserProfile view:

```javascript
class UserProfile extends AbstractView {
    constructor(params) {
        super();
        this.userId = params.id;
        this.setTitle("User Profile");
    }
    
    async getHtml() {
        try {
            const response = await fetch('/users.json');
            const users = await response.json();
            const user = users.find(u => u.id === parseInt(this.userId));
            
            if (!user) {
                return `
                    <h1>User Not Found</h1>
                    <p>No user exists with ID: ${this.userId}</p>
                    <p><a href="/users" data-link>Back to Users List</a></p>
                `;
            }
            
            return `
                <h1>${user.name}</h1>
                <div class="user-profile">
                    <p><strong>Role:</strong> ${user.role}</p>
                    <p><strong>Email:</strong> ${user.email}</p>
                    <p><strong>User ID:</strong> ${user.id}</p>
                </div>
                <p><a href="/users" data-link>← Back to Users List</a></p>
            `;
        } catch (error) {
            return `
                <h1>Error</h1>
                <p>Could not load user profile.</p>
                <p><a href="/users" data-link>Back to Users List</a></p>
            `;
        }
    }
}
```

**What's happening:**
- The constructor receives a `params` object containing URL parameters
- We extract `params.id` which comes from the `/user/:id` route pattern
- We fetch the users data and use `find()` to locate the specific user
- `parseInt(this.userId)` converts the string ID from the URL to a number for comparison
- If no user is found, we display a friendly error message
- We provide navigation back to the users list

Finally, update your navigation in `index.html` to include the Users link:

```html
<nav>
    <a href="/" data-link>Home</a>
    <a href="/about" data-link>About</a>
    <a href="/users" data-link>Users</a>
    <a href="/contact" data-link>Contact</a>
</nav>
```

And add some styling to `styles.css`:

```css
.users-list {
    list-style: none;
    padding: 0;
}

.users-list li {
    margin: 15px 0;
    padding: 15px;
    background-color: #f4f4f4;
    border-radius: 5px;
    transition: background-color 0.3s;
}

.users-list li:hover {
    background-color: #e8e8e8;
}

.users-list a {
    color: #333;
    text-decoration: none;
    display: block;
}

.user-profile {
    background-color: #f9f9f9;
    padding: 20px;
    border-radius: 5px;
    margin: 20px 0;
}

.user-profile p {
    margin: 10px 0;
}
```

**Important**: Don't forget to update your routes array in the `router()` function to include both new routes:

```javascript
const routes = [
    { path: "/", view: Home },
    { path: "/about", view: About },
    { path: "/users", view: Users },
    { path: "/user/:id", view: UserProfile },
    { path: "/contact", view: Contact }
];
```

**Key concepts in dynamic routing:**
1. **URL Parameters**: The `:id` syntax in `/user/:id` creates a placeholder that matches any value
2. **Parameter Extraction**: The `getParams()` function extracts these values and passes them to the view
3. **Data Fetching**: Using `fetch()` and `async/await` to load external data
4. **Dynamic Content**: Generating HTML based on data rather than static templates
5. **Error Handling**: Gracefully handling missing data or failed requests

---

## Complete Code Summary

Here's the full `app.js` file:

```javascript
// Abstract View
class AbstractView {
    constructor() {}
    
    setTitle(title) {
        document.title = title;
    }
    
    async getHtml() {
        return "";
    }
}

// Views
class Home extends AbstractView {
    constructor() {
        super();
        this.setTitle("Home");
    }
    
    async getHtml() {
        return `
            <h1>Welcome Home!</h1>
            <p>This is the home page of our Single Page Application.</p>
            <p>Navigate using the links above - notice how the page doesn't reload!</p>
        `;
    }
}

class About extends AbstractView {
    constructor() {
        super();
        this.setTitle("About");
    }
    
    async getHtml() {
        return `
            <h1>About Us</h1>
            <p>This SPA was built with vanilla JavaScript.</p>
            <p>No frameworks, no libraries - just pure JS!</p>
        `;
    }
}

class Contact extends AbstractView {
    constructor() {
        super();
        this.setTitle("Contact");
    }
    
    async getHtml() {
        return `
            <h1>Contact</h1>
            <p>Get in touch with us!</p>
        `;
    }
}

class Users extends AbstractView {
    constructor() {
        super();
        this.setTitle("Users");
    }
    
    async getHtml() {
        try {
            const response = await fetch('/users.json');
            const users = await response.json();
            
            const usersList = users.map(user => `
                <li>
                    <a href="/user/${user.id}" data-link>
                        <strong>${user.name}</strong> - ${user.role}
                    </a>
                </li>
            `).join('');
            
            return `
                <h1>Users</h1>
                <p>Click on a user to view their profile:</p>
                <ul class="users-list">
                    ${usersList}
                </ul>
            `;
        } catch (error) {
            return `
                <h1>Users</h1>
                <p>Error loading users. Please try again.</p>
            `;
        }
    }
}

class UserProfile extends AbstractView {
    constructor(params) {
        super();
        this.userId = params.id;
        this.setTitle("User Profile");
    }
    
    async getHtml() {
        try {
            const response = await fetch('/users.json');
            const users = await response.json();
            const user = users.find(u => u.id === parseInt(this.userId));
            
            if (!user) {
                return `
                    <h1>User Not Found</h1>
                    <p>No user exists with ID: ${this.userId}</p>
                    <p><a href="/users" data-link>Back to Users List</a></p>
                `;
            }
            
            return `
                <h1>${user.name}</h1>
                <div class="user-profile">
                    <p><strong>Role:</strong> ${user.role}</p>
                    <p><strong>Email:</strong> ${user.email}</p>
                    <p><strong>User ID:</strong> ${user.id}</p>
                </div>
                <p><a href="/users" data-link>← Back to Users List</a></p>
            `;
        } catch (error) {
            return `
                <h1>Error</h1>
                <p>Could not load user profile.</p>
                <p><a href="/users" data-link>Back to Users List</a></p>
            `;
        }
    }
}

class NotFound extends AbstractView {
    constructor() {
        super();
        this.setTitle("404 - Page Not Found");
    }
    
    async getHtml() {
        return `
            <h1>404 - Page Not Found</h1>
            <p>The page you're looking for doesn't exist.</p>
            <p><a href="/" data-link>Return to Home</a></p>
        `;
    }
}

// Router functions
const pathToRegex = path => new RegExp("^" + path.replace(/\//g, "\\/").replace(/:\w+/g, "(.+)") + "$");

const getParams = match => {
    const values = match.result.slice(1);
    const keys = Array.from(match.route.path.matchAll(/:(\w+)/g)).map(result => result[1]);
    return Object.fromEntries(keys.map((key, i) => [key, values[i]]));
};

const navigateTo = url => {
    history.pushState(null, null, url);
    router();
};

const router = async () => {
    const routes = [
        { path: "/", view: Home },
        { path: "/about", view: About },
        { path: "/users", view: Users },
        { path: "/user/:id", view: UserProfile },
        { path: "/contact", view: Contact }
    ];
    
    const potentialMatches = routes.map(route => ({
        route: route,
        result: location.pathname.match(pathToRegex(route.path))
    }));
    
    let match = potentialMatches.find(potentialMatch => potentialMatch.result !== null);
    
    if (!match) {
        match = {
            route: { path: location.pathname, view: NotFound },
            result: [location.pathname]
        };
    }
    
    const view = new match.route.view();
    document.querySelector("#app").innerHTML = await view.getHtml();
};

// Event listeners
document.addEventListener("DOMContentLoaded", () => {
    document.body.addEventListener("click", e => {
        if (e.target.matches("[data-link]")) {
            e.preventDefault();
            navigateTo(e.target.href);
        }
    });
    
    router();
});

window.addEventListener("popstate", router);
```

---

## Key Concepts Recap

1. **History API**: `history.pushState()` allows us to change the URL without reloading the page, and `popstate` events let us respond to back/forward navigation.

2. **Event Delegation**: By listening for clicks on `document.body` rather than individual links, our code works even for dynamically added links.

3. **Class Inheritance**: Using an abstract base class ensures all views follow the same interface.

4. **Async/Await**: Making `getHtml()` async prepares our code for future enhancements like API calls.

5. **Regular Expressions**: Pattern matching enables flexible routing, including support for URL parameters.

---

## Testing Your SPA

To test locally, you'll need a development server because modern browsers restrict `file://` protocol access and your SPA needs to handle routing properly.

**Using VS Code's Live Server extension:**

1. Install the "Live Server" extension by Ritwick Dey from the VS Code marketplace
2. Open your project folder in VS Code
3. Right-click on `index.html` and select "Open with Live Server"
4. Your browser will open at `http://localhost:5500` (or similar)
5. The page will automatically reload when you make changes to your files

Once running, test your SPA by:
- Clicking through all navigation links
- Using the browser's back and forward buttons
- Typing URLs directly (like `/users` or `/user/2`)
- Testing the 404 page with an invalid URL like `/nonexistent`

---

## Conclusion

You've now built a fully functional SPA using only vanilla JavaScript! While frameworks like React and Vue provide additional features and optimizations, understanding these fundamentals helps you appreciate what frameworks do under the hood and makes you a better developer overall.