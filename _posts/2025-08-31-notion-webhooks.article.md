---
title: 'Notion Webhooks: A Complete Guide for Developers (2025)'
description: >-
  Learn how to build and test Notion webhooks. This guide covers creating a
  Node.js endpoint, getting a public URL with Tunnelmole, and processing Notion
  data.
tags:
  - notion webhooks
  - notion
  - webhooks
  - api
  - automation
  - nodejs
  - expressjs
  - tunnelmole
  - opensource
  - javascript
  - typescript
  - python
  - programming
  - coding
canonical_url: 'https://softwareengineeringstandard.com/2025/08/31/notion-webhooks/'
slug: notion-webhooks
cover_image: >-
  https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp
claire_approved: false
human_approved: true
published: true
---

Notion has become a powerhouse for personal and collaborative productivity, but its true potential is unlocked when you start automating it. For developers, this often means working with webhooks. While Notion doesn't have traditional, one-click outgoing webhooks like GitHub or Slack, its powerful API allows you to create robust, webhook-like integrations.

This guide will walk you through the entire process of setting up a "Notion webhook" system. You'll learn how to build a listener service with Node.js and Express, expose it to the internet for testing and development with the open source Tunnelmole tunneling tool, and create a workflow that effectively gives you real-time notifications for changes in your Notion database.

By the end of this tutorial, you'll have a fully functional webhook endpoint that can receive and process data triggered by events in your Notion workspace.

### What are Webhooks?

Before diving into the specifics of Notion, let's quickly recap what a webhook is.

In a typical API interaction, your application (the client) sends a request to a server, and the server sends a response. You are initiating the communication.

Webhooks flip this model on its head. They are automated, server-to-server messages sent when a specific event occurs. Instead of your application polling for changes, the service (the "webhook provider") sends a POST request to a public URL you provide—your "webhook listener." This is often called a "reverse API" because it's the server that initiates the request.

This event-driven approach is far more efficient than constant polling, saving resources and providing near real-time data.

### How "Notion Webhooks" Work

As of 2025, Notion's API does not support native outgoing webhooks. You can't simply go into a Notion database, click "Add Webhook," and paste your URL.

So, how do we create a "Notion webhook"? We use a simple, powerful pattern:

1.  **Polling Service:** An intermediary service monitors your Notion database for changes. This can be a third-party automation platform like Zapier or Make, or a custom script running on a schedule (e.g., a cron job).
2.  **Webhook Trigger:** When this service detects a change (like a new page being added or a property being updated), it triggers an action.
3.  **HTTP Request:** That action is to send an HTTP POST request to a public URL that you control. This is your webhook endpoint.
4.  **Local Webhook Handler:** Your local application receives this request and processes the payload, which contains the data from the Notion event.

To make this work during development, your local server (running on `localhost`) needs a public URL. This is where a tunneling tool becomes essential.

### Prerequisites

To follow this guide, you will need:

*   A **Notion Account** and a workspace.
*   **Node.js** (v16.10 or later) and **npm** installed on your machine.
*   Basic understanding of **JavaScript** and the **Express.js** framework.
*   A code editor like **VS Code**.

### Step 1: Set Up Your Node.js Webhook Handler

First, let's build a simple Express.js application that will act as our webhook listener. This server will have one job: to listen for incoming POST requests on a specific endpoint.

Create a new directory for your project and initialize it with npm.

```bash
mkdir notion-webhook-handler
cd notion-webhook-handler
npm init -y
```

Next, install Express and `body-parser`, which helps parse the incoming request body.

```bash
npm install express body-parser
```

Now, create a file named `index.js` and add the following code:

```javascript
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

// Use body-parser middleware to parse JSON request bodies
app.use(bodyParser.json());

// Define the webhook endpoint
app.post('/notion-webhook', (req, res) => {
    console.log('🎉 Webhook received!');
    
    // The data from Notion (sent by the intermediary) will be in the request body
    const { body } = req;
    console.log('Payload:');
    console.log(JSON.stringify(body, null, 2));

    // Acknowledge receipt of the webhook
    res.status(200).send('Webhook received successfully.');
});

// A root endpoint to confirm the server is running
app.get('/', (req, res) => {
    res.send('Notion Webhook Handler is running!');
});

app.listen(port, () => {
    console.log(`Server listening at http://localhost:${port}`);
});
```

This code sets up a simple web server with two routes:
*   `GET /`: A simple health check to confirm the server is running.
*   `POST /notion-webhook`: This is our main webhook endpoint. It logs the received payload to the console and sends a `200 OK` response.

Run the server to make sure everything is working:

```bash
node index.js
```

You should see `Server listening at http://localhost:3000` in your terminal. You can open `http://localhost:3000` in your browser to see the "Webhook Handler is running!" message.

### Step 2: Get a Public URL with Tunnelmole

Your Express server is running on `localhost`, which is only accessible from your own computer. For an external service to send it a webhook, you need a publicly accessible HTTPS URL. This is where Tunnelmole comes in.

Tunnelmole is an open-source tool that creates a secure tunnel between a public URL and your local development environment. It's incredibly simple to use and you can be up and running in seconds.

#### Install Tunnelmole

You can install Tunnelmole using `npm` if you have Node.js installed:

```bash
sudo npm install -g tunnelmole
```

Alternatively, for Linux, Mac, or WSL, you can use the following `curl` command:

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

#### Run Tunnelmole

With your Node.js server still running, open a **new terminal window** and run the following command to tunnel port 3000:

```bash
tmole 3000
```

Tunnelmole will start and display a public URL that now points to your local server.

```
$ tmole 3000
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://k8sjer-ip-12-34-56-78.tunnelmole.net ⟶ http://localhost:3000
http://k8sjer-ip-12-34-56-78.tunnelmole.net ⟶ http://localhost:3000
```

Copy the `https://` URL. This is your public webhook URL. Anyone on the internet can now send requests to this URL, and they will be forwarded securely to your Express app running on `localhost:3000`.

Keep Tunnelmole running in this terminal window.

#### How Tunnelmole Works

The magic of Tunnelmole lies in its simplicity. It establishes a persistent WebSocket connection from your machine to the Tunnelmole cloud service. When a request hits your public URL, the service relays it through this tunnel to your local machine, which then forwards it to your application.

![How Tunnelmole works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

A key benefit of Tunnelmole is that it's **open source**. Both the client and the server-side code are available for review. For ultimate control and privacy, you can even **self-host** the Tunnelmole service on your own server.

### Step 3: Create a Notion Integration

To allow an application or service to access your Notion workspace, you need to create an "integration."

1.  **Go to the Integrations Page:** Navigate to [www.notion.so/my-integrations](https://www.notion.so/my-integrations).
2.  **Create a New Integration:** Click the `+ New integration` button.
3.  **Fill in the Details:**
    *   **Name:** Give it a descriptive name, like "My Database Webhook".
    *   **Associated workspace:** Select your workspace.
    *   **Capabilities:** For this example, "Read content" is sufficient.
    *   Click **Submit**.
4.  **Copy the Secret Token:** On the next screen, you'll see your "Internal Integration Token." Click "Show" and copy this token. Treat it like a password; don't expose it in public code repositories.

### Step 4: Create and Share a Notion Database

Now, let's create the Notion database that our integration will monitor.

1.  **Create a new page** in your Notion workspace.
2.  On the new page, select `/table` and choose **Table - Full page**.
3.  Give your database a name, like "My Webhook-Enabled Tasks."
4.  **Share the database with your integration:**
    *   Click the **•••** menu in the top-right corner of the database page.
    *   Click `+ Add connections`.
    *   Search for the integration you just created (e.g., "My Database Webhook") and select it.
    *   Confirm the connection.

### Step 5: Configure an Automation to Trigger the Webhook

As mentioned, we need an intermediary to watch for changes and call our webhook. For this tutorial, we'll use Zapier as an example, as it has a free tier that is perfect for this. The same principle applies to other services like Make (formerly Integromat) or Pipedream.

1.  **Sign up or log in** to [Zapier](https://zapier.com/).
2.  Click **Create Zap**.
3.  **Set up the Trigger:**
    *   Search for and select **Notion**.
    *   For the **Event**, choose **New Database Item**.
    *   Connect your Notion account, authorizing Zapier to use the integration you created.
    *   Select the **Database** you just created ("My Webhook-Enabled Tasks").
    *   Click **Test trigger**. Zapier will try to find a recent item in your database. Go ahead and add a new row to your Notion table so Zapier has something to find.

4.  **Set up the Action:**
    *   After the trigger test is successful, add a new action step.
    *   Search for and select **Webhooks by Zapier**.
    *   For the **Event**, choose **POST**.
    *   Now, configure the request:
        *   **URL:** Paste your public Tunnelmole URL here, followed by the endpoint path. For example: `https://k8sjer-ip-12-34-56-78.tunnelmole.net/notion-webhook`.
        *   **Payload Type:** Set this to `Json`.
        *   **Data:** This is where you map the data from Notion. You can create a JSON object and pull in fields from the Notion database item. For example:
            *   `name`: Map this to the `Name` property from Notion.
            *   `status`: Map this to the `Status` property.
            *   `url`: Map this to the `URL` of the Notion page.
        *   **Headers:** Leave this blank for now, but you could add an authentication token here for security.

5.  **Test and Publish:**
    *   Click **Test step**. Zapier will send a real POST request to your Tunnelmole URL.
    *   Look at your Node.js server's terminal. You should see the "🎉 Webhook received!" message and the full JSON payload from Zapier printed to the console!

    ```json
    {
      "name": "My First Task",
      "status": "Not started",
      "url": "https://www.notion.so/..."
    }
    ```
    * If everything looks good, **Publish** your Zap.

Congratulations! You now have a live webhook integration. Every time you add a new item to your Notion database, Zapier will detect it and send its data to your local server via Tunnelmole.

### Step 6: Secure Your Webhook Endpoint

Right now, your endpoint is public. Anyone who knows the URL can send data to it. In a production environment, you must verify that incoming requests are legitimate. A simple way to do this is with a shared secret token.

1.  **In Zapier:** In the **Headers** section of your Webhooks action, add a new header.
    *   **Key:** `X-Secret-Token`
    *   **Value:** A long, random string. You can generate one easily.

2.  **In your Node.js app:** Create a middleware function to check for this header before your main handler runs.

Update your `index.js`:

```javascript
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

// THIS IS YOUR SECRET. Store it securely, e.g., in an environment variable.
const SHARED_SECRET = 'your-super-secret-random-string';

app.use(bodyParser.json());

// Middleware to verify the secret token
const verifySecret = (req, res, next) => {
    const receivedSecret = req.get('X-Secret-Token');
    if (receivedSecret && receivedSecret === SHARED_SECRET) {
        // Secrets match, proceed to the handler
        next();
    } else {
        console.warn('⚠️ Unauthorized request: Invalid or missing secret token.');
        res.status(401).send('Unauthorized');
    }
};

// Apply the middleware ONLY to your webhook endpoint
app.post('/notion-webhook', verifySecret, (req, res) => {
    console.log('🎉 Webhook received (and authenticated)!');
    
    const { body } = req;
    console.log('Payload:');
    console.log(JSON.stringify(body, null, 2));

    res.status(200).send('Webhook received successfully.');
});

app.get('/', (req, res) => {
    res.send('Notion Webhook Handler is running!');
});

app.listen(port, () => {
    console.log(`Server listening at http://localhost:${port}`);
});
```

Restart your Node server (`Ctrl+C` then `node index.js`). Now, only requests from Zapier that include the correct secret token will be processed.

### Conclusion and Next Steps

You've successfully built a complete, secure webhook integration for Notion from scratch. You learned how to:
*   Create a webhook listener with Node.js and Express.
*   Expose your local server to the internet using the open-source tool, **Tunnelmole**.
*   Configure a Notion integration and connect it to a database.
*   Use an automation service like Zapier to poll for changes and trigger your webhook.
*   Secure your endpoint with a shared secret.

This foundation opens up a world of automation possibilities. You could extend this application to:
*   Send a Slack message when a task is marked "Done."
*   Create a calendar event from a new entry in a Notion "Meetings" database.
*   Sync contacts between Notion and a CRM.
*   Log changes to an external audit file.

Happy automating!
