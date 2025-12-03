# Node.js

**Node.js** is a JavaScript runtime environment that allows developers to run JavaScript code on the server side, outside of a browser.

## Key Features:
* **Asynchronous, non-blocking I/O**: Efficient handling of multiple tasks without waiting for previous ones to complete.
* **Fast execution**: Built on the V8 JavaScript engine, making it highly performant.
* **Single-threaded**: Uses a single-threaded model with event looping, which helps in handling multiple connections efficiently.
* **Rich ecosystem**: Access to a large collection of libraries through npm (Node Package Manager).

## Use Cases:
* **Real-time applications**: Chat apps, online gaming, live tracking.
* **APIs**: Building RESTful APIs for mobile or web apps.
* **Microservices**: Modular, lightweight applications.
* **Web servers**: Handling client requests and responses.

## Getting started with Node.js

Let's walk through a simple Node.js project that demonstrates how to use modules, including one that fetches data from an external API.

Assuming Node is installed on your local computer. 

Check using the following commands in your favourite bash/zsh Terminal:

```bash
node -v
npm -v
```

If not installed, download it from [nodejs.org](https://nodejs.org/), or install using [Homebrew](https://brew.sh) (for Mac) or similar.

### Step 1: Set Up Node.js Project

Make a new project folder in your `Repos/` folder (or wherever you place your work.)

**Tip**: Make a new github repo, including a README, and a `.gitignore` using the Node template, and clone that that to your computer.

Open your Terminal in the new project folder (eg. the built in Terminal in Visual Studio Code).

Initialize a New Project:

```bash
npm init -y
```

This will create a `package.json` file to manage dependencies.

### Step 2: Create the Project Structure
Your project will have two modules:

1. `main.js`: The main file that will run your program.
2. `apiModule.js`: A module that fetches data from an external API.

Tip: Use `touch main.js apiModule.js` to make the files:

```bash
nodejs-api-example/
│
├── main.js       # Main file
├── apiModule.js  # Module for API data
└── package.json
```

### Step 3: Write the Code

#### 1. `apiModule.js` (Module to fetch data from an API)
Here you will use the built-in https module to fetch data from the API.

**Note**: 
* **`https.get()`**: This function makes a GET request to the provided URL.
* **Handling Chunks**: Since the data is streamed, we collect the chunks into a single string (data).
* **Parsing Data**: Once all the data is received, we parse the JSON and extract the necessary information.

```js
const https = require('https');

// Function to fetch random user data using Node.js's built-in https module
function getRandomUserData() {
    return new Promise((resolve, reject) => {
        const url = 'https://randomuser.me/api/';

        https.get(url, (res) => {
            let data = '';

            // A chunk of data has been received.
            res.on('data', (chunk) => {
                data += chunk;
            });

            // The whole response has been received.
            res.on('end', () => {
                try {
                    const parsedData = JSON.parse(data);
                    const userData = parsedData.results[0];
                    resolve({
                        name: `${userData.name.first} ${userData.name.last}`,
                        email: userData.email,
                        location: userData.location.city
                    });
                } catch (error) {
                    reject('Error parsing data: ' + error.message);
                }
            });
        }).on('error', (err) => {
            reject('Error fetching data: ' + err.message);
        });
    });
}

module.exports = { getRandomUserData };
```

2. `main.js` (Main file)
Here, you’ll import the module and display the fetched data.

```js
const { getRandomUserData } = require('./apiModule');

// Use the module's function to get and display data
async function main() {
    try {
        const user = await getRandomUserData();
        console.log('Random User Data:');
        console.log(`Name: ${user.name}`);
        console.log(`Email: ${user.email}`);
        console.log(`Location: ${user.location}`);
    } catch (error) {
        console.error(error);
    }
}

main();
```

### Step 4: Run the Project
Now, run the project in the terminal:

```bash
node main.js
```

This will fetch the random user data and display it in the Terminal.

Add `console.log(userData);` to the `apiModule.js`:

```js
    try {
        const parsedData = JSON.parse(data);
        const userData = parsedData.results[0];
        console.log(userData); // Add this line
        resolve({
            name: `${userData.name.first} ${userData.name.last}`,
            email: userData.email,
            location: userData.location.city
        });
    }
```

Use `npm run` instead of:

```bash
node main.js
```

In `package.json`, edit the "scripts": 

```js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
}
```

Remove everything in "scripts" and replace it with:

```js
"scripts": {
    "main": "node main.js"
}
```

Now, run the project with:

```bash
npm run main
```

### Step 5: Set Up the Server

Make a new file in the project: `index.js`.

Tip: Use `touch index.js`

```bash
nodejs-api-example/
│
├── index.js      # New Main file
├── main.js       # Old Main file/test file
├── apiModule.js  # Module for API data
└── package.json
```

In the `index.js` file we will: 

* Include an HTTP server.
* Fetch data from the API when a request is made to the server
* Serve the data as JSON

```js
const http = require('http');
const { getRandomUserData } = require('./apiModule');

// Create a server that listens for requests and responds with the API data
const server = http.createServer(async (req, res) => {
    if (req.url === '/user' && req.method === 'GET') {
        try {
            // Fetch the random user data from the external API
            const user = await getRandomUserData();

            // Set the response headers to serve JSON
            res.writeHead(200, { 'Content-Type': 'application/json' });
            
            // Respond with the user data in JSON format
            res.end(JSON.stringify(user));
        } catch (error) {
            // Handle errors and respond with a 500 error status
            res.writeHead(500, { 'Content-Type': 'application/json' });
            res.end(JSON.stringify({ error: 'Failed to fetch user data' }));
        }
    } else {
        // Handle invalid routes or methods
        res.writeHead(404, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Route not found' }));
    }
});

// Start the server and listen on port 3000
const PORT = 3000;
server.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

### Step 6: Run the Server
Now, you can run the server by running the following command in your terminal:

```bash
node index.js
```

Now the Terminal shows: 

```bash
Server running on http://localhost:3000
```

You can click (or copy/paste) the link to open it in your browser.

*Tip*: "You exit the server running with ctrl-C (`^C`)"

Note: You will get an error: 

```sh
error	"Route not found"
```
(or `{"error":"Route not found"}`)

You need to navigate to http://localhost:3000/user to see some random user data:

Let's update our server settings to reflect this. In `index.js`, change the server settings: 

```js
server.listen(PORT, () => {
    // console.log(`Server running on http://localhost:${PORT}`);
    console.log(`Server running the example on http://localhost:${PORT}/user`);
});
```

Also, while we're at it, add another "script" to the `package.json` file (remember the `,`):

```js
  "scripts": {
    "main": "node main.js",
    "start": "node index.js"
  }
```

Now re-run the server with:

```bash
npm run start
```

Now the Terminal prompt says: 

```bash
Server running the example on http://localhost:3000/user
```

And you can click (or copy/paste) the link to get to the user Data.

Note that if you do changes to you code, you need to restart the server to see the changes.

To avoid this we can install a tool: **Nodemon**

#### Nodemon

**Nodemon** is a utility that helps with the development of Node.js applications by automatically restarting the application whenever files change. 

It monitors changes in the directory where the application runs, so you don't have to manually stop and restart the server each time you make a change.

Benefits of Using Nodemon:
* Automatic restarts: Your server will automatically restart when you save changes to any file.
* Improved development speed: You don’t need to manually stop and restart the server.

*Note that Nodemon does NOT refresh the web page, that you need to do manually!*

### Step 7:  Install Nodemon
You can install `nodemon` globally or as a development dependency for your project.

Globally (available system-wide):
```bash
npm install -g nodemon
```

Locally (within the project):
```bash
npm install nodemon --save-dev
```

Installing it locally adds it to your project's `package.json` file under devDependencies.

Here, install it locally under devDependencies with `npm install nodemon --save-dev`.

Also, update the "script" in `package.json`:

```js
  "scripts": {
    "test": "node test.js",
    "start": "node index.js",
    "start-dev": "nodemon index.js"
  },
```

Remember the comma `,` on the line above!

With this change, you can run nodemon using:

```bash
npm run start-dev
```

How Nodemon Works in the Example:
* Nodemon will detect changes in your `index.js` or `apiModule.js` file.
* It will restart the server each time you save your changes, allowing you to see updates without manually stopping and restarting the server.

Test it: 

* Comment out `console.log(userData);` in `apiModule.js`
* In `index.js` add a `console.log (user);` after it is fetched, but before you make the JSON response:

```js
// Fetch the random user data from the external API
const user = await getRandomUserData();

console.log (user);

// Set the response headers to serve JSON
res.writeHead(200, { 'Content-Type': 'application/json' });
```

* Now add another line in `index.js` just above the `console.log`:

```js
user.id = Math.floor(Math.random() * 10000000);
console.log (user);
```

Notice that `[nodemon] restarting due to changes...`

Now **refresh your browser**, and see the changes (both in the Browser and the Terminal).


#### Postman

Last tip: You can also use [Postman](https://www.postman.com/) to look at the data: 

![h:560px](../media/postman-node.png)
