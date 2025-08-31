---
title: 'Shopify Webhooks: The Complete Guide for Developers'
description: >-
  Learn everything you need to know about Shopify webhooks. This guide covers
  setup, local testing with NodeJS and Tunnelmole, security, and best practices.
tags:
  - shopify webhook
  - shopify
  - webhook
  - nodejs
  - expressjs
  - tunnelmole
  - opensource
  - javascript
  - typescript
  - python
  - programming
  - coding
  - api
  - ecommerce
canonical_url: 'https://softwareengineeringstandard.com/2025/08/31/shopify-webhook-2/'
slug: shopify-webhook
cover_image: >-
  https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp
claire_approved: true
human_approved: true
published: true
---

In the world of e-commerce, real-time data is king. Whether you're synchronizing orders, updating inventory, or connecting to a third-party service, the ability to react instantly to events is crucial for building modern, efficient applications. This is where Shopify Webhooks come in, serving as the backbone for countless app integrations and custom workflows.

However, for developers, working with webhooks presents a classic challenge: Shopify needs to send data to a public, internet-accessible URL, but your development environment runs on `localhost`. This disconnect can lead to slow, frustrating development cycles that involve constantly deploying code to a staging server just to test a small change.

This comprehensive guide will solve that problem for you. We'll dive deep into what Shopify webhooks are, why they're essential, and most importantly, how you can use an open-source tool called **Tunnelmole** to seamlessly test and debug your webhooks on your local machine.

By the end of this article, you'll have a complete workflow for building, testing, and securing a Shopify webhook handler using Node.js, from initial setup to production-ready best practices.

### What are Shopify Webhooks?

At its core, a webhook is an automated message sent from an application when a specific event occurs. Think of it as a reverse API. Instead of your application repeatedly asking Shopify, "Is there a new order yet?", Shopify proactively tells your application, "Hey, a new order just came in!" by sending an HTTP POST request to a URL you specify.

This event-driven approach is incredibly powerful and efficient. It eliminates the need for constant polling, which saves server resources and provides you with data the moment it becomes available.

#### Common Use Cases for Shopify Webhooks

Developers leverage Shopify webhooks to build a vast array of features and integrations. Here are some of the most common use cases:

*   **Order Fulfillment and Syncing:** When an `orders/create` event happens, a webhook can instantly send the order details to a third-party logistics (3PL) provider or an internal inventory management system.
*   **Customer Relationship Management (CRM):** The `customers/create` event can trigger a webhook to add the new customer's details to a CRM like HubSpot or Salesforce, ensuring your marketing and sales teams have the latest data.
*   **Product and Inventory Management:** Using the `products/update` webhook, you can sync product details (like price or description) with other platforms or use `inventory_levels/update` to keep stock levels synchronized across multiple channels.
*   **Custom Notifications:** Send customized SMS, email, or Slack notifications for specific events, such as high-value orders (`orders/paid`) or abandoned checkouts (`checkouts/create`).
*   **Data Warehousing and Analytics:** Capture every order, product, and customer event and send it to a data warehouse for in-depth analysis and business intelligence reporting.

Shopify provides a comprehensive list of webhook topics, covering nearly every event that can happen in a store's lifecycle. You can subscribe to events related to carts, checkouts, orders, customers, products, and more. Each webhook delivery includes a JSON payload containing the relevant data for that event.

Here is an example of a JSON payload for a new order:

```json
{
  "id": 2729912345678,
  "email": "customer@example.com",
  "created_at": "2025-07-04T10:30:00-04:00",
  "total_price": "59.99",
  "line_items": [
    {
      "sku": "TSHIRT-BLK-L",
      "name": "Black T-Shirt - Large",
      "quantity": 1,
      "price": "29.99"
    },
    {
      "sku": "STICKER-PACK",
      "name": "Awesome Sticker Pack",
      "quantity": 1,
      "price": "10.00"
    }
  ],
  "shipping_address": {
    "first_name": "John",
    "last_name": "Doe",
    "address1": "123 Fake St",
    "city": "Anytown",
    "zip": "12345",
    "country": "US"
  }
}
```

### The Challenge: Testing Webhooks on Localhost

The primary hurdle in webhook development is the URL requirement. Shopify's servers exist on the public internet and need a public URL to send notifications. Your local development server, typically running at an address like `http://localhost:3000`, is only accessible on your own computer. Shopify has no way to reach it.

So, how do developers traditionally get around this?

1.  **Deploy on Every Change:** The most common method is to deploy the code to a public staging or development server every time you make a change. This is incredibly inefficient. A simple logic change could require a full build and deployment process, turning a 30-second fix into a 10-minute task.
2.  **Manual Simulation:** Another approach involves manually copying an example payload, pasting it into a tool like Postman or `curl`, and sending it to your local server. This is better than constant deployment, but it's tedious, error-prone, and doesn't accurately replicate the headers and metadata sent by Shopify, which are crucial for security and debugging.

Neither of these methods is ideal. They break your development flow, slow you down, and add unnecessary complexity. What you really need is a way to make your local server temporarily public.

### The Solution: Tunnelmole for Effortless Local Development

This is where a tunneling tool becomes an essential part of your toolkit. **Tunnelmole** is an open-source application that creates a secure tunnel from a public URL to your local development environment.

When you run Tunnelmole, it generates a unique, public HTTPS URL (e.g., `https://random-subdomain.tunnelmole.net`). You can then provide this URL to Shopify. When Shopify sends a webhook to this URL, Tunnelmole forwards it through the tunnel directly to your application running on `localhost`.

![How Tunnelmole works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

This setup allows you to:

*   **Receive Real Webhooks:** Get actual, live webhook deliveries from Shopify directly to your local machine.
*   **Use a Real Debugger:** Set breakpoints in your code and step through your webhook handler line-by-line to inspect the payload, headers, and logic in real-time.
*   **Iterate Quickly:** Make a change to your code, save the file, and trigger the webhook again. No deployment needed.
*   **Emphasize Open Source and Self-Hosting:** Tunnelmole is fully open-source, meaning its code is transparent and can be audited. For maximum control and privacy, you can also self-host the Tunnelmole service on your own server.

### Step-by-Step: Testing Shopify Webhooks with Node.js and Tunnelmole

Let's walk through the entire process of setting up a webhook handler, exposing it with Tunnelmole, and receiving a live test notification from Shopify.

#### Prerequisites

*   **Node.js and npm:** Make sure you have a recent version of Node.js installed.
*   **A Shopify Partner Account and Development Store:** This is free to create and allows you to test your apps and integrations safely.
*   **A Text Editor:** Such as VS Code.

#### Step 1: Create a Basic Express.js Server

First, let's create a simple Node.js application using the Express framework to listen for incoming webhooks.

Create a new project folder, initialize it with npm, and install Express:

```bash
mkdir shopify-webhook-handler
cd shopify-webhook-handler
npm init -y
npm install express
```

Now, create a file named `server.js` and add the following code. This sets up a web server with a single endpoint, `/webhooks/orders`, which will listen for POST requests.

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Use built-in middleware for parsing JSON.
// We'll modify this later for signature verification.
app.use(express.json());

app.get('/', (req, res) => {
  res.send('Shopify Webhook Handler is running!');
});

app.post('/webhooks/orders', (req, res) => {
  console.log('🎉 Received a new webhook!');
  console.log(JSON.stringify(req.body, null, 2));

  // Acknowledge receipt of the webhook with a 200 OK
  res.status(200).send('Webhook received');
});

app.listen(PORT, () => {
  console.log(`Server is listening on http://localhost:${PORT}`);
});
```

Run your server with `node server.js`. It's now listening for requests on your local port 3000.

#### Step 2: Install and Run Tunnelmole

Next, let's get a public URL. The easiest way to install Tunnelmole is using the script for Linux/Mac or by downloading the binary for Windows.

For Linux, Mac, and Windows Subsystem for Linux (WSL):

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

Alternatively, if you have Node.js installed, you can use npm:

```bash
sudo npm install -g tunnelmole
```

With your local server running on port 3000, open a **new terminal window** and run:

```bash
tmole 3000
```

Tunnelmole will start and display your public URLs. Copy the HTTPS URL—it will look something like this:

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:3000
http://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:3000
```

This public URL is now forwarding all traffic directly to your local server on port 3000.

#### Step 3: Create the Webhook in Shopify

Now, let's tell Shopify where to send its webhook notifications.

1.  Log in to your Shopify development store's admin panel.
2.  Navigate to **Settings** in the bottom-left corner, then click on **Notifications**.
3.  Scroll all the way down to the **Webhooks** section and click **Create webhook**.
4.  Fill out the form:
    *   **Event:** Select the event you want to subscribe to. Let's use `Order creation` for this example.
    *   **Format:** Choose `JSON`.
    *   **URL:** Paste your public Tunnelmole URL, followed by the path to your webhook handler. For our example, this would be: `https://cqcu2t-ip-49-185-26-79.tunnelmole.net/webhooks/orders`.
    *   **Webhook API version:** Select the latest stable version.
5.  Click **Save**.

Your webhook is now active.

#### Step 4: Trigger a Test Webhook

You can now trigger a test event in two ways:

1.  **From the Shopify Admin:** On the Webhooks page, you'll see your newly created webhook. Click the **Send test notification** button next to it.
2.  **Perform the Action:** Alternatively, you can perform the actual action in your store. Since we subscribed to `Order creation`, just create a draft order and mark it as paid.

Go back to the terminal where your Node.js server is running. You should see the webhook payload printed to the console!

```
🎉 Received a new webhook!
{
  "id": 1234567890123,
  "name": "#1001",
  // ... and the rest of the order data
}
```

You have successfully received a live Shopify webhook on your local machine. You can now set a breakpoint in your `/webhooks/orders` handler and use a debugger to inspect the request in detail.

### Securing Your Shopify Webhook: Signature Verification

Receiving webhooks is great, but in a production environment, you must verify that they are actually coming from Shopify. Otherwise, anyone who discovers your webhook URL could send you fake data.

Shopify solves this by signing each webhook request with a **secret key**. When you create a webhook, Shopify generates a secret that is shared only between your app and Shopify. Each request includes a header, `X-Shopify-Hmac-Sha256`, containing a cryptographic signature.

Your job is to calculate the same signature on your end and ensure it matches the one Shopify sent.

#### Step 1: Update Your Server for HMAC Verification

First, you need the **raw request body** to generate the signature. The `express.json()` middleware modifies the body, so we need to use `express.raw` instead for our webhook endpoint. You'll also need Node's built-in `crypto` module.

Update your `server.js` file:

```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();
const PORT = 3000;

// IMPORTANT: Your Shopify Webhook Secret Key
const SHOPIFY_WEBHOOK_SECRET = 'your_shared_secret_from_shopify_dashboard';

// Middleware to verify the webhook signature
const verifyShopifyWebhook = (req, res, buf, encoding) => {
  const hmacHeader = req.get('X-Shopify-Hmac-Sha256');
  if (!hmacHeader) {
    console.error('Webhook missing X-Shopify-Hmac-Sha256 header');
    throw new Error('Webhook verification failed: Missing HMAC header');
  }

  const hash = crypto
    .createHmac('sha256', SHOPIFY_WEBHOOK_SECRET)
    .update(buf, encoding)
    .digest('base64');

  // Use crypto.timingSafeEqual to prevent timing attacks
  const hmacBuffer = Buffer.from(hmacHeader, 'base64');
  const hashBuffer = Buffer.from(hash, 'base64');

  if (!crypto.timingSafeEqual(hmacBuffer, hashBuffer)) {
    console.error('Webhook verification failed: HMAC mismatch');
    throw new Error('Webhook verification failed: Signature mismatch');
  }

  console.log('✅ Webhook signature verified successfully');
};


app.get('/', (req, res) => {
  res.send('Shopify Webhook Handler is running!');
});

// Apply the raw body parser and our verification middleware ONLY to the webhook route
app.post('/webhooks/orders', express.raw({ type: 'application/json' }), (req, res, next) => {
  try {
    verifyShopifyWebhook(req, res, req.body, 'utf8');
    // If verification passes, parse the JSON body and continue
    req.body = JSON.parse(req.body.toString());
    next();
  } catch (error) {
    res.status(401).send(error.message);
  }
}, (req, res) => {
  console.log('🎉 Received a new verified webhook!');
  console.log(JSON.stringify(req.body, null, 2));

  res.status(200).send('Webhook received and verified');
});

app.listen(PORT, () => {
  console.log(`Server is listening on http://localhost:${PORT}`);
});
```

#### Step 2: Get Your Webhook Secret Key

Go back to your webhook's configuration page in the Shopify admin (**Settings > Notifications > Webhooks**). You will see the shared secret displayed. Click to reveal it, copy it, and paste it into the `SHOPIFY_WEBHOOK_SECRET` constant in your `server.js` file.

Restart your Node server and send another test notification. This time, you should see the "Webhook signature verified successfully" message in your console, confirming that your security check is working correctly. If you try to send a request with an invalid signature (e.g., from Postman without the correct HMAC header), the server will correctly reject it with a `401 Unauthorized` error.

### Production Best Practices

With your handler working and secured, consider these best practices for a production-grade system:

1.  **Asynchronous Processing:** A webhook handler should response immediately with a `200 OK` status to let Shopify know it received the event. Don't perform long-running tasks like calling another API or processing a large file directly in the request handler. This can lead to timeouts. Instead, use a background job queue (e.g., BullMQ, RabbitMQ) to process the data asynchronously.
2.  **Idempotency:** Shopify's system guarantees "at-least-once" delivery, which means in rare cases, you might receive the same webhook more than once. Design your handler to be idempotent—meaning it can safely process the same event multiple times without causing issues. A common strategy is to log the `id` of each received event and skip any duplicates.
3.  **Error Handling and Monitoring:** Implement robust logging to track incoming webhooks and any errors that occur during processing. Set up monitoring and alerts to notify you if your endpoint starts failing, as Shopify will stop sending webhooks to an endpoint that fails repeatedly.
4.  **API Versioning:** Webhooks are tied to a specific Shopify API version. Shopify periodically releases new versions and retires old ones. Be sure to subscribe to a stable API version and have a plan for upgrading your code before your current version is sunset.

### Conclusion

Shopify webhooks are a powerful tool for creating dynamic, event-driven e-commerce applications. While local development has traditionally been a bottleneck, tools like the open-source **Tunnelmole** completely change the game. By creating a secure, public tunnel to your local machine, you can accelerate your development workflow, debug issues in real-time, and build more reliable integrations.

By following this guide, you now have a complete, secure, and efficient process for handling Shopify webhooks with Node.js. You can receive live events, verify their authenticity, and focus on building great features instead of fighting with deployment pipelines.

Ready to streamline your Shopify development? **Get started with Tunnelmole today** and see how easy local webhook testing can be.
