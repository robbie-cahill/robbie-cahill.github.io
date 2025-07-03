---
title: "How to Handle HubSpot Webhooks Locally with Tunnelmole and Express.js"
description: "Step-by-step guide to receiving and testing HubSpot webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool."
tags: ["hubspot", "webhook", "expressjs", "tunnelmole", "opensource", "nodejs", "crm", "integration", "automation"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/03/how-to-handle-hubspot-webhooks-locally-with-tunnelmole-and-express-js/"
slug: "hubspot-webhook"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Handle HubSpot Webhooks Locally with Tunnelmole and Express.js

**Learn how to receive and debug HubSpot webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool. Step-by-step guide for developers, CRM integrators, and automation pros.**

---

## Introduction

HubSpot is a leading CRM and marketing automation platform, powering everything from lead capture to sales pipelines. If you’re building or testing a HubSpot integration that uses webhooks, you’ll quickly hit a familiar roadblock: HubSpot requires a public HTTPS endpoint, but your Express.js app is running on `localhost`.

**Tunnelmole** solves this instantly by giving your local server a secure, public URL. In this guide, you’ll learn how to:

- Set up a local Express.js app to receive HubSpot webhooks
- Use Tunnelmole to expose your local server to the internet
- Register a webhook subscription in HubSpot to hit your local endpoint
- Debug and verify webhook delivery end-to-end

Whether you’re building custom CRM automations, syncing data, or learning about webhooks, this tutorial will get you up and running—no cloud deployment or firewall changes required.

---

## What are HubSpot Webhooks?

HubSpot webhooks let you receive real-time notifications when events happen in your CRM—like contact creation, property changes, deal updates, and more. You can use webhooks to:

- Sync HubSpot data to your own systems
- Trigger business logic in response to CRM events
- Integrate HubSpot with other tools and APIs

**How it works:**  
When a subscribed event occurs (e.g., a contact is created), HubSpot sends a POST request to your webhook URL with event details in JSON format.

---

## Why You Need a Public URL for HubSpot Webhooks

HubSpot’s servers must be able to reach your webhook endpoint over the public internet. If your Express.js app is running on `localhost:3000`, HubSpot can’t access it directly.

**You need a public HTTPS URL that forwards requests to your local server.**

**Typical workflow:**

1. Start your Express.js app on `localhost:3000`
2. Use Tunnelmole to get a public URL (e.g., `https://abc123.tunnelmole.net`)
3. Register this URL as your webhook endpoint in HubSpot
4. HubSpot sends webhooks to your Tunnelmole URL, which forwards them to your local app

---

## Why Use Tunnelmole for HubSpot Webhooks?

Tunnelmole is a modern, open source tunneling tool that makes local webhook development fast and easy:

- 100% open source (MIT/AGPLv3)
- Free public HTTPS URLs (no login required)
- Self-hostable for privacy and custom domains
- Native Node.js app, easy one-line install
- Works on Mac, Linux, Windows

---

## Step 1: Create a Local Express.js Webhook Receiver

If you already have an endpoint to receive HubSpot webhooks, you can skip this section.

Let’s build a simple Express.js app that listens for POST requests from HubSpot.

**Install dependencies:**

```bash
npm init -y
npm install express body-parser
```

**Create [`hubspot-webhook-server.js`](hubspot-webhook-server.js):**

```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = 3000;

// Parse JSON and form data
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Webhook endpoint (e.g., /webhook/hubspot)
app.post('/webhook/hubspot', (req, res) => {
  console.log('Received HubSpot webhook:', req.body);
  // You can add validation or business logic here
  res.status(200).send('Webhook received');
});

app.listen(PORT, () => {
  console.log(`Express webhook server listening at http://localhost:${PORT}`);
});
```

**Start your server:**

```bash
node hubspot-webhook-server.js
```

You should see:

```
Express webhook server listening at http://localhost:3000
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

**Copy the HTTPS URL** (e.g., `https://abc123.tunnelmole.net`)—you’ll use this as your HubSpot webhook endpoint.

---

## Step 3: Register a Webhook Subscription in HubSpot

Now, let’s set up a webhook subscription in HubSpot to send events to your local server (via Tunnelmole).

### 1. Create a HubSpot App (for Webhooks API)

- Go to your [HubSpot Developer Account](https://developers.hubspot.com/)
- Click **Create app**
- Fill in the app details (name, description, etc.)

### 2. Add a Webhook Subscription

- In your app, go to **Webhooks** in the left sidebar
- Click **Create subscription**
- **Event type:** Choose an event (e.g., Contact creation)
- **Webhook URL:** Paste your Tunnelmole HTTPS URL + endpoint (e.g., `https://abc123.tunnelmole.net/webhook/hubspot`)
- **HTTP method:** POST
- **Version:** v3 (recommended)

### 3. Save and Authorize the App

- Save your webhook subscription
- Install the app to your HubSpot account (follow the OAuth flow)
- HubSpot will send a test webhook to your endpoint to verify it

**Note:**  
If you’re using HubSpot’s legacy workflows or Marketing Hub Professional/Enterprise, you can also configure webhooks directly in workflows. The process is similar—just use your Tunnelmole URL as the webhook destination.

---

## Step 4: Test and Debug HubSpot Webhooks Locally

**Check your Express app’s terminal:**

You should see the webhook payload logged:

```
Received HubSpot webhook: [ { ...event data... } ]
```

**Debugging tips:**

- If you don’t see the webhook, check that Tunnelmole is running and the URL matches your Express endpoint
- Check HubSpot’s webhook logs for delivery status and errors
- Use `console.log` or a debugger to inspect the request payload

---

## Security: Validating HubSpot Webhook Requests

HubSpot webhooks include a signature in the `X-HubSpot-Signature` header (for apps using v3 webhooks). You should verify this signature to ensure the request is from HubSpot.

**Example: Validate the webhook signature**

1. In your HubSpot app, get your client secret.
2. In your Express handler, compute the HMAC SHA-256 hash of the request body using your client secret.
3. Compare it to the `X-HubSpot-Signature` header.

**Sample code:**

```js
const crypto = require('crypto');

app.post('/webhook/hubspot', (req, res) => {
  const signature = req.get('X-HubSpot-Signature');
  const clientSecret = 'YOUR_CLIENT_SECRET'; // Replace with your app's client secret
  const payload = JSON.stringify(req.body);

  const hash = crypto
    .createHmac('sha256', clientSecret)
    .update(payload)
    .digest('hex');

  if (signature !== hash) {
    return res.status(401).send('Unauthorized');
  }

  console.log('Received verified HubSpot webhook:', req.body);
  res.status(200).send('Webhook received');
});
```

**Note:**  
- Never expose secrets in public repos or logs.
- See [HubSpot’s webhook security docs](https://developers.hubspot.com/docs/api/webhooks/webhooks-overview#securing-your-webhooks) for details.

---

## Troubleshooting Common Issues

**1. HubSpot says “Webhook failed”**

- Double-check your Tunnelmole URL and endpoint path
- Ensure your Express app is running and listening on the correct port
- Make sure Tunnelmole is running and hasn’t timed out

**2. Webhook handler not triggered**

- Confirm the webhook subscription is active and the app is installed
- Check for typos in the endpoint path (e.g., `/webhook/hubspot`)
- Look at HubSpot’s webhook logs for error details

**3. Signature validation fails**

- Ensure your client secret matches the one in your HubSpot app
- Check for differences in how the payload is serialized (use `JSON.stringify`)
- Verify the header name and case sensitivity

---

## Summary and Next Steps

With Tunnelmole, you can develop and debug HubSpot webhooks on your local machine—no cloud deployment, no firewall headaches, and no closed-source black boxes.

**Next steps:**
- Add more webhook endpoints for different HubSpot events
- Integrate webhook handling into your app’s business logic
- Use Tunnelmole’s self-hosting for custom domains or privacy
- Share your public URL with collaborators for live demos

**Ready to build smarter CRM automations?**  
[Get started with Tunnelmole](https://tunnelmole.com/?utm_source=hubspot-webhook-article)

---

## FAQ: HubSpot Webhooks and Tunnelmole

### How do I test HubSpot webhooks locally?

Use Tunnelmole to expose your local Express.js server to the internet. Register the Tunnelmole HTTPS URL as your webhook endpoint in HubSpot.

### Is Tunnelmole secure for handling webhooks?

Tunnelmole uses HTTPS for all public URLs. For production, always validate incoming requests (e.g., with the HubSpot signature) and consider self-hosting for maximum control.

### Can I use Tunnelmole for other webhook providers?

Yes! Tunnelmole works with any service that needs to reach your local server—Stripe, GitHub, Shopify, Zapier, IFTTT, and more.

### What happens if I restart Tunnelmole?

You’ll get a new public URL unless you use a custom domain (paid/self-hosted). Update your HubSpot webhook endpoint if the URL changes.

---

## References

- [HubSpot Webhooks API Documentation](https://developers.hubspot.com/docs/api/webhooks)
- [Tunnelmole GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Website](https://tunnelmole.com/?utm_source=hubspot-webhook-article)
- [Express.js Documentation](https://expressjs.com/)

---

**Build, test, and debug HubSpot webhooks locally—faster and easier with Tunnelmole.**