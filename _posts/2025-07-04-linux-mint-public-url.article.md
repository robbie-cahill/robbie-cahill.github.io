---
title: "How to Get a Public URL for Linux Mint with Tunnelmole"
description: "Expose your Linux Mint machine to the internet with a secure public URL using Tunnelmole. Step-by-step guide for developers and tinkerers."
tags: ["linuxmint", "linux", "mint", "publicurl", "opensource", "javascript", "typescript", "python", "programming", "coding", "devops", "cloud", "tunneling", "webhooks", "nodejs", "selfhost", "networking"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/04/how-to-get-a-public-url-for-linux-mint-with-tunnelmole/"
slug: "linux-mint"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Get a Public URL for Linux Mint with Tunnelmole

If you’re running **Linux Mint** and want to share a web server, API, or any local service with the outside world, you’ve probably hit a wall with network configuration, NAT, or firewall rules. Whether you’re developing, testing webhooks, or showing off a project, getting a public URL for Linux Mint can be tricky—especially if you don’t want to mess with your router or expose your home network.

**Tunnelmole** is an open source tunneling tool that makes it easy to get a secure, public URL for any service running on your Linux Mint machine. In this guide, you’ll learn how to use Tunnelmole to expose your Linux Mint desktop or laptop to the internet in minutes, with no cloud deploys, no firewall headaches, and no vendor lock-in.

---

## Why Expose Linux Mint to the Internet?

Linux Mint is a popular choice for developers, makers, and tinkerers. But by default, services running on your Mint machine are only accessible from your local network.

**Common use cases for exposing Linux Mint:**

- Testing webhooks from SaaS providers (Shopify, Stripe, GitHub, etc.)
- Sharing a development site or API with teammates or clients
- Mobile device testing (accessing your dev server from your phone)
- IoT device integration and remote debugging
- Running demos or proof-of-concept apps without deploying to production

But opening ports on your Mint machine is often blocked by NAT, firewalls, or ISP restrictions. That’s where a tunneling tool like Tunnelmole comes in.

---

## What is Tunnelmole?

Tunnelmole is an **open source tunneling tool** that gives your locally running HTTP(s) servers a public URL. It’s designed for developers and hobbyists who need a quick, secure way to expose services running on their machine to the internet.

**Key features:**

- **Open Source:** MIT/AGPLv3 licensed, self-hostable, and auditable.
- **Free public URLs:** Instantly get a public HTTPS URL for your service.
- **Custom subdomains:** Use a memorable subdomain (paid or self-hosted).
- **Native Node.js app:** Works on Linux Mint, Ubuntu, Debian, and more. Precompiled binaries also work on these platforms.
- **No login required:** Start tunneling in seconds.

---

## How Tunnelmole Works on Linux Mint

Tunnelmole creates a secure tunnel from your Linux Mint machine to a public endpoint. Here’s how it works:

1. **You run Tunnelmole on your Mint machine**, specifying the port your service is running on (e.g., 3000).
2. **Tunnelmole connects to the Tunnelmole service** (hosted or self-hosted) and requests a public URL.
3. **Incoming requests to the public URL** are securely forwarded to your local service via a persistent WebSocket tunnel.
4. **You share the public URL** with anyone—no need to open firewall ports or configure NAT.

![How Tunnelmole Works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

---

## Step-by-Step: Expose Your Linux Mint Machine with Tunnelmole

### 1. Prerequisites

- A Linux Mint desktop or laptop
- Node.js 16.10+ (for npm install) **or** ability to run a precompiled binary
- A service running on your Mint machine (e.g., a web server on port 3000)

### 2. Install Tunnelmole on Linux Mint

You can install Tunnelmole in two ways: via npm (if you have Node.js) or by downloading a precompiled binary.

#### **Option 1: Install with npm**

```bash
sudo npm install -g tunnelmole
```

#### **Option 2: Install Precompiled Binary**

If you don’t have Node.js, use the install script:

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

The script auto-detects your OS and installs the right binary.

---

### 3. Start Your Service on Linux Mint

For example, if you’re running a Node.js app:

```bash
node app.js
# or for Python
python3 -m http.server 3000
```

Make sure you know the port your service is listening on (e.g., 3000).

---

### 4. Run Tunnelmole to Get a Public URL

In your terminal, run:

```bash
tmole 3000
```

Replace `3000` with your service’s port.

**Example output:**

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:3000
http://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:3000
```

You now have a public HTTPS URL that forwards to your Linux Mint’s local service!

---

### 5. Test Your Public URL

- Open the HTTPS URL in your browser (from any device).
- You should see your service running from your Linux Mint machine.
- Share the URL with teammates, webhook providers, or test from your phone.

---

## Advanced: Custom Subdomains and Self-Hosting

### Use a Custom Subdomain

Want a memorable, stable URL? Use a custom subdomain:

```bash
tmole 3000 as mymint.tunnelmole.net
```

- **Hosted service:** Custom subdomains require a paid subscription.
- **Self-hosted:** You can run your own Tunnelmole service and use any subdomain you control.

[Learn more about custom subdomains](https://dashboard.tunnelmole.com/?utm_source=tunnelmoleClientGithub)

### Self-Host Tunnelmole

Tunnelmole is fully open source. You can run your own Tunnelmole service for maximum control and privacy.

- [Tunnelmole Client (MIT)](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Service (AGPLv3)](https://github.com/robbie-cahill/tunnelmole-service)

---

## Example: Exposing a React App on Linux Mint

Let’s walk through a real-world example: exposing a React development server running on Linux Mint.

### 1. Start the React App

```bash
npm start
# or
yarn start
```

By default, React runs on port 3000.

### 2. Start Tunnelmole

```bash
tmole 3000
```

Tunnelmole will output a public URL like:

```
https://abc12-ip-203-0-113-42.tunnelmole.net ⟶ http://localhost:3000
```

### 3. Access from Anywhere

Open the HTTPS URL in your browser—your React app is now live on the internet, running from your Linux Mint machine!

---

## Use Cases: Why Tunnelmole is Perfect for Linux Mint Users

- **Webhook development:** Receive webhooks from Stripe, GitHub, or any SaaS directly on your Mint machine.
- **Remote demos:** Share your in-progress app with clients or teammates without deploying.
- **IoT and device testing:** Expose device APIs running on Mint to the cloud.
- **Mobile testing:** Access your dev server from your phone or tablet.
- **Learning and experimentation:** Quickly share projects with friends or online communities.

---

## Security Considerations

- **Public URLs are accessible internet-wide.** Don’t expose sensitive services without authentication.
- **HTTPS by default:** Tunnelmole provides secure URLs.

---

## Troubleshooting: Common Linux Mint Networking Issues

- **Firewall blocks:** If your Mint machine has a local firewall (e.g., `ufw`), ensure the port is open locally.
- **No Node.js?** Use the precompiled binary install method.
- **No outbound internet?** Tunnelmole requires outbound WebSocket access to the Tunnelmole service.
- **Works on all Mint editions:** Cinnamon, MATE, Xfce, and more.

---

## Frequently Asked Questions

### Is Tunnelmole free for Linux Mint?

Yes! You can use Tunnelmole’s hosted service to get free public URLs for your Mint machine. Custom subdomains require a paid plan or self-hosting.

### Can I use Tunnelmole on any Linux Mint edition?

Tunnelmole works on all Linux Mint editions (Cinnamon, MATE, Xfce) as long as you have Node.js or can run the precompiled binary.

### Does Tunnelmole work with Docker containers on Linux Mint?

Absolutely. Run Tunnelmole on Mint and point it to the Docker container’s exposed port.

---

## Summary: Instantly Expose Linux Mint to the Internet

Getting a public URL for your Linux Mint machine doesn’t have to be hard. With Tunnelmole, you can expose any service running on Mint to the internet in seconds—no port forwarding, no cloud deploys, no hassle.

- **Open source, free, and self-hostable**
- **Works on all Linux Mint editions**
- **Perfect for webhooks, demos, and remote testing**

Ready to try it? Install Tunnelmole on your Linux Mint machine and get your public URL today.

---

**Try Tunnelmole now and expose your Linux Mint machine to the world:**

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
tmole 3000
```

[Learn more and get started](https://tunnelmole.com/?utm_source=linux-mint-article)
