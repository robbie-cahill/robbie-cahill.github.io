---
title: "How to Handle Zapier Webhooks Locally with Tunnelmole and Express.js"
description: "Step-by-step guide to receiving and testing Zapier webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool."
tags: ["zapier", "webhook", "expressjs", "tunnelmole", "opensource", "nodejs", "automation", "integration"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/03/how-to-handle-zapier-webhooks-locally-with-tunnelmole-and-express-js/"
slug: "zapier-webhook-tunnelmole-express"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Handle Zapier Webhooks Locally with Tunnelmole and Express.js

**Learn how to receive and debug Zapier webhooks on your local Express.js app using Tunnelmole, the open source tunneling tool. Step-by-step guide for developers and automation pros.**

---

## Introduction

Zapier is the go-to platform for automating workflows between thousands of apps. But when you’re building or testing a Zap that sends webhooks to your own code, you hit a common roadblock: Zapier requires a public HTTPS endpoint, but your Express.js app is running on `localhost`.

**Tunnelmole** solves this instantly by giving your local server a secure, public URL. In this guide, you’ll learn how to:

- Set up a local Express.js app to receive Zapier webhooks
- Use Tunnelmole to expose your local server to the internet
- Configure a Zapier webhook action to hit your local endpoint
- Debug and verify webhook delivery end-to-end

Whether you’re building custom integrations, testing automations, or learning about webhooks, this tutorial will get you up and running—no cloud deployment or firewall changes required.

---

## What are Zapier Webhooks?

Zapier webhooks let you connect any app or service to your own code. With the “Webhooks by Zapier” app, you can:

- **Trigger a Zap** when your server receives a webhook (Catch Hook)
- **Send data** from a Zap to your server (POST, GET, PUT, etc.)

This guide focuses on the **“POST”** action—where Zapier sends a POST request to your endpoint with data from your Zap.

**Common use cases:**

- Integrate SaaS tools with your custom backend
- Automate notifications, data sync, or business logic
- Prototype new APIs and integrations

When a Zap runs, Zapier sends a POST request (with JSON or form data) to your specified URL.

---

## Why You Need a Public URL for Zapier Webhooks

Zapier’s servers must be able to reach your webhook endpoint over the public internet. If your Express.js app is running on `localhost:3000`, Zapier can’t access it directly.

**You need a public HTTPS URL that forwards requests to your local server.**

**Typical workflow:**

1. Start your Express.js app on `localhost:3000`
2. Use Tunnelmole to get a public URL (e.g., `https://abc123.tunnelmole.net`)
3. Configure your Zap to send webhooks to this URL
4. Zapier sends webhooks to your Tunnelmole URL, which forwards them to your local app

---

## Why Use Tunnelmole for Zapier Webhooks?

Tunnelmole is a modern, open source tunneling tool that makes local webhook development fast and easy:

- 100% open source (MIT/AGPLv3)
- Free public HTTPS URLs (no login required)
- Self-hostable for privacy and custom domains
- Native Node.js app, easy one-line install
- Works on Mac, Linux, Windows

## Step 1: Create a Local Express.js Webhook Receiver

If you already have an endpoint to receive Zapier webhooks, you can skip this section.

Let’s build a simple Express.js app that listens for POST requests from Zapier.

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

// Parse JSON and form data
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// Webhook endpoint (e.g., /webhook/zapier)
app.post('/webhook/zapier', (req, res) => {
  console.log('Received Zapier webhook:', req.body);
  // You can add validation or business logic here
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

**Copy the HTTPS URL** (e.g., `https://abc123.tunnelmole.net`)—you’ll use this as your Zapier webhook endpoint.

---

## Step 3: Configure a Zapier Webhook Action

Now, let’s set up a Zap in Zapier to send a webhook to your local server (via Tunnelmole).

### 1. Create a New Zap

- Go to [Zapier](https://zapier.com/) and log in.
- Click **Create Zap**.

### 2. Set Up a Trigger

- Choose any trigger app (e.g., “Schedule by Zapier” for testing).
- Configure the trigger as needed.

### 3. Add a Webhooks by Zapier Action

- Click **+ Add Action**.
- Search for **Webhooks by Zapier**.
- Choose **POST**.

### 4. Configure the Webhook

- **URL:** Paste your Tunnelmole HTTPS URL + endpoint (e.g., `https://abc123.tunnelmole.net/webhook/zapier`)
- **Payload Type:** `json` (recommended)
- **Data:** Add key-value pairs to send (e.g., `name: "Tunnelmole Test"`)
- **Headers:** (Optional) Add custom headers if needed

**Example configuration:**

| Field         | Value                                      |
|---------------|--------------------------------------------|
| URL           | https://abc123.tunnelmole.net/webhook/zapier |
| Payload Type  | json                                       |
| Data          | name: Tunnelmole Test                      |

### 5. Test the Action

- Click **Test & Review** or **Test & Continue**.
- Zapier will send a POST request to your Tunnelmole URL.

---

## Step 4: Test and Debug Zapier Webhooks Locally

**Check your Express app’s terminal:**

You should see the webhook payload logged:

```
Received Zapier webhook: { name: 'Tunnelmole Test' }
```

**Debugging tips:**

- If you don’t see the webhook, check that Tunnelmole is running and the URL matches your Express endpoint
- Check Zapier’s task history for errors
- Use `console.log` or a debugger to inspect the request payload

---

## Security: Validating Zapier Webhook Requests

Unlike Shopify, Zapier webhooks don’t include a built-in signature or secret by default. If you want to restrict access:

- **Add a secret token as a header or query param in your Zapier action**
- **Check for this token in your Express handler**

**Example:**

In Zapier, add a header:

| Key           | Value         |
|---------------|--------------|
| X-Webhook-Key | my-secret-key |

In your Express handler:

```js
app.post('/webhook/zapier', (req, res) => {
  const key = req.get('X-Webhook-Key');
  if (key !== 'my-secret-key') {
    return res.status(401).send('Unauthorized');
  }
  console.log('Received verified Zapier webhook:', req.body);
  res.status(200).send('Webhook received');
});
```

**Note:** Never expose secrets in public repos or logs.

---

## Troubleshooting Common Issues

**1. Zapier says “We had trouble sending your request”**

- Double-check your Tunnelmole URL and endpoint path
- Ensure your Express app is running and listening on the correct port
- Make sure Tunnelmole is running and hasn’t timed out

**2. Webhook handler not triggered**

- Confirm the Zap is actually running (check Zapier task history)
- Check for typos in the endpoint path (e.g., `/webhook/zapier`)
- Look at Zapier’s error details for more info

**3. Security/validation fails**

- Ensure your secret token matches between Zapier and your Express handler
- Check for header name typos or case sensitivity

---

## Summary and Next Steps

With Tunnelmole, you can develop and debug Zapier webhooks on your local machine—no cloud deployment, no firewall headaches, and no closed-source black boxes.

**Next steps:**
- Add more webhook endpoints for different Zaps or apps
- Integrate webhook handling into your app’s business logic
- Use Tunnelmole’s self-hosting for custom domains or privacy
- Share your public URL with collaborators for live demos

**Ready to automate faster?**  
[Get started with Tunnelmole](https://tunnelmole.com/?utm_source=zapier-webhook-article)

---

## FAQ: Zapier Webhooks and Tunnelmole

### How do I test Zapier webhooks locally?

Use Tunnelmole to expose your local Express.js server to the internet. Register the Tunnelmole HTTPS URL as your webhook endpoint in Zapier.

### Is Tunnelmole secure for handling webhooks?

Tunnelmole uses HTTPS for all public URLs. For production, always validate incoming requests (e.g., with a secret token) and consider self-hosting for maximum control.

### Can I use Tunnelmole for other webhook providers?

Yes! Tunnelmole works with any service that needs to reach your local server—Stripe, GitHub, Shopify, IFTTT, and more.

### What happens if I restart Tunnelmole?

You’ll get a new public URL unless you use a custom domain (paid/self-hosted). Update your Zapier webhook endpoint if the URL changes.

---

## References

- [Zapier Webhooks Documentation](https://zapier.com/help/doc/how-get-started-webhooks-zapier)
- [Tunnelmole GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Website](https://tunnelmole.com/?utm_source=zapier-webhook-article)
- [Express.js Documentation](https://expressjs.com/)

---

**Build, test, and debug Zapier webhooks locally—faster and easier with Tunnelmole.**