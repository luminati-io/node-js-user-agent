# Setting and Changing User Agent in Node.js

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.com/) 

This guide explains hot to set the `User-Agent` header with Node.js and implement user agent rotation to evade anti-bot detection:

- [How to Change the Node.js User Agent Using the Fetch API](#how-to-change-the-nodejs-user-agent-using-the-fetch-api)
  - [Set a User Agent Locally](#set-a-user-agent-locally)
  - [Set a User Agent Globally](#set-a-user-agent-globally)
- [Implement User Agent Rotation in Node.js](#implement-user-agent-rotation-in-nodejs)
  - [Step 1: Retrieve a List of User Agents](#step-1-retrieve-a-list-of-user-agents)
  - [Step 2: Randomly Pick a User Agent](#step-2-randomly-pick-a-user-agent)
  - [Step 3: Make the HTTP Request with a Random User Agent](#step-3-make-the-http-request-with-a-random-user-agent)
  - [Step 4: Put It All Together](#step-4-put-it-all-together)

## Why Setting a User Agent Is So Important

The [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) header is a string that identifies the client making an HTTP request. It usually contains details about the browser, application, operating system, and system architecture.

For example, here is the user agent string set by Chrome when making a request:

```
Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36
```

Below is a breakdown of the components in this user agent string:

* `Mozilla/5.0`: Originally used to denote compatibility with Mozilla browsers, this prefix is now added for compatibility purposes.
* `Windows NT 10.0; Win64; x64:` Indicates the operating system (`Windows NT 10.0`), platform (`Win64`), and system architecture (`x64`).
* `AppleWebKit/537.36`: Refers to the browser engine that Chrome uses.
* (`KHTML, like Gecko`): Shows compatibility with the KHTML and Gecko layout engines.
* `Chrome/127.0.0.0`: Specifies the browser name and version.
* `Safari/537.36`: Indicates compatibility with Safari.

The `User-Agent` header helps determine whether a request comes from a trusted browser or automated software.

Web scraping bots often use default or non-browser user agents, making them easy targets for anti-bot systems. These systems analyze the `User-Agent` header to distinguish real users from bots.

## What Is the Node.js Default User Agent?

Since version 18, Node.js includes [`fetch()`](https://nodejs.org/dist/latest/docs/api/globals.html) as a built-in implementation of the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API). It is the recommended way to perform HTTP requests in Node.js without external dependencies. Learn more in our guide on [HTTP requests in Node.js with Fetch API](/blog/how-tos/fetch-api-nodejs).

Like most HTTP clients, `fetch()` automatically sets a default `User-Agent` header. The same behavior occurs in the [Python `requests` library](/faqs/python-requests/what-is-python-requests).

By default, `fetch()` in Node.js sets the following `User-Agent` string:

```
node
```

You can check the default `User-Agent` set by `fetch()` by making a GET request to [`httpbin.io/user-agent`](https://httpbin.io/user-agent). This endpoint returns the `User-Agent` header of the incoming request, allowing you to identify the user agent used by an HTTP client.

To test this, create a Node.js script, define an [`async`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) function, and use `fetch()` to make the request:

```js
async function getFetchDefaultUserAgent() {

// make an HTTP request to the HTTPBin endpoint

// to get the user agent

const response = await fetch("https://httpbin.io/user-agent");

// read the default user agent from the response

// and print it

const data = await response.json();

console.log(data);

}

getFetchDefaultUserAgent();
```

Execute the JavaScript code above, and you will receive the following string:

```
{ 'user-agent': 'node' }
```

By default, `fetch()` in Node.js sets the `User-Agent` to `node`, which differs significantly from browser user agents. This can trigger [anti-bot systems](/webinar/bot-detection).

Anti-bot solutions detect unusual user agents and flag such requests as bots, leading to blocks. Changing the default Node.js `User-Agent` helps avoid detection.

## How to Change the Node.js User Agent Using the Fetch API

The Fetch API specification does not include a built-in method for changing the `User-Agent`. However, since it is just an HTTP header, you can customize its value using [`fetch()` header options](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch#setting_headers).

**Set a User Agent Locally**

`fetch()` supports header customization via the `headers` option. Use it to set the `User-Agent` header when making a specific HTTP request as follows:

```js
const response = await fetch("https://httpbin.io/user-agent", {

headers: {

"User-Agent":

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

},

});
```

Put it all together and you will get:

```js
async function getFetchUserAgent() {

// make an HTTP request to HTTPBin

// with a custom user agent

const response = await fetch("https://httpbin.io/user-agent", {

headers: {

"User-Agent":

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

},

});

// read the default user agent from the response

// and print it

const data = await response.json();

console.log(data);

}

getFetchUserAgent();
```

Launch the above script, and this time the result will be:

```
{

'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36'

}
```

**Set a User Agent Globally**

While setting the `User-Agent` per request is simple, it can lead to repetitive code. However, the `fetch()` API does not currently support global overrides for its default settings.

To work around this, you can create a wrapper function to customize `fetch()` with your desired configurations:

```js
function customFetch(url, options = {}) {

// custom headers

const customHeaders = {

"User-Agent":

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

...options.headers, // merge with any other headers passed in the options

};

const mergedOptions = {

...options,

headers: customHeaders,

};

return fetch(url, mergedOptions);

}
```

You can now make an HTTP request with a custom user agent by calling `customFetch()` instead of `fetch()`:

```js
const response = await customFetch("https://httpbin.io/user-agent");
```

The complete Node.js script will be:

```js
function customFetch(url, options = {}) {

// add a custom user agent header

const customHeaders = {

"User-Agent":

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

...options.headers, // merge with any other headers passed in the options

};

const mergedOptions = {

...options,

headers: customHeaders,

};

return fetch(url, mergedOptions);

}

async function getFetchUserAgent() {

// make an HTTP request to HTTPBin

// through the custom fetch wrapper

const response = await customFetch("https://httpbin.io/user-agent");

// read the default user agent from the response

// and print it

const data = await response.json();

console.log(data);

}

getFetchUserAgent();
```

Launch the Node.js script above, and it will print:

```
{

'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36'

}
```

## Implement User Agent Rotation in Node.js

Simply replacing the default `User-Agent` with a browser string may not bypass anti-bot detection. If multiple requests originate from the same IP with the same user agent, anti-scraping systems can still flag the activity as automated.

To reduce detection risk in Node.js, introduce variability in your requests. One effective method is **user agent rotation**, where the `User-Agent` header changes on each request. This makes your requests appear to come from different browsers, reducing the chance of being blocked.

Let's implement user agent rotation in Node.js.

### Step #1: Retrieve a List of User Agents

Visit a site like [WhatIsMyBrowser.com](https://www.whatismybrowser.com/guides/the-latest-user-agent/) and populate a list of some valid user agent values:

```js
const userAgents = [

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.5 Safari/605.1.15",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/126.0.2592.113",

// other user agents...

];
```

> **Tip**:
> 
> The more real-world user agent strings this array contains, the better to avoid anti-bot detection.

### Step #2: Randomly Pick a User Agent

Create a function that randomly selects and returns a user agent string from the list:

```js
function getRandomUserAgent() {

const userAgents = [

// user agents omitted for brevity...

];

// return a user agent randomly

// extracted from the list

return userAgents[Math.floor(Math.random() * userAgents.length)];

}
```

Let’s break down what happens in this function:

* [`Math.random()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random) generates a random number between 0 and 1
* This number is then multiplied by the length of the `userAgents` array.
* [`Math.floor()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/floor) rounds down the resulting number to the largest integer less than or equal to that number.
* The resulting number from the previous operations corresponds to a randomly generated index that goes from 0 to `userAgents.length - 1`.
* The index is then used to return a random user agent from the array of user agents.

Every time you call the `getRandomUserAgent()` function, you will likely get a different user agent.

### Step #3: Make the HTTP Request with a Random User Agent

To implement user agent rotation in Node.js using `fetch()`, set the `User-Agent` header with the value from the `getRandomUserAgent()` function:

```js
const response = await fetch("https://httpbin.io/user-agent", {

headers: {

"User-Agent": getRandomUserAgent(),

},

});
```

The HTTP request performed via the Fetch API will now have a random user agent.

### Step #4: Put It All Together

Add the snippets of the previous steps to a Node.js script, and then wrap the logic for making a `fetch()` request in an `async` function.

Here is what your final Node.js user agent rotation script should look like:

```js
function getRandomUserAgent() {

const userAgents = [

"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:128.0) Gecko/20100101 Firefox/128.0",

"Mozilla/5.0 (Macintosh; Intel Mac OS X 14_6) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/17.5 Safari/605.1.15",

"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36 Edg/126.0.2592.113",

// other user agents...

];

// return a user agent randomly

// extracted from the list

return userAgents[Math.floor(Math.random() * userAgents.length)];

}

async function getFetchUserAgent() {

// make an HTTP request with a random user agent

const response = await fetch("https://httpbin.io/user-agent", {

headers: {

"User-Agent": getRandomUserAgent(),

},

});

// read the default user agent from the response

// and print it

const data = await response.json();

console.log(data);

}

getFetchUserAgent();
```

Run the script 3 or 4 times. Statistically, you should see different user agent responses as below:

![different user agent responses](https://brightdata.com/wp-content/uploads/2024/08/different-user-agent-responses-1024x298.png)

This demonstrates that the user agent rotation is working effectively.

Et voilà! You are now skilled at setting user agents in Node.js using the Fetch API.

## Conclusion

Implementing user agent rotation in Node.js helps evade basic anti-scraping systems. Yet, more advanced systems can still detect and block your automated requests. To avoid IP bans, consider [Web Scraper API](https://brightdata.com/products/web-scraper) that effectively bypasses anti-scraping measures through features like IP and user agent rotation, making web scraping easier than ever.

Sign up now and start your free trial today!