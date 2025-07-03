---
title: "How to Handle Shopify Webhooks with Tunnelmole and Express.js"
description: "Learn how to receive Shopify webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool. Step-by-step guide for developers."
tags: ["shopify", "webhook", "expressjs", "tunnelmole", "opensource", "nodejs"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/03/how-to-handle-shopify-webhooks-with-tunnelmole-and-express-js/"
slug: "shopify-webhook"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Handle Shopify Webhooks with Tunnelmole and Express.js

**Learn how to receive Shopify webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool. Step-by-step guide for developers.**

---

## Introduction

Shopify webhooks are essential for building real-time integrations, automations, and apps on the Shopify platform. But testing webhooks locally can be a pain—Shopify requires a publicly accessible HTTPS endpoint, while your Express.js app is running on `localhost`. 

**Tunnelmole** solves this problem by giving your local server a secure, public URL in seconds. In this guide, you'll learn how to:

- Set up a local Express.js app to receive Shopify webhooks
- Use Tunnelmole to expose your local server to the internet
- Register a webhook with Shopify (using the admin dashboard or API)
- Debug and verify webhook delivery end-to-end

Whether you're building a custom Shopify app, automating order processing, or just learning about webhooks, this tutorial will get you up and running—no need to deploy to the cloud or mess with firewalls.

---

## Table of Contents

- [What are Shopify Webhooks?](#what-are-shopify-webhooks)
- [Why You Need a Public URL for Webhooks](#why-you-need-a-public-url-for-webhooks)
- [Why Use Tunnelmole for Shopify Webhooks?](#why-use-tunnelmole-for-shopify-webhooks)
- [Step 1: Create a Local Express.js Webhook Receiver](#step-1-create-a-local-expressjs-webhook-receiver)
- [Step 2: Install and Run Tunnelmole](#step-2-install-and-run-tunnelmole)
- [Step 3: Register Your Webhook with Shopify](#step-3-register-your-webhook-with-shopify)
- [Step 4: Test and Debug Shopify Webhooks Locally](#step-4-test-and-debug-shopify-webhooks-locally)
- [Security: Verifying Shopify Webhook Signatures](#security-verifying-shopify-webhook-signatures)
- [Troubleshooting Common Issues](#troubleshooting-common-issues)
- [Feature Comparison: Tunnelmole Features](#feature-comparison-tunnelmole-features)
- [Summary and Next Steps](#summary-and-next-steps)
- [FAQ: Shopify Webhooks and Tunnelmole](#faq-shopify-webhooks-and-tunnelmole)

---

## What are Shopify Webhooks?

Shopify webhooks are HTTP callbacks that notify your app when specific events happen in a store—like a new order, product update, or customer creation. Instead of polling the Shopify API, you can react instantly to changes.

**Common Shopify webhook events:**

- `orders/create` — New order placed
- `products/update` — Product updated
- `customers/create` — New customer registered
- `app/uninstalled` — App removed from store

When an event occurs, Shopify sends a POST request with a JSON payload to your specified webhook URL.

---

## Why You Need a Public URL for Webhooks

Shopify must be able to reach your webhook endpoint over the public internet. If your Express.js app is running on `localhost:3000`, Shopify can't access it directly.

**You need a public HTTPS URL that forwards requests to your local server.**

**Typical workflow:**

1. Start your Express.js app on `localhost:3000`
2. Use Tunnelmole to get a public URL (e.g., `https://abc123.tunnelmole.net`)
3. Register this URL as your webhook endpoint in Shopify
4. Shopify sends webhooks to your Tunnelmole URL, which forwards them to your local app

---

## Why Use Tunnelmole for Shopify Webhooks?

Tunnelmole is a modern, open source tunneling tool that makes local webhook development fast and easy:

- 100% open source (MIT/AGPLv3)
- Free public HTTPS URLs (no login required)
- Self-hostable for privacy and custom domains
- Native Node.js app, easy one-line install
- Works on Mac, Linux, Windows

**Feature Comparison:**

| Feature                | Tunnelmole         |
|------------------------|--------------------|
| Open source            | ✅ Yes             |
| Free public URLs       | ✅ Yes             |
| Custom domains         | ✅ Paid/Self-host  |
| Self-hostable          | ✅ Yes             |
| Native Node.js         | ✅ Yes             |
| One-line install       | ✅ Yes             |
| Unlimited tunnels      | ❌ No              |
| Bandwidth limits       | ❌ Yes (fair use)  |

---

## Step 1: Create a Local Express.js Webhook Receiver

If you already have an endpoint to send the shopify webhooks to, you can skip this section.

Let's start by building a simple Express.js app that listens for Shopify webhooks.

**Install dependencies:**

```bash
npm init -y
npm install express body-parser
```

**Create [`webhook-server.js`](webhook-server.js):**

```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = 3000;

// Shopify sends JSON (sometimes with HMAC signature)
app.use(bodyParser.json({
  type: 'application/json'
}));

// Webhook endpoint (e.g., /webhook/orders-create)
app.post('/webhook/orders-create', (req, res) => {
  console.log('Received Shopify webhook:', req.body);
  // TODO: Verify HMAC signature for security (see below)
  res.status(200).send('Webhook received');
});

app.listen(PORT, () => {
  console.log(`Express webhook server listening at http://localhost:${PORT}`);
});
```

**Start your server:**

```bash
node webhook-server.js
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

**Copy the HTTPS URL** (e.g., `https://abc123.tunnelmole.net`)—you'll use this as your Shopify webhook endpoint.

---

## Step 3: Register Your Webhook with Shopify

You can register webhooks in Shopify via the admin dashboard or the API.

### Option 1: Shopify Admin Dashboard

1. Go to your Shopify store admin
2. Click **Settings** > **Notifications** > **Webhooks**
3. Click **Create webhook**
4. Choose the event (e.g., "Order creation")
5. Paste your Tunnelmole HTTPS URL + endpoint (e.g., `https://abc123.tunnelmole.net/webhook/orders-create`)
6. Set format to **JSON**
7. Click **Save**

### Option 2: Shopify Admin API (cURL example)

```bash
curl -X POST "https://your-store.myshopify.com/admin/api/2023-07/webhooks.json" \
  -H "X-Shopify-Access-Token: <your-access-token>" \
  -H "Content-Type: application/json" \
  -d '{
    "webhook": {
      "topic": "orders/create",
      "address": "https://abc123.tunnelmole.net/webhook/orders-create",
      "format": "json"
    }
  }'
```

Replace `<your-access-token>` and the URL as needed.

---

## Step 4: Test and Debug Shopify Webhooks Locally

**Trigger a webhook event** (e.g., place a test order in your store).

- Shopify will send a POST request to your Tunnelmole URL
- Tunnelmole forwards the request to your local Express app
- You should see the webhook payload logged in your terminal

**Example output:**

```
Received Shopify webhook: { id: 123456789, ... }
```

**Debugging tips:**

- If you don't see the webhook, check that Tunnelmole is running and the URL matches your Express endpoint
- Check Shopify's webhook delivery logs in the admin dashboard for errors
- Use `console.log` or a debugger to inspect the request payload

---

## Security: Verifying Shopify Webhook Signatures

Shopify signs webhook requests with an HMAC header (`X-Shopify-Hmac-Sha256`). You should verify this signature to ensure the request is authentic.

**Add signature verification to your Express handler:**

```js
const crypto = require('crypto');

app.post('/webhook/orders-create', (req, res) => {
  const hmac = req.get('X-Shopify-Hmac-Sha256');
  const secret = '<your-shopify-app-secret>'; // Get this from your Shopify app settings

  const body = JSON.stringify(req.body);
  const hash = crypto
    .createHmac('sha256', secret)
    .update(body, 'utf8')
    .digest('base64');

  if (hash !== hmac) {
    console.log('Webhook signature verification failed');
    return res.status(401).send('Unauthorized');
  }

  console.log('Verified Shopify webhook:', req.body);
  res.status(200).send('Webhook received');
});
```

**Note:** Replace `<your-shopify-app-secret>` with your app's shared secret.

---

## Troubleshooting Common Issues

**1. Shopify says "Failed to deliver webhook"**

- Double-check your Tunnelmole URL and endpoint path
- Ensure your Express app is running and listening on the correct port
- Make sure Tunnelmole is running and hasn't timed out

**2. Webhook handler not triggered**

- Confirm the webhook event is actually firing (e.g., create a real order)
- Check for typos in the endpoint path (e.g., `/webhook/orders-create`)
- Look at Shopify's webhook delivery logs for error details

**3. Signature verification fails**

- Ensure you're using the correct app secret
- Make sure you're hashing the raw request body, not the parsed object
- Use a tool like [Shopify's webhook tester](https://shopify.dev/docs/apps/webhooks/test) for debugging

---

## Feature Comparison: Tunnelmole Features

| Feature                | Tunnelmole         |
|------------------------|--------------------|
| Open source            | ✅ Yes             |
| Free public URLs       | ✅ Yes             |
| Custom domains         | ✅ Paid/Self-host  |
| Self-hostable          | ✅ Yes             |
| Native Node.js         | ✅ Yes             |
| One-line install       | ✅ Yes             |
| Unlimited tunnels      | ❌ No              |
| Bandwidth limits       | ❌ Yes (fair use)  |

---

## Summary and Next Steps

With Tunnelmole, you can develop and debug Shopify webhooks on your local machine—no cloud deployment, no firewall headaches, and no closed-source black boxes. 

**Next steps:**

- Add more webhook endpoints for other Shopify events
- Integrate webhook handling into your app's business logic
- Use Tunnelmole's self-hosting for custom domains or privacy
- Share your public URL with collaborators for live demos

**Ready to build Shopify apps faster?**  
[Get started with Tunnelmole](https://tunnelmole.com/?utm_source=shopify-webhook-article)

---

## FAQ: Shopify Webhooks and Tunnelmole

### How do I test Shopify webhooks locally?

Use Tunnelmole to expose your local Express.js server to the internet. Register the Tunnelmole HTTPS URL as your webhook endpoint in Shopify.

### Is Tunnelmole secure for handling webhooks?

Tunnelmole uses HTTPS for all public URLs. For production, always verify Shopify's HMAC signature and consider self-hosting for maximum control.

### Can I use Tunnelmole for other webhook providers?

Yes! Tunnelmole works with any service that needs to reach your local server—Stripe, GitHub, IFTTT, and more.

### What happens if I restart Tunnelmole?

You'll get a new public URL unless you use a custom domain (paid/self-hosted). Update your Shopify webhook endpoint if the URL changes.

---

## References

- [Shopify Webhooks Documentation](https://shopify.dev/docs/api/webhooks)
- [Tunnelmole GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Website](https://tunnelmole.com/?utm_source=shopify-webhook-article)
- [Express.js Documentation](https://expressjs.com/)

---

**Build, test, and debug Shopify webhooks locally—faster and easier with Tunnelmole.**
