---
title: "Remotely Access an IoT Device Instantly with Tunnelmole"
description: "Learn how to instantly get remote access to any IoT device, like a Raspberry Pi, with a secure public URL. No port forwarding or complex setup required."
tags: ["remote access to iot device", "iot", "publicurl", "server", "opensource", "javascript", "typescript", "python", "programming", "coding", "raspberrypi", "expressjs"]
canonical_url: "https://softwareengineeringstandard.com/2025/07/12/remote-access-to-iot-device/"
slug: "remote-access-to-iot-device"
cover_image: "https://softwareengineeringstandard.com/wp-content/uploads/2025/07/example-webhooks-app.webp"
published: true
---

## Remotely Access an IoT Device Instantly with Tunnelmole

You've just built an amazing Internet of Things (IoT) project. Maybe it's a home automation system on a Raspberry Pi, a weather station powered by an ESP32, or a custom web dashboard running on a single-board computer. It works perfectly on your local network, but now you face a common challenge: how do you get **remote access to your IoT device** from anywhere in the world?

Traditionally, this problem required navigating a maze of complex solutions like port forwarding, dealing with dynamic IP addresses, setting up Dynamic DNS (DDNS), or configuring a VPN. These methods are often complicated, insecure if not done correctly, and may not even work if your internet provider uses Carrier-Grade NAT (CGNAT).

This is where Tunnelmole comes in. It's a simple, open-source tool that provides a secure, public URL for your local IoT device in seconds, with just a single command. This guide will walk you through why you need remote access for your IoT projects and provide a step-by-step example of how to achieve it using Tunnelmole and a simple Express.js application on a Raspberry Pi.

## What is an IoT Device and Why Do You Need Remote Access?

The Internet of Things (IoT) refers to the vast network of physical devices embedded with sensors, software, and other technologies that connect and exchange data over the internet. These can range from consumer gadgets like smart speakers to industrial machinery. For hobbyists and developers, popular platforms include:

*   **Raspberry Pi:** A versatile, credit-card-sized computer capable of running a full Linux OS.
*   **Arduino:** A microcontroller board perfect for controlling simple electronics.
*   **ESP32/ESP8266:** Low-cost, low-power microcontrollers with built-in Wi-Fi, ideal for sensor-based projects.

So, why is remote access to these devices so crucial?

*   **Monitoring and Data Collection:** View real-time sensor data from your weather station, check the status of your 3D printer, or monitor your home security camera feed while you're away.
*   **Control and Automation:** Remotely control actuators, such as turning lights on and off, adjusting a thermostat, or opening a garage door.
*   **Web-Based Dashboards:** Access a web interface to visualize data and manage your device without being on the same Wi-Fi network.
*   **Webhook Integrations:** Connect your IoT device to third-party services like IFTTT, Zapier, or home automation platforms (like Home Assistant) that use webhooks to send commands and data.
*   **Development and Demonstration:** Share your work-in-progress with collaborators or clients without a complicated deployment process.

## The Challenge: Bridging Local and Public Networks

When your IoT device is running a web server, it's typically only accessible via its local IP address (e.g., `192.168.1.100`) or `localhost`. This address is private to your network. The public internet has no direct way to reach it due to Network Address Translation (NAT), a technology used in virtually all routers to manage traffic between your private network and the public internet.

To make matters more complex, many internet and mobile providers now use Carrier-Grade NAT (CGNAT). With CGNAT, you don't even get a unique public IP address for your router, making traditional methods like port forwarding completely impossible.

This is the problem that tunneling tools are designed to solve.

## Instant Remote Access for your IoT device with Tunnelmole

Tunnelmole is a lightweight and open-source tool that creates a secure tunnel from your IoT device running on your local network to a public server. It then gives you a unique, publicly accessible URL that forwards all traffic through this tunnel to your device.

Here's how it solves the remote access problem:

1.  Your IoT device doesn't need a public IP address.
2.  You almost never need to configure your router, firewall, or DDNS.
3.  It bypasses CGNAT, NAT, all but the most locked down firewalls and other network restrictions effortlessly.
4.  It provides a secure HTTPS URL, encrypting the traffic from the end-user to the Tunnelmole public endpoint.

This makes it an ideal solution for providing instant and secure remote access to any IoT device.

## Practical Example: A Raspberry Pi Web Server for Sensor Data

Let's walk through a real-world scenario. Imagine we have a Raspberry Pi connected to a sensor, and we want to display its data on a web page that we can access from our phone or laptop anywhere.

### Step 1: Set Up a Node.js Web Server on Your Raspberry Pi

First, we need a web application running on the Pi to serve our data. We'll use Node.js and Express.js to create a minimal web server.

Assuming you have Node.js and `npm` installed on your Raspberry Pi OS:

1.  Create a project directory and initialize it:
    ```bash
    mkdir iot-server
    cd iot-server
    npm init -y
    ```
2.  Install Express.js:
    ```bash
    npm install express
    ```
3.  Create a file named `server.js` and add the following code. This app simulates reading sensor data and displays it on a simple HTML page.

    ```javascript
    // server.js
    const express = require('express');
    const app = express();
    const port = 3000;

    // A simple function to simulate reading from a sensor
    const getSensorData = () => {
        return {
            temperature: (20 + Math.random() * 5).toFixed(2), // Simulate 20-25 degrees Celsius
            humidity: (40 + Math.random() * 10).toFixed(2),    // Simulate 40-50% humidity
            timestamp: new Date().toISOString()
        };
    };

    app.get('/', (req, res) => {
        const data = getSensorData();
        res.send(`
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="refresh" content="5">
                <title>IoT Sensor Dashboard</title>
                <style>
                    body { 
                        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        flex-direction: column;
                        height: 100vh;
                        margin: 0;
                        background-color: #f4f6f8;
                        color: #333;
                    }
                    .container { 
                        text-align: center; 
                        padding: 2.5rem; 
                        border-radius: 12px; 
                        box-shadow: 0 8px 25px rgba(0,0,0,0.1); 
                        background-color: white; 
                        border: 1px solid #e0e0e0;
                    }
                    h1 { color: #1a237e; }
                    p { font-size: 1.5rem; color: #3949ab; margin: 0.5rem 0; }
                    .timestamp { font-size: 0.8rem; color: #777; margin-top: 1rem; }
                </style>
            </head>
            <body>
                <div class="container">
                    <h1>Live IoT Sensor Data</h1>
                    <p>Temperature: ${data.temperature} &deg;C</p>
                    <p>Humidity: ${data.humidity} %</p>
                    <p class="timestamp">Last updated: ${data.timestamp}</p>
                </div>
            </body>
            </html>
        `);
    });

    app.listen(port, () => {
        console.log(`IoT server running at http://localhost:${port}`);
    });
    ```
4.  Run your server:
    ```bash
    node server.js
    ```
    Your IoT dashboard is now running and accessible at `http://localhost:3000` on the Raspberry Pi itself.

### Step 2: Install Tunnelmole on Your Raspberry Pi

To install Tunnelmole on Raspberry pi, you first need to install NodeJS 16+.

Run the following to install Tunnelmole:
```bash
sudo npm install -g tunnelmole
```

### Step 3: Get a Public URL for Your IoT Device

With your web server running on port 3000 and Tunnelmole installed, getting remote access is as simple as running one command in the new terminal:

```bash
tmole 3000
```

Tunnelmole will start and display output similar to this:

```
$ tmole 3000
Your Tunnelmole Public URLs are below and are accessible internet wide. Always use HTTPs for the best security

https://k29s3d-ip-192-168-1-10.tunnelmole.net ⟶ http://localhost:3000
http://k29s3d-ip-192-168-1-10.tunnelmole.net ⟶ http://localhost:3000
```
That's it! You now have a public HTTPS URL. You can open this URL in a web browser on your phone, laptop, or any other device connected to the internet, and you will see your IoT sensor dashboard. The page will even auto-refresh every 5 seconds with new "sensor" data.

## Key Benefits of Using Tunnelmole for IoT Remote Access

Using Tunnelmole for your IoT projects offers several distinct advantages:

*   **Simplicity:** It replaces complex network configurations with a single, easy-to-remember command. This is ideal for developers who want to focus on their application, not network administration.
*   **Works From Anywhere:** It reliably bypasses network restrictions, including firewalls and CGNAT, which is particularly useful for IoT devices on cellular or other restricted networks.
*   **Enhanced Security:** By providing an HTTPS URL, Tunnelmole helps encrypt traffic. Your device's local port is not directly exposed to the internet, reducing its attack surface.
*   **Open Source and Self-Hostable:** For maximum security and control, advanced users can inspect the source code and even self-host the Tunnelmole service on their own server, ensuring data never passes through third-party infrastructure.

## Conclusion: Unleash Your IoT Project's Potential

The ability to access and control your IoT devices remotely opens up a world of possibilities, transforming local projects into globally accessible applications. While traditional methods of achieving this are often fraught with complexity and security concerns, tools like Tunnelmole democratize remote access, making it simple, fast, and secure.

Whether you're a hobbyist looking to share your latest creation, a student building a project for a class, or a developer prototyping a commercial product, having a public URL in seconds is a game-changer. It allows for rapid testing, seamless integration with cloud services, and easy demonstration to anyone, anywhere.

Ready to give your IoT project a global reach? Get started with Tunnelmole and bring your device online today.