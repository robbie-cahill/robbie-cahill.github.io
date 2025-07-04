---
title: "How to Use Tunnelmole as a Reverse Proxy: The Open Source Approach"
description: "Learn how to set up a reverse proxy with Tunnelmole, the open source tunneling tool for developers. Step-by-step guide for Node.js, Python, and more."
tags: ["reverseproxy", "opensource", "javascript", "typescript", "python", "programming", "coding", "devops", "nodejs", "webdevelopment", "cloud", "proxy", "networking", "selfhosted"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/04/how-to-use-tunnelmole-as-a-reverse-proxy-the-open-source-approach/"
slug: "reverse-proxy"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png"
published: true
---

# How to Use Tunnelmole as a Reverse Proxy

Reverse proxies are a foundational tool for modern web development, enabling secure, flexible, and scalable access to internal services. Whether you’re exposing a local API for testing, sharing a development site, or integrating with third-party webhooks, a reverse proxy can make your life much easier.

In this guide, you’ll learn how to use [Tunnelmole](https://tunnelmole.com/?utm_source=reverse-proxy-article), an open source tunneling tool, as a reverse proxy for your local applications. We’ll cover what a reverse proxy is, why you might need one, and provide hands-on examples for Node.js, Python, and Dockerized services.

---

## What is a Reverse Proxy?

A **reverse proxy** is a server that sits in front of one or more backend servers and forwards client requests to them. Unlike a traditional (forward) proxy, which routes outbound traffic from clients to the internet, a reverse proxy handles inbound traffic, acting as a gateway between external users and your internal services.

**Key benefits of using a reverse proxy:**

- **Expose local services to the internet** without deploying to production
- **Secure endpoints** with HTTPS, even for local development
- **Test webhooks** from third-party providers (Shopify, Stripe, GitHub, etc.)
- **Share work-in-progress** with teammates or clients
- **Bypass NAT/firewall restrictions** for IoT, mobile, or remote devices

---

## Why Use Tunnelmole as a Reverse Proxy?

Tunnelmole is an open source tunneling tool that makes it easy to create a reverse proxy for any local HTTP or HTTPS service. Unlike some proprietary solutions, Tunnelmole is:

- **Open Source**: Both client and server are open source, so you can self-host or audit the code.
- **Free to Use**: Get public URLs for your local services at no cost.
- **Custom Subdomains**: Use your own subdomain (with a paid plan or self-hosted).
- **Self-Hostable**: Run your own Tunnelmole server for full control.
- **Native Node.js**: Easy integration with JavaScript and TypeScript projects.

Tunnelmole is ideal for developers who want a transparent, flexible, and developer-friendly reverse proxy solution.

---

## How Tunnelmole Works as a Reverse Proxy

Tunnelmole creates a secure tunnel from your local machine to a public URL. When someone accesses the public URL, Tunnelmole forwards the request to your local server—acting as a reverse proxy.

![How Tunnelmole Works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

**Typical workflow:**

1. Start your local server (e.g., Node.js on port 3000).
2. Run Tunnelmole to expose that port.
3. Receive a public HTTPS URL that forwards requests to your local server.

---

## Installing Tunnelmole

Tunnelmole supports Linux, macOS, and Windows. You can install it via npm or download a precompiled binary.

### Install with npm (requires Node.js 16.10+)

```bash
sudo npm install -g tunnelmole
```

### Install via script (Linux/macOS/WSL)

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

### Windows

1. [Download tmole.exe](https://tunnelmole.com/downloads/tmole.exe)
2. Add it to your PATH.

For more details, see the [official Tunnelmole installation guide](https://github.com/robbie-cahill/tunnelmole-client/#installation).

---

## Using Tunnelmole as a Reverse Proxy: Step-by-Step

Let’s walk through practical examples of using Tunnelmole as a reverse proxy for different development scenarios.

### 1. Exposing a Local Node.js App

Suppose you have an Express app running on port 3000:

```js
// app.js
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello from local Node.js!'));
app.listen(3000, () => console.log('App listening on port 3000'));
```

Start your app:

```bash
node app.js
```

Now, use Tunnelmole to create a reverse proxy:

```bash
tmole 3000
```

You’ll get output like:

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://abc12.tunnelmole.net ⟶ http://localhost:3000
http://abc12.tunnelmole.net ⟶ http://localhost:3000
```

Anyone visiting the public URL will be reverse proxied to your local app.

---

### 2. Reverse Proxy for a Python Flask App

Let’s say you have a Flask app on port 5000:

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello from Flask!"

if __name__ == '__main__':
    app.run(port=5000)
```

Start your Flask app:

```bash
python app.py
```

Expose it with Tunnelmole:

```bash
tmole 5000
```

Now, your Flask app is accessible via a public reverse proxy URL.

---

### 3. Exposing a Dockerized Service

If you’re running a service in Docker, map the container port to your host, then use Tunnelmole:

```bash
docker run -p 8080:80 nginx
tmole 8080
```

Tunnelmole will reverse proxy requests from the public URL to your Nginx container.

---

### 4. Using a Custom Subdomain

For a stable, memorable URL, use a custom subdomain (requires a paid plan or self-hosting):

```bash
tmole 3000 as myapp.tunnelmole.net
```

Now, your reverse proxy URL will be `https://myapp.tunnelmole.net`.

---

## Advanced: Integrating Tunnelmole in Node.js/TypeScript Projects

Tunnelmole can be used programmatically as an npm dependency:

```bash
npm install --save tunnelmole
```

Example usage in a Node.js script:

```js
import { tunnelmole } from 'tunnelmole';

(async () => {
  const url = await tunnelmole({ port: 3000 });
  console.log('Public reverse proxy URL:', url);
})();
```

You can also specify a custom domain:

```js
const url = await tunnelmole({
  port: 3000,
  domain: 'myapp.tunnelmole.net'
});
```

---

## Common Use Cases for Reverse Proxy with Tunnelmole

- **Webhook Development**: Receive webhooks from Stripe, Shopify, GitHub, etc., on your local machine.
- **Mobile App Testing**: Test APIs from real devices using a public URL.
- **Client Demos**: Share a work-in-progress site with stakeholders.
- **IoT Device Integration**: Expose local device endpoints for remote access.
- **Bypass Corporate Firewalls**: Access local services from anywhere.

---

## Security Considerations

When using a reverse proxy to expose local services, keep these best practices in mind:

- **Use HTTPS URLs**: Tunnelmole provides HTTPS by default for security.
- **Limit Exposure**: Only expose ports/services you need.
- **Authentication**: Add authentication to sensitive endpoints.
- **Monitor Usage**: Be aware that public URLs are accessible internet-wide.

---

## Self-Hosting Tunnelmole for Full Control

If you need complete control over your reverse proxy setup, you can self-host the Tunnelmole service. This is ideal for:

- Enterprise environments
- Custom domain requirements
- Regulatory compliance

See the [Tunnelmole Service GitHub repo](https://github.com/robbie-cahill/tunnelmole-service/) for setup instructions.

---

## Frequently Asked Questions

**Q: Is Tunnelmole really open source?**  
A: Yes, both the client and server are open source. You can self-host or use the hosted service.

**Q: Does Tunnelmole hide my IP?**  
A: No, the hosted service adds an `X-Forwarded-For` header with your IP for transparency. You can self-host and modify this if needed.

---

## Summary

Tunnelmole makes it easy to set up a reverse proxy for any local service, whether you’re building with Node.js, Python, Docker, or other platforms. As an open source, self-hostable, and developer-friendly tool, Tunnelmole is a great choice for anyone needing secure, flexible, and fast reverse proxy access.

**Ready to try Tunnelmole as your reverse proxy?**

- [Get started with Tunnelmole](https://tunnelmole.com/?utm_source=reverse-proxy-article)
- [View the GitHub repo](https://github.com/robbie-cahill/tunnelmole-client/)
- [Read the full documentation](https://github.com/robbie-cahill/tunnelmole-client/#readme)

---

**Next Steps:**

- Install Tunnelmole and expose your first local service
- Try integrating with webhooks or mobile devices

---

**Tunnelmole: The open source reverse proxy for developers.**
