---
title: "How to Handle Slack Webhooks and Events Locally with Tunnelmole and Express.js"
description: "Step-by-step guide to receiving and testing Slack webhooks and Events API notifications on your local Express.js app using Tunnelmole, the open source tunneling tool."
tags: ["slack", "webhook", "eventsapi", "expressjs", "tunnelmole", "opensource", "nodejs", "integration", "automation"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/03/slack-webhook/"
slug: "slack-webhook"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Handle Slack Webhooks and Events Locally with Tunnelmole and Express.js

**Learn how to receive and debug Slack webhooks and Events API notifications on your local Express.js app using Tunnelmole, the open source tunneling tool. Step-by-step guide for developers and automation pros.**

---

## Introduction

Slack is the backbone of modern team communication, and its webhooks and Events API make it easy to automate workflows, build bots, and react to activity in real time. 

## What are Slack Webhooks and Events?

Slack offers two main ways to send data to your app:

- **Incoming Webhooks:** Let you send messages _to_ Slack channels from your app.
- **Outgoing Webhooks (Legacy):** Let Slack send data _to your app_ when certain keywords are mentioned (not recommended for new projects).
- **Events API:** The modern way to receive notifications when things happen in Slack (e.g., a user uploads a file, a message is posted, a channel is created).

This guide focuses on **receiving** data from Slack—especially via the Events API, which is the recommended approach for most use cases.

**Common use cases:**

- Build bots that react to messages or user actions
- Automate workflows when files are uploaded or channels are created
- Integrate Slack with your internal tools or databases

When an event occurs, Slack sends a POST request (with JSON) to your specified public URL.

---

## Why You Need a Public URL for Slack Webhooks and Events

Slack’s servers must be able to reach your webhook endpoint over the public internet. If your Express.js app is running on `localhost:3000`, Slack can’t access it directly.

**You need a public HTTPS URL that forwards requests to your local server.**

**Typical workflow:**

1. Start your Express.js app on `localhost:3000`
2. Use Tunnelmole to get a public URL (e.g., `https://abc123.tunnelmole.net`)
3. Register this URL in your Slack app’s configuration
4. Slack sends webhooks/events to your Tunnelmole URL, which forwards them to your local app

---

## Why Use Tunnelmole for Slack Webhooks?

Tunnelmole is a modern, open source tunneling tool that makes local webhook development fast and easy:

- 100% open source (MIT/AGPLv3)
- Free public HTTPS URLs (no login required)
- Self-hostable for privacy and custom domains
- Native Node.js app, easy one-line install
- Works on Mac, Linux, Windows

---

## Step 1: Create a Local Express.js Webhook Receiver

If you already have an endpoint to receive Slack events, you can skip this section.

Let’s build a simple Express.js app that listens for POST requests from Slack.

**Install dependencies:**

```bash
npm init -y
npm install express body-parser
```

**Create [`slack-webhook-server.js`](slack-webhook-server.js):**

```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = 3000;

// Parse JSON
app.use(bodyParser.json());

// Slack Events endpoint (e.g., /slack/events)
app.post('/slack/events', (req, res) => {
  const { type, challenge, event } = req.body;

  // URL Verification challenge (required for Events API setup)
  if (type === 'url_verification') {
    return res.status(200).send(challenge);
  }

  // Handle actual events
  if (type === 'event_callback') {
    console.log('Received Slack event:', event);
    // Add your business logic here (e.g., react to file_created, message, etc.)
    res.status(200).send('Event received');
  } else {
    res.status(200).send('OK');
  }
});

// (Optional) Outgoing webhook endpoint (legacy)
app.post('/slack/webhook', (req, res) => {
  console.log('Received Slack outgoing webhook:', req.body);
  res.status(200).send('Webhook received');
});

app.listen(PORT, () => {
  console.log(`Express Slack webhook server listening at http://localhost:${PORT}`);
});
```

**Start your server:**

```bash
node slack-webhook-server.js
```

You should see:

```
Express Slack webhook server listening at http://localhost:3000
```

---

## Step 2: Install and Run Tunnelmole

Tunnelmole gives your local server a public HTTPS URL in seconds.

**Install Tunnelmole (Linux/Mac/WSL):**

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

**Or install with npm (requires Node.js 16.10+):**

```bash
sudo npm install -g tunnelmole
```

**Start a tunnel to your Express app:**

```bash
tmole 3000
```

**Example output:**

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://abc123.tunnelmole.net ⟶ http://localhost:3000
http://abc123.tunnelmole.net ⟶ http://localhost:3000
```

**Copy the HTTPS URL** (e.g., `https://abc123.tunnelmole.net`)—you’ll use this as your Slack Events/Webhook endpoint.

---

## Step 3: Create and Configure a Slack App

To receive events from Slack, you need to create a Slack app and subscribe to events.

### 1. Create a New Slack App

- Go to [Slack API: Your Apps](https://api.slack.com/apps) and click **Create New App**.
- Choose **From scratch** and give your app a name and workspace.

### 2. Enable Events API

- In your app settings, go to **Event Subscriptions**.
- Toggle **Enable Events** to "On".
- In **Request URL**, enter your Tunnelmole HTTPS URL + endpoint (e.g., `https://abc123.tunnelmole.net/slack/events`).

#### Slack will immediately send a `url_verification` challenge to your endpoint. Your Express handler (see above) must respond with the `challenge` value to verify the URL.

### 3. Subscribe to Bot/User Events

- Under **Subscribe to bot events** (or **user events**), click **Add Bot User Event**.
- Add events you want to receive, e.g.:
  - `file_created` (when a user creates a file)
  - `message.channels` (when a message is posted in a channel)
  - `reaction_added`, etc.

### 4. Install the App to Your Workspace

- Go to **OAuth & Permissions**.
- Click **Install App to Workspace** and authorize.

---

## Step 4: Test Slack Events Locally

**Trigger an event in Slack:**

- For `file_created`: Upload a file in any channel.
- For `message.channels`: Post a message in a channel.

**Check your Express app’s terminal:**

You should see the event payload logged:

```
Received Slack event: {
  type: 'file_created',
  file_id: 'F12345678',
  user_id: 'U12345678',
  ...
}
```

or

```
Received Slack event: {
  type: 'message',
  text: 'Hello from Slack!',
  ...
}
```

**Debugging tips:**

- If you don’t see the event, check that Tunnelmole is running and the URL matches your Express endpoint.
- Check the Slack app’s Event Subscriptions page for error messages.
- Use `console.log` or a debugger to inspect the request payload.

---

## Step 5: (Optional) Handle Outgoing Webhooks (Legacy)

If you’re maintaining an older Slack integration that uses outgoing webhooks:

- In Slack, configure the outgoing webhook to point to your Tunnelmole URL + `/slack/webhook`.
- Your Express handler will log the payload.

---

## Security: Validating Slack Requests

Slack signs every request with a signature in the `X-Slack-Signature` header and a timestamp in `X-Slack-Request-Timestamp`. You should verify these to ensure requests are genuinely from Slack.

**Example:**

```js
const crypto = require('crypto');

function verifySlackRequest(req, signingSecret) {
  const timestamp = req.headers['x-slack-request-timestamp'];
  const sig = req.headers['x-slack-signature'];
  const body = JSON.stringify(req.body);

  // Prevent replay attacks
  if (Math.abs(Date.now() / 1000 - timestamp) > 60 * 5) {
    return false;
  }

  const basestring = `v0:${timestamp}:${body}`;
  const mySig = 'v0=' + crypto.createHmac('sha256', signingSecret)
    .update(basestring)
    .digest('hex');

  return crypto.timingSafeEqual(Buffer.from(mySig), Buffer.from(sig));
}

// In your handler:
app.post('/slack/events', (req, res) => {
  if (!verifySlackRequest(req, process.env.SLACK_SIGNING_SECRET)) {
    return res.status(401).send('Unauthorized');
  }
  // ... handle event as before
});
```

**Note:** Never expose secrets in public repos or logs. Store your `SLACK_SIGNING_SECRET` in environment variables.

---

## Troubleshooting Common Issues

**1. Slack says “Your Request URL responded with an error”**

- Double-check your Tunnelmole URL and endpoint path
- Ensure your Express app is running and listening on the correct port
- Make sure Tunnelmole is running and hasn’t timed out

**2. URL Verification fails**

- Ensure your handler responds with the `challenge` value from Slack’s request
- Check for typos in the endpoint path (e.g., `/slack/events`)

**3. Event handler not triggered**

- Confirm the app is installed in your workspace
- Check the app’s permissions and event subscriptions
- Look at Slack’s error details for more info

**4. Signature validation fails**

- Ensure your signing secret matches the one in your Slack app settings
- Check for header name typos or case sensitivity

---

## Summary and Next Steps

With Tunnelmole, you can develop and debug Slack webhooks and Events API integrations on your local machine—no cloud deployment, no firewall headaches, and no closed-source black boxes.

**Next steps:**
- Add more event handlers for different Slack events
- Integrate webhook handling into your app’s business logic
- Use Tunnelmole’s self-hosting for custom domains or privacy
- Share your public URL with collaborators for live demos

**Ready to build better Slack integrations?**  
[Get started with Tunnelmole](https://tunnelmole.com/?utm_source=slack-webhook-article)

---

## FAQ: Slack Webhooks, Events API, and Tunnelmole

### How do I test Slack webhooks and events locally?

Use Tunnelmole to expose your local Express.js server to the internet. Register the Tunnelmole HTTPS URL as your webhook/events endpoint in your Slack app.

### Is Tunnelmole secure for handling Slack events?

Tunnelmole uses HTTPS for all public URLs. For production, always validate incoming requests (using Slack’s signature verification) and consider self-hosting for maximum control.

### Can I use Tunnelmole for other webhook providers?

Yes! Tunnelmole works with any service that needs to reach your local server—Stripe, GitHub, Shopify, Zapier, IFTTT, and more.

### What happens if I restart Tunnelmole?

You’ll get a new public URL unless you use a custom domain (paid/self-hosted). Update your Slack app’s endpoint if the URL changes.

---

## References

- [Slack Events API Documentation](https://api.slack.com/apis/connections/events-api)
- [Slack Outgoing Webhooks (Legacy)](https://api.slack.com/legacy/outgoing-webhooks)
- [Tunnelmole GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Website](https://tunnelmole.com/?utm_source=slack-webhook-article)
- [Express.js Documentation](https://expressjs.com/)

---

**Build, test, and debug Slack webhooks and events locally—faster and easier with Tunnelmole.**