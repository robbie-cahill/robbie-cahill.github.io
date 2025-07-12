---
title: "How to Give Your IoT Device a Public URL (for Any IoT Platform) with Tunnelmole"
description: "Learn how to expose your IoT device to the internet using Tunnelmole, an open source tunneling tool. Step-by-step guide for any IoT platform, with a Raspberry Pi example."
tags: ["iot platform", "iot", "platform", "opensource", "javascript", "typescript", "python", "programming", "coding", "raspberrypi", "tunneling", "networking", "webhooks", "nodejs", "linux", "automation", "cloud", "devops"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/04/iot-platform/"
slug: "iot-platform"
cover_image: "https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/tunnelmole-demo.gif"
published: true
---

# How to Give Your IoT Device a Public URL (for Any IoT Platform) with Tunnelmole

If you’re building or running an **IoT platform**—whether it’s a smart home dashboard, a Raspberry Pi, or a custom sensor—remote access is essential. But most IoT devices are hidden behind NAT, firewalls, or dynamic IPs, making them invisible to the outside world.

**In this guide, you’ll learn how to give your IoT device a public URL using Tunnelmole, an open source tunneling tool.** This works for any IoT platform, from a single device to a fleet of sensors. We’ll walk through a practical example with a Raspberry Pi, but these steps apply to any IoT device or platform that can run Node.js or a Linux binary.

---

## Why Your IoT Platform Needs a Public URL

Modern IoT platforms and devices often need to:

- Receive webhooks or push notifications from cloud services
- Be accessed remotely for monitoring, debugging, or control
- Integrate with third-party APIs or automation platforms (e.g., IFTTT, Zapier, Home Assistant)
- Share live sensor data or dashboards with collaborators

But by default, most IoT devices are only accessible on your local network. This makes remote access, cloud integration, and real-world testing difficult.

**A public URL solves this by making your IoT device accessible from anywhere—securely and on-demand. This is a must-have for any modern IoT platform.**

---

## Common Challenges: NAT, Firewalls, and Dynamic IPs

Before we dive in, let’s look at why exposing an IoT device or platform is tricky:

- **NAT (Network Address Translation):** Most home and office networks use NAT, which hides devices behind a single public IP.
- **Firewalls:** Block incoming connections by default.
- **Dynamic IPs:** Your public IP may change, breaking direct access.
- **ISP Restrictions:** Many ISPs block inbound traffic or charge for static IPs.

**Traditional solutions** (like port forwarding or VPNs) are complex, require router access, and can introduce security risks.

---

## What is Tunnelmole?

[Tunnelmole](https://tunnelmole.com/?utm_source=iot-platform-article) is an open source tunneling tool that gives your locally running HTTP(s) servers a public URL. It works on Linux, Mac, and Windows, and is perfect for IoT platforms and devices. It can work seamlessly through NAT, home and business routers and most firewalls. It works on almost any internet connected device - even a 4G/5G device that only has mobile internet will likely work fine with Tunnelmole.

**Key features:**

- **Open Source:** Inspect, modify, or self-host the code.
- **Free HTTPS URLs:** Instantly get a secure public URL for your IoT device or platform.
- **Custom Subdomains:** Use a memorable URL (paid or self-hosted).
- **Self-Hostable:** Run your own Tunnelmole server for full control.
- **Native Node.js Application:** Lightweight and easy to run on most IoT platforms.

---

## How Tunnelmole Works

Tunnelmole creates a secure tunnel from your IoT device or platform to a public server. When you run `tmole <port>`, it:

1. Opens a persistent WebSocket connection to the Tunnelmole service.
2. Assigns you a unique public HTTPS URL (e.g., `https://abc12.tunnelmole.net`).
3. Forwards all requests from that URL to your device’s local port.

![How Tunnelmole Works](https://raw.githubusercontent.com/robbie-cahill/tunnelmole-client/main/docs/img/how-tunnelmole-works.png)

**No router config, no firewall changes, no static IP required.**

---

## Step-by-Step: Expose Your IoT Device with Tunnelmole

### 1. Prerequisites

- Your IoT device or platform runs Linux (Raspberry Pi, BeagleBone, etc.), Mac, or Windows.
- You have a service running on a local port (e.g., a web dashboard, REST API, or MQTT bridge).
- Node.js 16.10+ (for npm install) or ability to run a Linux binary.

### 2. Install Tunnelmole

#### Option A: Install via npm (requires Node.js)

```bash
sudo npm install -g tunnelmole
```

#### Option B: Install precompiled binary (no Node.js required)

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

- The script auto-detects your OS (works on Raspberry Pi OS, Ubuntu, Debian, etc.).
- For Windows, [download tmole.exe](https://tunnelmole.com/downloads/tmole.exe) and add to your PATH.

### 3. Start Your IoT Device Service

Make sure your device is running the service you want to expose. For example, a web dashboard on port 8080:

```bash
python3 -m http.server 8080
```

Or a Node.js/Express server:

```bash
node app.js
```

Or a Home Assistant dashboard (default port 8123):

```bash
# Home Assistant usually runs as a service, but on Linux you can check with
sudo systemctl status home-assistant@homeassistant
```

### 4. Run Tunnelmole

In a new terminal on your IoT device or platform, run:

```bash
tmole 8080
```

Replace `8080` with the port your service is running on.

**Example output:**

```
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://abc12-ip-192-168-1-100.tunnelmole.net ⟶ http://localhost:8080
http://abc12-ip-192-168-1-100.tunnelmole.net ⟶ http://localhost:8080
```

### 5. Access Your Device Remotely

- Open the HTTPS URL in your browser from any device, anywhere.
- Share the URL with collaborators, or use it as a webhook endpoint.

---

## Example: Exposing a Raspberry Pi IoT Platform Dashboard

Let’s walk through a real-world example using a Raspberry Pi as an IoT device and platform.

### Scenario

You have a Raspberry Pi running a temperature sensor and serving a simple web dashboard on port 5000.

#### 1. Start the Dashboard

Suppose your dashboard is a Python Flask app:

```python
# app.py
from flask import Flask
import random

app = Flask(__name__)

@app.route("/")
def index():
    temp = 20 + random.random() * 5
    return f"<h1>Current Temperature: {temp:.2f}°C</h1>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

Start the app:

```bash
python3 app.py
```

#### 2. Install Tunnelmole

```bash
curl -O https://install.tunnelmole.com/xD345/install && sudo bash install
```

#### 3. Run Tunnelmole

```bash
tmole 5000 # Replace 5000 with the port number your service is running on
```

#### 4. Get Your Public URL

Tunnelmole will output something like:

```
https://xyz12-ip-10-0-0-42.tunnelmole.net ⟶ http://localhost:5000
```

#### 5. View the Dashboard Remotely

Open the HTTPS URL in your browser from your phone, laptop, or share it with a friend.

**Now your Raspberry Pi IoT device and platform dashboard is accessible from anywhere!**

---

## Security Considerations

Exposing your IoT device or platform to the public internet comes with risks. Here are some best practices:

- **Use HTTPS URLs** (Tunnelmole provides these by default).
- **Protect sensitive endpoints** with authentication.
- **Limit exposure:** Only tunnel the ports/services you need.
- **Monitor logs** for unexpected access.
- **Disable the tunnel when not in use.**

> **Note:** Tunnelmole adds an `X-Forwarded-For` header with your device’s IP for transparency. If you need to hide your origin IP, self-host Tunnelmole and modify the code.

---

## Advanced: Custom Subdomains and Self-Hosting

### Custom Subdomains

Want a memorable URL (e.g., `mydevice.tunnelmole.net`)?

- Run:  
  ```bash
  tmole 5000 as mydevice.tunnelmole.net
  ```
- **Note:** Custom subdomains require a paid subscription on the hosted service, or you can self-host for free.

### Self-Hosting Tunnelmole

For full control, privacy, or unlimited tunnels, you can run your own Tunnelmole server.

- [Tunnelmole Service GitHub Repo](https://github.com/robbie-cahill/tunnelmole-service/)
- Host on your own VPS, cloud, or even another Raspberry Pi.

---

## Troubleshooting & FAQ

### My IoT device is behind a strict firewall/NAT. Will Tunnelmole work?

Yes! Tunnelmole uses outbound WebSocket connections, so it works even when inbound connections are blocked.

### Can I use Tunnelmole for MQTT, SSH, or non-HTTP protocols?

Tunnelmole is designed for HTTP/HTTPS traffic. For other protocols, consider running a lightweight HTTP API or dashboard on your device or platform.

### Is Tunnelmole really open source?

Yes. Both the client and server are open source. See the [Tunnelmole Client](https://github.com/robbie-cahill/tunnelmole-client) and [Tunnelmole Service](https://github.com/robbie-cahill/tunnelmole-service) repos.

### Does Tunnelmole hide my IP?

No. The hosted service adds your IP to the public URL and in the `X-Forwarded-For` header for abuse prevention. You can self-host and modify this if needed.

### How do I stop the tunnel?

Just press `Ctrl+C` in the terminal running Tunnelmole.

---

## Summary & Next Steps

Giving your IoT device a public URL unlocks powerful new use cases: remote access, cloud integration, real-world testing, and more. Tunnelmole makes this process simple, secure, and open source—perfect for any IoT platform.

**Key takeaways:**

- Tunnelmole works on any IoT device or platform that can run Node.js or a Linux binary.
- No router config, no static IP, no firewall headaches.
- Free HTTPS URLs, with custom subdomains and self-hosting options.

**Ready to try it?**

- [Get started with Tunnelmole](https://tunnelmole.com/?utm_source=iot-platform-article)
- [View the Tunnelmole Client on GitHub](https://github.com/robbie-cahill/tunnelmole-client)
- [See the Tunnelmole Service for self-hosting](https://github.com/robbie-cahill/tunnelmole-service)

---

**Want more guides like this?**  
Share this article, star the Tunnelmole repo, or join the [Tunnelmole community](https://tunnelmole.com/?utm_source=iot-platform-article#community).

---