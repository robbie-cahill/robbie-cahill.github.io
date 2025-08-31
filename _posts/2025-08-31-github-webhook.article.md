---
title: 'GitHub Webhook: A Complete Guide to Automation'
description: >-
  Learn how to set up and use GitHub Webhooks from scratch. This comprehensive
  guide covers creating a local server, exposing it with a public URL, and
  securing your endpoints.
tags:
  - github webhook
  - github
  - webhook
  - automation
  - ci/cd
  - nodejs
  - expressjs
  - opensource
  - javascript
  - typescript
  - python
  - programming
  - coding
canonical_url: 'https://softwareengineeringstandard.com/2025/08/31/github-webhook/'
slug: github-webhook
cover_image: >-
  https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp
claire_approved: true
human_approved: true
published: true
---

In the world of modern software development, automation is the key to efficiency, consistency, and speed. One of the most powerful tools for automation within the developer ecosystem is the **GitHub webhook**. Whether you're looking to build a CI/CD pipeline, send notifications to Slack, or trigger custom deployment scripts, understanding GitHub webhooks is a fundamental skill.

This comprehensive guide will walk you through everything you need to know about GitHub webhooks. We'll start with the basics, build a real-world example using Node.js, and show you how to securely test your integration locally using open-source tools. By the end, you'll be able to confidently integrate GitHub webhooks into your own projects to automate your workflows.

## What Exactly is a Webhook?

Before diving into the specifics of GitHub, it's crucial to understand the core concept of a webhook.

Traditionally, if you wanted to know if a status changed in another system (like a new sale on your e-commerce store), you would have to constantly ask it, "Is there anything new? Is there anything new? Is there anything new?" This process is called **polling**. It's inefficient, resource-intensive, and there's always a delay between the event happening and your system finding out about it.

Webhooks flip this model on its head. Instead of you asking for new information, the service tells you about it the moment it happens. This is often described as a "reverse API."

Here’s a simple breakdown:
1.  You provide a URL (your webhook endpoint) to a service (the webhook provider).
2.  You tell the provider which events you're interested in (e.g., a "new purchase").
3.  When that event occurs, the provider immediately sends an HTTP request (usually a `POST` request) to your URL with a payload of data describing the event.
4.  Your application, which is listening at that URL, receives the data and can take immediate action.

This event-driven approach is far more efficient and enables real-time integrations between different services.

## Understanding the GitHub Webhook Ecosystem

A **GitHub webhook** works on the exact principles described above. You can configure your GitHub repository to send a webhook payload to a specified URL whenever a certain event occurs. The range of events GitHub supports is vast, making it an incredibly powerful tool for automation.

Some of the most common events you can subscribe to include:

*   **`push`**: Triggered whenever a commit is pushed to a branch. This is the cornerstone of most CI/CD pipelines.
*   **`pull_request`**: Triggered when a pull request is opened, closed, reopened, or synchronized. Essential for code review automation.
*   **`issues`**: Triggered when an issue is opened, edited, closed, or labeled. Useful for project management integrations.
*   **`release`**: Triggered when a new release is published. Perfect for automating deployment notifications or package publishing.
*   **`fork`**: Triggered when a repository is forked.
*   **`workflow_run`**: Triggered when a GitHub Actions workflow is requested or completed.

Each event sends a unique **payload**—a JSON body containing detailed information about the event. For example, a `push` event payload includes the repository name, the branch that was pushed to, the commit hashes, the author's details, and more. Your application can parse this JSON to perform context-aware actions.

## Building Our First GitHub Webhook Receiver

The best way to learn is by doing. We're going to build a simple webhook receiver that listens for `push` events from a GitHub repository. When a push occurs, our application will log the committer's name and the commit message to the console.

To do this, we need three key components:
1.  A web server running locally to act as our webhook endpoint.
2.  A way to give our local server a public URL that GitHub can reach.
3.  A webhook configured in our GitHub repository to point to that public URL.

### Prerequisites

*   **Node.js and npm**: Make sure you have a recent version of Node.js installed. You can get it from [nodejs.org](https://nodejs.org/).
*   **A GitHub Account**: You'll need an account and a repository to test with. If you don't have one, create a new demo repository.
*   **A Text Editor**: Visual Studio Code or any other editor of your choice.

### Step 1: Create a Local Web Server with Express.js

We'll use Express.js, a minimal and flexible Node.js web application framework, to create our server.

First, create a new project directory and initialize it with npm.

```bash
mkdir github-webhook-receiver
cd github-webhook-receiver
npm init -y
```

Now, install Express:

```bash
npm install express
```

Next, create a file named `app.js` and add the following code:

```javascript
/*
 * app.js - A simple Express server to receive GitHub webhooks.
 */

const express = require('express');

// Create the Express app
const app = express();
const port = 8080;

// Middleware to parse JSON bodies. GitHub sends webhooks as JSON.
// We need the raw body for signature verification, so we'll use a custom parser.
app.use(express.json({
    verify: (req, res, buf) => {
        req.rawBody = buf;
    }
}));


// Define the endpoint for our GitHub webhook
app.post('/github-webhook', (req, res) => {
    // The event type is in the X-GitHub-Event header
    const githubEvent = req.headers['x-github-event'];

    console.log(`Received a webhook for the '${githubEvent}' event`);
    console.log('---');

    // We're only interested in 'push' events for this demo
    if (githubEvent === 'push') {
        const payload = req.body;
        
        // Extract relevant information from the push payload
        const repositoryName = payload.repository.full_name;
        const commitAuthor = payload.head_commit.author.name;
        const commitMessage = payload.head_commit.message;
        const branch = payload.ref.split('/').pop();

        console.log(`New push to repository: ${repositoryName}`);
        console.log(`Branch: ${branch}`);
        console.log(`Author: ${commitAuthor}`);
        console.log(`Commit Message: "${commitMessage}"`);
    }

    // Always respond with a 200 OK to let GitHub know the webhook was received successfully.
    // If GitHub doesn't receive a 2xx response, it will consider the delivery a failure
    // and may retry, leading to duplicate processing.
    res.status(200).send('Webhook received.');
});

// A simple root endpoint to confirm the server is running
app.get('/', (req, res) => {
    res.send('Server is up and running. Ready to receive GitHub webhooks at /github-webhook.');
});

// Start the server
app.listen(port, () => {
    console.log(`Server started on http://localhost:${port}`);
});
```

Let's break down this code:
*   We initialize an Express app and tell it to listen on port `8080`.
*   Crucially, we use `express.json()` to parse incoming JSON payloads from GitHub.
*   We define a single endpoint: `POST /github-webhook`. This is the URL we'll give to GitHub.
*   Inside the handler, we check the `x-github-event` header to ensure it's a `push` event.
*   We then parse the `req.body` (the JSON payload) to extract and log the repository name, author, and commit message.
*   Finally, we send a `200 OK` response. This is **vital**. If GitHub doesn't get a `2xx` response, it will assume the delivery failed and retry, which could cause your automation to run multiple times for a single event.

Start your server by running:

```bash
node app.js
```

You should see the message: `Server started on http://localhost:8080`. If you visit `http://localhost:8080` in your browser, you'll see our confirmation message.

### Step 2: Expose Your Local Server with Tunnelmole

Our server is running, but it's only accessible on `localhost`. GitHub's servers on the public internet have no way to reach it. We need to create a secure tunnel from a public URL to our local machine.

This is where **Tunnelmole** comes in. Tunnelmole is an open-source tool that creates a public HTTPS URL for your locally running servers. It's simple, fast, and perfect for testing webhooks.

First, install Tunnelmole. The simplest way is to use the install script for Linux, Mac, or WSL.

```bash
# For Linux, macOS, and WSL
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```
For Windows users, you can download the executable and add it to your PATH. Visit the [Tunnelmole website](https://tunnelmole.com) for the latest installation instructions.

Once installed, open a **new terminal window** (leave your Node.js server running in the first one) and run the following command, telling Tunnelmole to forward traffic to your local port `8080`.

```bash
tmole 8080
```

Tunnelmole will start and generate a unique public URL that forwards to your local server. The output will look something like this:

```
$ tmole 8080
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://k2e6yq-ip-12-34-56-78.tunnelmole.net ⟶ http://localhost:8080
http://k2e6yq-ip-12-34-56-78.tunnelmole.net ⟶ http://localhost:8080
```

**Copy the `https` URL**. This is the public address for your webhook. Anyone on the internet can now send requests to this URL, and they will be securely tunneled to your Express app running on `localhost:8080`.

#### How Does Tunnelmole Work?

Tunnelmole works by establishing a secure, persistent connection from the client on your machine to the Tunnelmole service in the cloud. When a request hits your public URL, the service sends it down this tunnel to your local client, which then forwards it to your local server.

![How Tunnelmole works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

A key advantage of Tunnelmole is that it's **fully open source**, including the server component. This means you have the option to self-host the entire service for maximum privacy and control, which is a great option for corporate environments with strict data policies.

### Step 3: Configure the Webhook in Your GitHub Repository

Now we have all the pieces. Let's wire them together.

1.  Go to the GitHub repository you want to use for testing.
2.  Click on the **"Settings"** tab.
3.  In the left sidebar, click on **"Webhooks"**.
4.  Click the **"Add webhook"** button in the top right.

You'll see a configuration form. Fill it out as follows:

*   **Payload URL**: Paste your HTTPS Tunnelmole URL here, and append your endpoint path `/github-webhook`. For example: `https://k2e6yq-ip-12-34-56-78.tunnelmole.net/github-webhook`
*   **Content type**: Select `application/json`. Our Express app is configured to parse JSON.
*   **Secret**: This is a critical field for security. Create a strong, random string to use as a secret and paste it here. You can use a password generator for this. We'll see how to use this secret in the next section. For now, let's say our secret is `this-is-a-very-secret-string-12345`.
*   **Which events would you like to trigger this webhook?**: For this demo, select "Just the push event." In a real-world scenario, you might want to select "Send me everything" or manually choose specific events.

![github webhook setup](https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp)


Finally, click the **"Add webhook"** button at the bottom of the form.

### Step 4: Test Your GitHub Webhook!

As soon as you add the webhook, GitHub sends a special `ping` event to your Payload URL to verify that it's reachable.

Look at your running Node.js server's terminal. You should see output like this:

```
Received a webhook for the 'ping' event
---
```

This confirms that the entire chain is working: GitHub -> Tunnelmole -> Your Local Express App.

Now for the real test. Make a change in your local repository, commit it, and push it to GitHub.

```bash
# Make a change, for example, edit your README.md
echo "Testing my new webhook" >> README.md

# Commit and push
git add .
git commit -m "feat: Implement amazing new feature"
git push
```

As soon as the push is complete, check your server's terminal again. This time, you'll see the detailed output from our `push` event handler:

```
Received a webhook for the 'push' event
---
New push to repository: your-username/github-webhook-receiver
Branch: main
Author: Your Name
Commit Message: "feat: Implement amazing new feature"
```

Success! You have successfully created, configured, and tested a `github webhook` that communicates with a server on your local machine.

## Securing Your Webhook Endpoint

Our current setup works, but it has a major security flaw. The Tunnelmole URL is public. Anyone who finds it could send fake `POST` requests to our endpoint, pretending to be GitHub and potentially triggering our automation with malicious data.

This is why GitHub has the **"Secret"** field. When you set a secret, GitHub uses it to create a hash-based message authentication code (HMAC) signature for each webhook payload. This signature is sent with the request in the `X-Hub-Signature-256` header.

Your application can then compute its own signature using the same secret and compare it with the one sent by GitHub. If they match, you can be 100% certain the request is authentic and came from GitHub.

Let's update our `app.js` to implement this verification.

First, install the built-in `crypto` module (it comes with Node.js, so no `npm install` is needed). We'll also use an environment variable to store our secret securely instead of hardcoding it.

Update `app.js`:

```javascript
/*
 * app.js - A SECURE Express server to receive GitHub webhooks.
 */

const express = require('express');
const crypto = require('crypto'); // Built-in Node.js module

const app = express();
const port = 8080;

// Get the webhook secret from an environment variable for security
const GITHUB_WEBHOOK_SECRET = process.env.GITHUB_WEBHOOK_SECRET;

if (!GITHUB_WEBHOOK_SECRET) {
    console.error('Error: GITHUB_WEBHOOK_SECRET environment variable not set.');
    process.exit(1);
}

// Middleware to parse JSON bodies.
// IMPORTANT: We need the raw request body (a buffer) to verify the signature.
// The `verify` function lets us capture it.
app.use(express.json({
    verify: (req, res, buf) => {
        req.rawBody = buf;
    }
}));

// A middleware function to verify the GitHub signature
function verifyGitHubSignature(req, res, next) {
    const signatureHeader = req.headers['x-hub-signature-256'];
    if (!signatureHeader) {
        return res.status(401).send('Unauthorized: No signature provided.');
    }

    // Create our own signature using the secret and the raw request body
    const hmac = crypto.createHmac('sha256', GITHUB_WEBHOOK_SECRET);
    hmac.update(req.rawBody);
    const expectedSignature = `sha256=${hmac.digest('hex')}`;

    console.log(`Received Signature: ${signatureHeader}`);
    console.log(`Expected Signature: ${expectedSignature}`);

    // Use a timing-safe comparison to prevent timing attacks
    const isSignatureValid = crypto.timingSafeEqual(Buffer.from(signatureHeader), Buffer.from(expectedSignature));

    if (!isSignatureValid) {
        return res.status(401).send('Unauthorized: Invalid signature.');
    }

    // If the signature is valid, proceed to the next middleware/handler
    next();
}


// Apply the verification middleware ONLY to our webhook endpoint
app.post('/github-webhook', verifyGitHubSignature, (req, res) => {
    const githubEvent = req.headers['x-github-event'];
    console.log(`Received a verified webhook for the '${githubEvent}' event`);
    console.log('---');

    if (githubEvent === 'push') {
        const payload = req.body;
        const repositoryName = payload.repository.full_name;
        const commitAuthor = payload.head_commit.author.name;
        const commitMessage = payload.head_commit.message;
        const branch = payload.ref.split('/').pop();

        console.log(`New push to repository: ${repositoryName}`);
        console.log(`Branch: ${branch}`);
        console.log(`Author: ${commitAuthor}`);
        console.log(`Commit Message: "${commitMessage}"`);
    }

    res.status(200).send('Webhook received and verified.');
});

app.get('/', (req, res) => {
    res.send('Server is up and running. Ready to receive GitHub webhooks at /github-webhook.');
});

app.listen(port, () => {
    console.log(`Server started on http://localhost:${port}`);
});
```
Key changes:
1.  We read the secret from `process.env.GITHUB_WEBHOOK_SECRET`.
2.  We created a middleware function `verifyGitHubSignature`.
3.  Inside the middleware, we compute our own HMAC SHA256 signature from the raw request body buffer (`req.rawBody`) that we cleverly saved earlier.
4.  We use `crypto.timingSafeEqual` to compare our signature with the one from the header. This is important to prevent timing attacks.
5.  If the signatures don't match, we immediately send a `401 Unauthorized` and stop processing.
6.  We apply this middleware specifically to our `/github-webhook` route.

Now, restart your server with the secret set as an environment variable. **Replace the secret with the one you configured in the GitHub UI.**

```bash
# On Linux/macOS
GITHUB_WEBHOOK_SECRET="this-is-a-very-secret-string-12345" node app.js

# On Windows (Command Prompt)
set GITHUB_WEBHOOK_SECRET="this-is-a-very-secret-string-12345"
node app.js
```

To test this, go back to your webhook settings in GitHub, click "Edit", and go to the "Recent Deliveries" tab at the bottom. Click the three dots next to the last delivery and select "Redeliver." GitHub will send the same payload again. This time, your secure server will verify the signature before processing it.

## Conclusion

The **GitHub webhook** is a gateway to powerful automation. By understanding how to create, secure, and test webhook endpoints, you can build seamless CI/CD pipelines, integrate with project management tools, create custom notifications, and much more.

In this guide, we've walked through the entire process:
1.  We learned the fundamentals of webhooks.
2.  We built a Node.js and Express server to act as a webhook receiver.
3.  We used the open-source tool **Tunnelmole** to expose our local server to the internet for easy and fast testing.
4.  We configured a `github webhook` in our repository to send `push` events to our server.
5.  Most importantly, we secured our endpoint by verifying the HMAC signature, ensuring that we only process legitimate requests from GitHub.

The combination of a flexible local server and a secure tunneling tool like Tunnelmole provides the perfect development environment for building and debugging any webhook integration. You can now apply these principles to any webhook provider, not just GitHub, to build robust, event-driven applications.
