---
title: 'Telegram Webhook: A Developer''s Guide to Local Testing'
description: >-
  Learn how to set up, test, and debug Telegram webhooks on your local machine
  using Node.js and Tunnelmole, an open-source tunneling tool.
tags:
  - telegram webhook
  - telegram bot
  - webhook
  - nodejs
  - javascript
  - expressjs
  - opensource
  - programming
  - coding
  - api
canonical_url: 'https://softwareengineeringstandard.com/2025/08/26/telegram-webhook/'
slug: telegram-webhook
cover_image: >-
  https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp
claire_approved: false
human_approved: true
published: true
---

## Introduction to Telegram Webhooks

Developing a Telegram bot is a fantastic way to automate tasks, deliver information, and create interactive experiences. When building a bot, you have two primary methods for receiving updates from Telegram's servers: `long polling` and `webhooks`.

While long polling is simpler to start with (your application repeatedly asks Telegram for updates), it's often inefficient. It can lead to delays and unnecessary network traffic. This is where Telegram webhooks shine.

A **Telegram webhook** reverses the flow of communication. Instead of your bot asking for updates, Telegram actively sends updates to your bot whenever a user interacts with it. These updates are sent as HTTP POST requests to a public URL that you specify. This is a much more efficient, real-time, and scalable approach for most applications.

However, there's a common challenge developers face: Telegram requires the webhook URL to be a public, HTTPS-enabled endpoint. This poses a problem when you're developing the bot on your local machine, where your server is typically running on `localhost`, a private address inaccessible from the internet.

How can you test your Telegram webhook integration without deploying your code to a public server every time you make a change?

The solution is to use a tunneling tool. In this guide, we'll walk you through setting up a local development environment for your Telegram bot using Node.js, Express.js, and **Tunnelmole**, a free and open-source tool that creates a public URL for your local server.

By the end of this article, you will be able to:
- Create a Telegram bot and get an API token.
- Build a simple Express.js server to handle webhook requests.
- Use Tunnelmole to expose your local server to the internet.
- Register your public URL with Telegram to receive webhook updates.
- Test and debug your bot in real-time.

## Prerequisites

Before we begin, ensure you have the following installed and set up:

- **Node.js and npm**: You can download them from the [official Node.js website](https://nodejs.org/).
- **A Telegram Account**: You'll need this to create and interact with your bot.
- **Basic JavaScript Knowledge**: Familiarity with Express.js is helpful but not strictly required.

## Step 1: Create Your Telegram Bot with BotFather

The first step is to create a new bot on the Telegram platform. Telegram has a dedicated bot for this, aptly named **BotFather**.

1.  **Start a chat with BotFather**: Open your Telegram app, search for `@BotFather`, and start a conversation.
2.  **Create a new bot**: Send the `/newbot` command.
3.  **Choose a name and username**: BotFather will ask you for a display name and a username for your bot. The username must be unique and end in "bot" (e.g., `MyTestTaskBot`).
4.  **Save your API Token**: Once you've successfully created the bot, BotFather will give you a unique API token. This token is your key to controlling the bot. Treat it like a password and keep it secure.

You'll need this token in the upcoming steps, so have it ready.

## Step 2: Build a Local Server with Node.js and Express.js

Now, let's create a simple web server that will listen for the incoming webhook requests from Telegram.

First, set up a new Node.js project:

```bash
mkdir telegram-bot-server
cd telegram-bot-server
npm init -y
npm install express
```

Next, create a file named `index.js` and add the following code:

```javascript
const express = require('express');
const app = express();
const port = 3000;

// Middleware to parse JSON bodies. Telegram sends webhooks as JSON.
app.use(express.json());

// Define the endpoint for our webhook
app.post('/telegram-webhook', (req, res) => {
    console.log("Received a webhook update from Telegram!");

    // The update from Telegram is in the request body
    const update = req.body;

    // Log the entire update object to see its structure
    console.log(JSON.stringify(update, null, 2));

    // Acknowledge receipt of the webhook
    res.status(200).send('OK');
});

app.get('/', (req, res) => {
    res.send('Hello! Your Telegram bot server is running.');
});

app.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});
```

Let's break down this code:
- We import the `express` library to create a web server.
- `app.use(express.json())` is a crucial piece of middleware that tells Express to automatically parse incoming JSON payloads. Telegram sends its updates in this format.
- We define a POST endpoint at `/telegram-webhook`. This is the path Telegram will send requests to.
- Inside the handler, we log the received data (`req.body`) to the console. This is essential for debugging and understanding the data structure Telegram sends.
- We send back a `200 OK` status to let Telegram know we've successfully received the update. If Telegram doesn't receive this confirmation, it will try to resend the webhook, leading to duplicate notifications.

Start your server by running:

```bash
node index.js
```

You should see the message: `Server is listening on http://localhost:3000`. Your server is now running, but it's only accessible on your local machine.

## Step 3: Get a Public URL with Tunnelmole

To allow Telegram's servers to send requests to your local application, you need a public URL. This is where Tunnelmole comes in. Tunnelmole creates a secure tunnel from a public URL to your local machine.

### How Tunnelmole Works

Tunnelmole establishes a persistent connection from your machine to its public service. When a request hits your public Tunnelmole URL, it's instantly forwarded through this tunnel to your local server. The response from your server is then sent back through the tunnel to the user.

![How Tunnelmole works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

One of the key advantages of Tunnelmole is that it's **open source**. Both the client you run on your machine and the server that handles the public URLs are available on GitHub. This provides transparency and allows you to **self-host** the service if you prefer, giving you complete control over your tunneling infrastructure.

### Installing and Running Tunnelmole

You can install Tunnelmole using npm, which is the easiest method if you have Node.js installed.

Open a **new terminal window** (keep your Express server running in the other one) and run:

```bash
sudo npm install -g tunnelmole
```

Alternatively, you can use the installation script for Linux or macOS:
```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

Once installed, running it is simple. Since our Express server is on port `3000`, we run:

```bash
tmole 3000
```

Tunnelmole will start and display a few public URLs. Copy one of the HTTPS URLs. It will look something like this: `https://b3dkls-ip-XX-XX-XX-XX.tunnelmole.net`.

![Tunnelmole generating a public URL](https://softwareengineeringstandard.com/wp-content/uploads/2025/07/tunnelmole-running.png)

This URL is now live and forwards all requests to your `localhost:3000`.

## Step 4: Register Your Webhook URL with Telegram

With your public URL in hand, it's time to tell Telegram where to send updates. You do this by calling the `setWebhook` method in the Telegram Bot API.

You'll need:
1.  Your Bot API Token from Step 1.
2.  Your public HTTPS URL from Tunnelmole from Step 3.

Construct the following URL in your browser or use a tool like `curl`. Replace `<YOUR_BOT_TOKEN>` and `<YOUR_TUNNELMOLE_URL>` with your actual values. The full URL to your endpoint is your Tunnelmole URL plus the `/telegram-webhook` path we defined in our Express app.

**Using `curl` in your terminal:**
```bash
curl "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook?url=<YOUR_TUNNELMOLE_URL>/telegram-webhook"
```

For example:
```bash
curl "https://api.telegram.org/bot123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11/setWebhook?url=https://b3dkls-ip-XX-XX-XX-XX.tunnelmole.net/telegram-webhook"
```

After executing the command, Telegram will respond with a success message:

```json
{
    "ok": true,
    "result": true,
    "description": "Webhook was set"
}
```

Your webhook is now active! Telegram will stop delivering updates via long polling and start sending them to your new URL.

## Step 5: Test Your Telegram Webhook

The setup is complete. Let's test it.

1.  Open Telegram and find the bot you created.
2.  Send it a message, like "Hello, bot!".

Now, look at the terminal where your Express server is running. You should see the console output with the full JSON object from Telegram!

```
Received a webhook update from Telegram!
{
  "update_id": 843621457,
  "message": {
    "message_id": 123,
    "from": {
      "id": 123456789,
      "is_bot": false,
      "first_name": "John",
      "last_name": "Doe",
      "username": "johndoe",
      "language_code": "en"
    },
    "chat": {
      "id": 123456789,
      "first_name": "John",
      "last_name": "Doe",
      "username": "johndoe",
      "type": "private"
    },
    "date": 1678886400,
    "text": "Hello, bot!"
  }
}
```

Congratulations! You are now receiving real-time updates from Telegram directly to your local development server.

## Expanding Your Bot: Sending a Reply

Logging messages is a great start, but a bot should interact back. Let's modify our server to "echo" back any message it receives.

You'll need a library to make HTTP requests from your server to the Telegram API. `axios` is a popular choice.

Install it first:
```bash
npm install axios
```

Now, update your `index.js` file:

```javascript
const express = require('express');
const axios = require('axios');
const app = express();
const port = 3000;

// Your bot token from BotFather
const TELEGRAM_BOT_TOKEN = 'YOUR_BOT_TOKEN';
// The base URL for the Telegram Bot API
const TELEGRAM_API_URL = `https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}`;

app.use(express.json());

app.post('/telegram-webhook', async (req, res) => {
    console.log("Received a webhook update from Telegram!");
    const update = req.body;
    console.log(JSON.stringify(update, null, 2));

    // Ensure it's a message and has text
    if (update.message && update.message.text) {
        const chatId = update.message.chat.id;
        const receivedText = update.message.text;

        // Echo the message back to the user
        try {
            await axios.post(`${TELEGRAM_API_URL}/sendMessage`, {
                chat_id: chatId,
                text: `You said: ${receivedText}`
            });
            console.log("Successfully sent reply to Telegram");
        } catch (error) {
            console.error("Error sending message to Telegram:", error.response.data);
        }
    }

    res.status(200).send('OK');
});

app.get('/', (req, res) => {
    res.send('Hello! Your Telegram bot server is running.');
});

app.listen(port, () => {
    console.log(`Server is listening on http://localhost:${port}`);
});
```

**Key changes:**
1.  We imported `axios`.
2.  We added a variable `TELEGRAM_BOT_TOKEN` (remember to replace `'YOUR_BOT_TOKEN'` with your actual token).
3.  Inside the webhook handler, we extract the `chat.id` and the `text` of the message.
4.  We use `axios.post` to call the `sendMessage` method of the Telegram API, sending the original message back to the same chat.

Stop your server (`Ctrl+C`) and restart it (`node index.js`). Now, send another message to your bot. It should instantly reply!

## Conclusion

Webhooks are the preferred way to build robust and scalable Telegram bots. By using **Tunnelmole**, you can overcome the hurdle of local development and create a seamless workflow for building, testing, and debugging your bot. You can iterate quickly on your code without the friction of constant deployments, all while leveraging a secure, open-source tool that you can even host yourself for maximum control.

You've learned how to:
- Set up a bot and a local server.
- Expose your local server with Tunnelmole.
- Connect Telegram's webhooks to your local environment.
- Process incoming messages and send replies.

Now you have a solid foundation to build more complex and interesting Telegram bots. Happy coding!
