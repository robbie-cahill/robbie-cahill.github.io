---
title: "How to Get a Public URL for Your Linux Virtual Machine with Tunnelmole"
description: "Expose your Linux virtual machine to the internet with a secure public URL using Tunnelmole. Step-by-step guide for developers and sysadmins."
tags: ["linux", "linuxvirtualmachine", "vm", "virtualmachine", "opensource", "javascript", "typescript", "python", "programming", "coding", "devops", "cloud", "tunneling", "webhooks", "nodejs", "selfhost", "networking"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/04/how-to-get-a-public-url-for-your-linux-virtual-machine-with-tunnelmole/"
slug: "linux-virtual-machine"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Get a Public URL for Your Linux Virtual Machine with Tunnelmole

If you’re running a **Linux virtual machine** (VM) and need to expose a web server, API, or any local service to the internet, you’ve probably run into the pain of network configuration, NAT, and firewall rules. Whether you’re developing, testing webhooks, or sharing a demo, getting a public URL for your Linux virtual machine can be a challenge—especially if you don’t control the network or want to avoid complex port forwarding.

**Tunnelmole** is an open source tunneling tool that makes it easy to get a secure, public URL for any service running on your Linux VM. In this guide, you’ll learn how to use Tunnelmole to expose your Linux virtual machine to the world in minutes, with no cloud deploys, no firewall headaches, and no vendor lock-in.

---

## Why Expose a Linux Virtual Machine to the Internet?

Linux virtual machines are everywhere: on cloud providers (AWS EC2, GCP Compute Engine, Azure VMs), on-premises servers, or even your local workstation via VirtualBox, VMware, or KVM. But by default, services running inside a VM are only accessible from the host or private network.

**Common use cases for exposing a Linux VM:**

- Testing webhooks from SaaS providers (Shopify, Stripe, GitHub, etc.)
- Sharing a development site or API with teammates or clients
- Mobile device testing (accessing your dev server from your phone)
- IoT device integration and remote debugging
- Running demos or proof-of-concept apps without deploying to production

But opening ports on your VM’s network is often blocked by NAT, firewalls, or cloud security groups. That’s where a tunneling tool like Tunnelmole comes in.

---

## What is Tunnelmole?

Tunnelmole is an **open source tunneling tool** that gives your locally running HTTP(s) servers a public URL. It’s designed for developers and sysadmins who need a quick, secure way to expose services running on their machine or VM to the internet.

**Key features:**

- **Open Source:** MIT/AGPLv3 licensed, self-hostable, and auditable.
- **Free public URLs:** Instantly get a public HTTPS URL for your service.
- **Custom subdomains:** Use a memorable subdomain (paid or self-hosted).
- **Native Node.js app:** Works on Linux, Mac, and Windows.
- **No login required:** Start tunneling in seconds.

---

## How Tunnelmole Works with a Linux Virtual Machine

Tunnelmole creates a secure tunnel from your Linux VM to a public endpoint. Here’s how it works:

1. **You run Tunnelmole on your Linux VM**, specifying the port your service is running on (e.g., 8080).
2. **Tunnelmole connects to the Tunnelmole service** (hosted or self-hosted) and requests a public URL.
3. **Incoming requests to the public URL** are securely forwarded to your VM’s local service via a persistent WebSocket tunnel.
4. **You share the public URL** with anyone—no need to open firewall ports or configure NAT.

![How Tunnelmole Works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

---

## Step-by-Step: Expose Your Linux Virtual Machine with Tunnelmole

### 1. Prerequisites

- A Linux virtual machine (cloud or local)
- Node.js 16.10+ (for npm install) **or** ability to run a precompiled binary
- A service running on your VM (e.g., a web server on port 8080)

### 2. Install Tunnelmole on Your Linux VM

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

### 3. Start Your Service on the Linux VM

For example, if you’re running a Node.js app:

```bash
node app.js
# or for Python
python3 -m http.server 8080
```

Make sure you know the port your service is listening on (e.g., 8080).

---

### 4. Run Tunnelmole to Get a Public URL

In your VM’s terminal, run:

```bash
tmole 8080
```

Replace `8080` with your service’s port.

**Example output:**

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:8080
http://cqcu2t-ip-49-185-26-79.tunnelmole.net ⟶ http://localhost:8080
```

You now have a public HTTPS URL that forwards to your Linux VM’s local service!

---

### 5. Test Your Public URL

- Open the HTTPS URL in your browser (from any device).
- You should see your service running from inside your Linux virtual machine.
- Share the URL with teammates, webhook providers, or test from your phone.

---

## Advanced: Custom Subdomains and Self-Hosting

### Use a Custom Subdomain

Want a memorable, stable URL? Use a custom subdomain:

```bash
tmole 8080 as myvm.tunnelmole.net
```

- **Hosted service:** Custom subdomains require a paid subscription.
- **Self-hosted:** You can run your own Tunnelmole service and use any subdomain you control.

[Learn more about custom subdomains](https://dashboard.tunnelmole.com/?utm_source=tunnelmoleClientGithub)

### Self-Host Tunnelmole

Tunnelmole is fully open source. You can run your own Tunnelmole service for maximum control and privacy.

- [Tunnelmole Client (MIT)](https://github.com/robbie-cahill/tunnelmole-client)
- [Tunnelmole Service (AGPLv3)](https://github.com/robbie-cahill/tunnelmole-service)

---

## Example: Exposing a Flask App on a Linux VM

Let’s walk through a real-world example: exposing a Python Flask app running on a Linux virtual machine.

### 1. Start the Flask App

```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Linux VM!"

if __name__ == "__main__":
    app.run(port=5000)
```

Run the app:

```bash
python3 app.py
```

### 2. Start Tunnelmole

```bash
tmole 5000
```

Tunnelmole will output a public URL like:

```
https://abc12-ip-203-0-113-42.tunnelmole.net ⟶ http://localhost:5000
```

### 3. Access from Anywhere

Open the HTTPS URL in your browser—your Flask app is now live on the internet, running from your Linux virtual machine!

---

## Use Cases: Why Developers Love Tunnelmole for Linux Virtual Machines

- **Webhook development:** Receive webhooks from Stripe, GitHub, or any SaaS directly on your VM.
- **Remote demos:** Share your in-progress app with clients or teammates without deploying.
- **IoT and device testing:** Expose device APIs running on a VM to the cloud.
- **Mobile testing:** Access your dev server from your phone or tablet.
- **CI/CD pipelines:** Test integrations that require a public callback URL.

---

> Tunnelmole stands out for its open source license, self-hosting capability, and ease of use on Linux virtual machines.

---

## Security Considerations

- **Public URLs are accessible internet-wide.** Don’t expose sensitive services without authentication.
- **HTTPS by default:** Tunnelmole provides secure URLs.

---

## Troubleshooting: Common Linux VM Networking Issues

- **Firewall blocks:** If your VM has a local firewall (e.g., `ufw`), ensure the port is open locally.
- **No Node.js?** Use the precompiled binary install method.
- **No outbound internet?** Tunnelmole requires outbound WebSocket access to the Tunnelmole service.
- **Cloud VMs:** Works on AWS, GCP, Azure, DigitalOcean, and more.

---

## Frequently Asked Questions

### Is Tunnelmole free for Linux virtual machines?

Yes! You can use Tunnelmole’s hosted service to get free public URLs for your Linux VM. Custom subdomains require a paid plan or self-hosting.

### Can I use Tunnelmole on any Linux distribution?

Tunnelmole works on all major Linux distributions (Ubuntu, Debian, CentOS, Fedora, Arch, etc.) as long as you have Node.js or can run the precompiled binary.

### Does Tunnelmole work with Docker containers inside a Linux VM?

Absolutely. Run Tunnelmole on the VM and point it to the Docker container’s exposed port.

### Is Tunnelmole safe to use in production?

Tunnelmole is designed for development and testing. For production, consider security best practices and self-hosting if needed.

---

## Summary: Instantly Expose Your Linux Virtual Machine

Getting a public URL for your Linux virtual machine doesn’t have to be hard. With Tunnelmole, you can expose any service running on your VM to the internet in seconds—no port forwarding, no cloud deploys, no hassle.

- **Open source, free, and self-hostable**
- **Works on any Linux VM—cloud or local**
- **Perfect for webhooks, demos, and remote testing**

Ready to try it? Install Tunnelmole on your Linux virtual machine and get your public URL today.

---

**Try Tunnelmole now and expose your Linux virtual machine to the world:**

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
tmole 8080
```

[Learn more and get started](https://tunnelmole.com/?utm_source=linux-vm-article)
