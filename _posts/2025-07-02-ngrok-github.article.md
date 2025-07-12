---
title: "Ngrok GitHub: The New Ngrok Isn’t Open Source, But Here’s a drop-in Open Source Alternative"
description: "Looking for an open source tunneling solution? Discover Tunnelmole, a fully open source tunneling tool available on GitHub, with free public URLs and self-hosting options. You won't find the latest ngrok in the ngrok github repos"
slug: "the-latest-ngrok-is-not-on-the-ngrok-github"
tags: ["ngrok", "github", "opensource", "tunneling", "devtools"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/02/the-latest-ngrok-is-not-on-the-ngrok-github/"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# Looking for an open source tunneling solution? Meet Tunnelmole 
If you’re searching for an open source tunneling solution, you won’t find the latest version of ngrok in the ngrok GitHub repositories. Ngrok has not been open source for several years, and only the older v1 code remains available on GitHub.

**Tunnelmole** is the answer. It’s a modern, open source alternative to ngrok, available right now on [GitHub](https://github.com/robbie-cahill/tunnelmole-client). Tunnelmole gives you free public URLs for your localhost, supports self-hosting, and is built for developers who value transparency and control.

In this article, you’ll learn:

- Why ngrok isn’t on GitHub since many years ago (and what that means for you)
- What makes Tunnelmole different 
- How to get started with Tunnelmole in seconds
- Real-world use cases for webhooks, API testing, and remote collaboration
- A feature-by-feature comparison: Tunnelmole vs. ngrok 

---

## Why you won't find the latest ngrok in the ngrok GitHub repos
Ngrok was open source only in its early versions—specifically, ngrok v1. That version's code is still available in the ngrok GitHub repositories. However, starting with ngrok v2, the project became closed-source. The creators chose to keep the newer versions proprietary and as a result, the latest features and updates are no longer shared in the public ngrok GitHub repos. Only the old v1 code remains open source; any recent ngrok releases are not available on GitHub.

There is a GitHub issue [here](https://web.archive.org/web/20201019191920/https://github.com/inconshreveable/ngrok/issues/255#issuecomment-221707261) from 2015 with users reacting with surprise that the v2 code is closed source. The author of ngrok (inconshreveable) replied and said "Thanks! Sadly the realities and incentives of running ngrok as a business mean that it's better off for 2.0 to be closed." - so its unlikely the newest ngrok versions will ever be open source.

---

## Tunnelmole: 100% Open Source, On GitHub, and Free to Use

Here’s what sets Tunnelmole apart:

- **Fully open source**: Both the [Tunnelmole client](https://github.com/robbie-cahill/tunnelmole-client) (MIT) and [Tunnelmole service](https://github.com/robbie-cahill/tunnelmole-service) (AGPLv3) are on GitHub.
- **Free public URLs**: Instantly get a public HTTPS URL for your local server—no login required.
- **Self-hosting**: Run your own Tunnelmole server for maximum privacy and custom domains.
- **Native Node.js app**: Easy to install, script, and integrate with your dev workflow.
- **One-line install**: Get started in seconds on Linux, Mac, or Windows.

---

## Tunnelmole vs. ngrok (Free): Feature Comparison

| Feature                | Tunnelmole         | ngrok      |
|------------------------|--------------------|-------------------|
| Open source            | ✅ Yes             | ❌ No             |
| Free public URLs       | ✅ Yes             | ✅ Yes            |
| Custom domains         | ✅ Paid/Self-host  | ✅ Paid only      |
| Self-hostable          | ✅ Yes             | ❌ No             |
| Native Node.js         | ✅ Yes             | ❌ No             |
| One-line install       | ✅ Yes             | ❌ No             |
| Bandwidth limits       | ❌ Yes (fair use, or self host to set your own)  | ❌ Yes            |

---

## How to Get Tunnelmole

There are a few options to install Tunnelmole, pick the one thats easiest for you.

### Universal installer for Linux, Mac and Windows Subsystem for Linux
Use the one-line install script. Copy and paste the following into a terminal:

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

### Install via NPM
Tunnelmole is a native Node.JS application so can be installed directly from NPM.

```bash
sudo npm install -g tunnelmole
```


### Windows without WSL
For Windows without WSL, download the binary here: [tmole.exe](https://tunnelmole.com/downloads/tmole.exe).


### Expose Your Local Server with Tunnelmole

Start your local app (e.g., on port 8080), then run:

```bash
tmole 8080
```

You’ll get a public HTTPS URL like:

```
https://df34.tunnelmole.net ⟶ http://localhost:8080
```

Share this URL with anyone, or use it for webhook testing, mobile device previews, or remote demos.

---

### Optional: Integrate with Your Node.js/TypeScript Project

Tunnelmole is available as an npm dependency for programmatic use:

```bash
npm install --save tunnelmole
```

Example usage in your code:

```js
import { tunnelmole } from 'tunnelmole';

const url = await tunnelmole({ port: 3000 });
// url = https://idsq6j-ip-157-211-195-169.tunnelmole.net
```

This is a great option for IoT and Robotics - anything needing public URLs generated from code.

## Why Open Source Matters for Tunneling Tools

When you use a tunneling tool, you’re trusting it with your local development environment and your data. Here’s why open source is critical:

- **Transparency**: Audit the code for security, privacy, and reliability.
- **Control**: Modify or extend the tool to fit your workflow.
- **Self-hosting**: Run your own instance for maximum privacy and compliance.
- **Community-driven**: Contribute features, report bugs, and help shape the roadmap.

Tunnelmole’s code is open for inspection and improvement. You can see exactly how it works, or even run your own server for total control.

---

## Real-World Use Cases for Tunnelmole

- **Webhook development**: Test Stripe, GitHub, or IFTTT webhooks locally—no need to deploy.
- **API prototyping**: Share your local API with teammates or clients instantly.
- **Mobile device testing**: Preview your local site on real devices, not just emulators.
- **Remote demos**: Show off your work to anyone, anywhere, without firewall headaches.
- **CI/CD integration**: Automate public URL creation in your build/test pipelines.

---

## How Tunnelmole Works (Under the Hood)

Tunnelmole uses a persistent WebSocket connection between your machine and a host server (either the public Tunnelmole service or your own self-hosted instance). Incoming requests to your public URL are securely forwarded to your local server, and responses are sent back through the tunnel.

![How Tunnelmole Works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

---

## Frequently Asked Questions

### Is Tunnelmole really open source?

Yes! Both the client and server are open source. The client is MIT-licensed, and the server is AGPLv3. [See the code on GitHub](https://github.com/robbie-cahill/tunnelmole-client).

### Can I self-host Tunnelmole?

Absolutely. You can run your own Tunnelmole server for custom domains, privacy, or compliance. [Check out the server repo](https://github.com/robbie-cahill/tunnelmole-service).

### Does Tunnelmole hide my IP?

No. For abuse prevention, the hosted service adds your IP to the `X-Forwarded-For` header and the generated URL. If you self-host, you can modify this behavior.

### What are the limits on the free plan?

Tunnelmole enforces fair use limits on bandwidth and concurrent tunnels. For unlimited usage or custom domains, consider a paid plan or self-hosting.

---

## Get Started with Tunnelmole Today

If you want a real open source tunneling solution and couldn't find it on the ngrok github, Tunnelmole is your answer. It’s free, easy to use, and gives you the transparency and control you need.

- [Tunnelmole Client on GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Service (Self-host) on GitHub](https://github.com/robbie-cahill/tunnelmole-service)
- [Official Website & Docs](https://tunnelmole.com/?utm_source=ngrok-github-article)

---

## Summary & Next Steps

- **Tunnelmole is fully open source, on GitHub, and free for public URLs**
- **Self-hosting, custom domains, and Node.js integration are all supported**
- **Ready to get started? [Check out Tunnelmole on GitHub](https://github.com/robbie-cahill/tunnelmole-client) or [read the full docs](https://tunnelmole.com/?utm_source=ngrok-github-article).**


## Disclosures
- The author of this article is not affiliated with ngrok in any way, shape or form
- This article was written by Robbie Cahill, CEO of Tunnelmole Pty Ltd
- Features and limits shown may be subject to change, see the [tunnelmole website](https://tunnelmole.com) for the latest information

