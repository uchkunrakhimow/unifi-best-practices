# UniFi Controller API - Professional Guide

A comprehensive, high-level reference for developers working with the Ubiquiti UniFi Controller API.

## Table of Contents

- [UniFi Controller API - Professional Guide](#unifi-controller-api---professional-guide)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [Core Concepts](#core-concepts)
  - [Quick Start](#quick-start)
  - [API Endpoint Reference](#api-endpoint-reference)
    - [Authentication](#authentication)
    - [Device Management](#device-management)
    - [Client Management](#client-management)
    - [Internet Access Control](#internet-access-control)
    - [Network Configuration](#network-configuration)
    - [Site Management](#site-management)
    - [Statistics \& Monitoring](#statistics--monitoring)
    - [Pagination and Filtering](#pagination-and-filtering)
  - [WebSocket API](#websocket-api)
    - [Connection Establishment](#connection-establishment)
    - [Common Event Types](#common-event-types)
    - [WebSocket Event Handling Best Practices](#websocket-event-handling-best-practices)

## Overview

The UniFi Controller API allows programmatic control over your entire UniFi network, including devices, clients, and configuration settings. This document provides a structured guide to the most common and useful API endpoints.

## Core Concepts

- **Authentication**: Session-based with cookie persistence
- **Site Structure**: Most endpoints require a site identifier (`default` is the primary site)
- **Response Format**: JSON with standard structure `{ "meta": { "rc": "ok" }, "data": [...] }`
- **MAC Addresses**: Use lowercase, typically without separators (e.g., `00112233aabb`)

## Quick Start

```javascript
// Example authentication flow
const response = await axios.post(
  "https://unifi.example.com/api/login",
  {
    username: "admin",
    password: "password",
    remember: true,
  },
  {
    withCredentials: true,
    httpsAgent: new https.Agent({ rejectUnauthorized: false }), // For self-signed certs
  }
);

// The session cookie is now stored in axios instance
```

## API Endpoint Reference

### Authentication

| Operation      | Method | Endpoint      | Description                     |
| -------------- | ------ | ------------- | ------------------------------- |
| Login          | POST   | `/api/login`  | Establish authenticated session |
| Verify Session | GET    | `/api/self`   | Check if session is valid       |
| Logout         | POST   | `/api/logout` | End current session             |

**Login Example Body:**

```json
{
  "username": "admin",
  "password": "your_password",
  "remember": true
}
```

### Device Management

| Operation        | Method | Endpoint                                | Description              |
| ---------------- | ------ | --------------------------------------- | ------------------------ |
| List All Devices | GET    | `/api/s/{site}/stat/device`             | Get all network devices  |
| Device Details   | GET    | `/api/s/{site}/stat/device/{mac}`       | Get specific device info |
| Device Stats     | GET    | `/api/s/{site}/stat/device/{mac}/stats` | Get performance metrics  |
| Restart Device   | POST   | `/api/s/{site}/cmd/devmgr`              | Restart specific device  |
| Upgrade Firmware | POST   | `/api/s/{site}/cmd/devmgr`              | Upgrade device firmware  |

**Restart Device Example Body:**

```json
{
  "cmd": "restart",
  "mac": "00:11:22:33:44:55"
}
```

### Client Management

| Operation         | Method | Endpoint                        | Description                        |
| ----------------- | ------ | ------------------------------- | ---------------------------------- |
| Active Clients    | GET    | `/api/s/{site}/stat/sta`        | List all connected clients         |
| All Clients       | GET    | `/api/s/{site}/stat/alluser`    | List all clients (inc. historical) |
| Client Details    | GET    | `/api/s/{site}/stat/user/{mac}` | Get specific client details        |
| Block Client      | POST   | `/api/s/{site}/cmd/stamgr`      | Block network access               |
| Unblock Client    | POST   | `/api/s/{site}/cmd/stamgr`      | Restore network access             |
| Disconnect Client | POST   | `/api/s/{site}/cmd/stamgr`      | Force client disconnection         |

**Block Client Example Body:**

```json
{
  "cmd": "block-sta",
  "mac": "aa:bb:cc:dd:ee:ff"
}
```

### Internet Access Control

| Operation        | Method | Endpoint                              | Description                   |
| ---------------- | ------ | ------------------------------------- | ----------------------------- |
| Set Access       | POST   | `/api/s/{site}/rest/user/{client_id}` | Allow/deny internet access    |
| Bandwidth Limits | POST   | `/api/s/{site}/rest/trafficrule`      | Create client bandwidth rules |

**Control Internet Access Example Body:**

```json
{
  "use_fixedip": true,
  "network_access": "deny", // or "allow"
  "fixedip": "192.168.1.100", // client's current IP
  "usergroup_id": "existing-group-id"
}
```

### Network Configuration

| Operation      | Method | Endpoint                              | Description                 |
| -------------- | ------ | ------------------------------------- | --------------------------- |
| List Networks  | GET    | `/api/s/{site}/rest/networkconf`      | Get all networks            |
| Create Network | POST   | `/api/s/{site}/rest/networkconf`      | Create new wireless network |
| Update Network | PUT    | `/api/s/{site}/rest/networkconf/{id}` | Modify existing network     |
| Delete Network | DELETE | `/api/s/{site}/rest/networkconf/{id}` | Remove network              |

**Create WiFi Network Example Body:**

```json
{
  "name": "Guest Network",
  "x_passphrase": "guestpassword",
  "security": "wpapsk",
  "wpa_mode": "wpa2",
  "enabled": true,
  "is_guest": true,
  "vlan_enabled": false
}
```

### Site Management

| Operation   | Method | Endpoint                     | Description              |
| ----------- | ------ | ---------------------------- | ------------------------ |
| List Sites  | GET    | `/api/self/sites`            | Get all accessible sites |
| Site Health | GET    | `/api/s/{site}/stat/health`  | Get site health metrics  |
| Create Site | POST   | `/api/s/default/cmd/sitemgr` | Create new site          |
| Delete Site | POST   | `/api/s/default/cmd/sitemgr` | Remove existing site     |

**Create Site Example Body:**

```json
{
  "cmd": "add-site",
  "name": "New Office",
  "desc": "Branch office location"
}
```

### Statistics & Monitoring

| Operation        | Method | Endpoint                               | Description                      |
| ---------------- | ------ | -------------------------------------- | -------------------------------- |
| DPI Stats        | GET    | `/api/s/{site}/stat/dpi`               | Get Deep Packet Inspection stats |
| Client DPI       | GET    | `/api/s/{site}/stat/stadpi`            | Get client-specific DPI stats    |
| Historical Stats | GET    | `/api/s/{site}/stat/report/daily.site` | Get site history metrics         |
| Gateway Stats    | GET    | `/api/s/{site}/stat/gateway`           | Get USG performance metrics      |

### Pagination and Filtering

| Parameter | Description                                       | Example                                        |
| --------- | ------------------------------------------------- | ---------------------------------------------- |
| `_limit`  | Maximum number of items to return                 | `/api/s/{site}/stat/sta?_limit=50`             |
| `_start`  | Index to start from (for offset-based pagination) | `/api/s/{site}/stat/sta?_start=50`             |
| `mac`     | Filter by MAC address                             | `/api/s/{site}/stat/sta?mac=001122334455`      |
| `ip`      | Filter by IP address                              | `/api/s/{site}/stat/sta?ip=192.168.1.100`      |
| `within`  | Timeframe for historical data in seconds          | `/api/s/{site}/stat/sta?within=86400`          |
| `attrs`   | Comma-separated list of attributes to include     | `/api/s/{site}/stat/sta?attrs=mac,ip,hostname` |

**Example Filtered Request:**

```javascript
// Fetch the last 10 active clients with only specific attributes
const response = await api.get("/api/s/default/stat/sta", {
  params: {
    _limit: 10,
    _sort: "-last_seen", // Sort by most recently seen
    attrs: "mac,hostname,ip,signal,tx_bytes,rx_bytes",
  },
});
```

## WebSocket API

UniFi Controller also provides a WebSocket interface for real-time event monitoring. This allows you to receive immediate notifications about network changes without polling.

### Connection Establishment

```javascript
const WebSocket = require("ws");
const axios = require("axios");
const https = require("https");
const cookieJar = {};

// First authenticate via the REST API
const api = axios.create({
  baseURL: "https://unifi.example.com",
  withCredentials: true,
  httpsAgent: new https.Agent({ rejectUnauthorized: false }),
});

// Login to get the authentication cookie
await api
  .post("/api/login", {
    username: "admin",
    password: "password",
  })
  .then((response) => {
    // Extract and store cookies for WebSocket connection
    const cookies = response.headers["set-cookie"];
    if (cookies) {
      cookies.forEach((cookie) => {
        const [key, value] = cookie.split(";")[0].split("=");
        cookieJar[key] = value;
      });
    }
  });

// Format cookies for WebSocket connection
const cookieString = Object.entries(cookieJar)
  .map(([key, value]) => `${key}=${value}`)
  .join("; ");

// Connect to WebSocket
const ws = new WebSocket("wss://unifi.example.com/wss/s/default/events", {
  headers: {
    Cookie: cookieString,
  },
  rejectUnauthorized: false, // For self-signed certificates
});

// Handle WebSocket events
ws.on("open", () => {
  console.log("WebSocket connection established");
});

ws.on("message", (data) => {
  const event = JSON.parse(data);
  console.log("Received event:", event);

  // Handle different event types
  if (event.data && event.data.meta && event.data.meta.message) {
    const msgType = event.data.meta.message;
    switch (msgType) {
      case "sta:sync":
        console.log("Client update:", event.data.data);
        break;
      case "device:sync":
        console.log("Device update:", event.data.data);
        break;
      // Add handlers for other events
    }
  }
});

ws.on("error", (error) => {
  console.error("WebSocket error:", error);
});

ws.on("close", () => {
  console.log("WebSocket connection closed");
});
```

### Common Event Types

| Event Type       | Description                | When Triggered                      |
| ---------------- | -------------------------- | ----------------------------------- |
| `sta:sync`       | Client status updates      | Client connects/disconnects/updates |
| `device:sync`    | Device status updates      | Device status changes               |
| `alarm`          | System alarms and warnings | New alert triggered                 |
| `speedtest:done` | WAN speed test completed   | After running a speed test          |
| `backup:done`    | Backup creation completed  | After creating a backup             |
| `evt`            | Various system events      | Various actions on the system       |
| `notification`   | System notifications       | New notification                    |

### WebSocket Event Handling Best Practices

1. **Automatic Reconnection**

```javascript
function connectWebSocket() {
  const ws = new WebSocket("wss://unifi.example.com/wss/s/default/events", {
    headers: { Cookie: cookieString },
    rejectUnauthorized: false,
  });

  // Set timeout for connection attempts
  const connectionTimeout = setTimeout(() => {
    if (ws.readyState !== WebSocket.OPEN) {
      ws.terminate();
      console.log("Connection attempt timed out, retrying...");
      setTimeout(connectWebSocket, 5000); // Try again in 5 seconds
    }
  }, 10000);

  ws.on("open", () => {
    clearTimeout(connectionTimeout);
    console.log("WebSocket connection established");

    // Set up a keepalive ping
    const pingInterval = setInterval(() => {
      if (ws.readyState === WebSocket.OPEN) {
        ws.ping();
      } else {
        clearInterval(pingInterval);
      }
    }, 30000);
  });

  ws.on("close", () => {
    console.log("Connection closed, attempting to reconnect...");
    setTimeout(connectWebSocket, 5000);
  });

  // Handle other events as before
}

// Initial connection
connectWebSocket();
```

2. **Event Filtering and Processing**

```javascript
// Define callback handlers for different event types
const eventHandlers = {
  "sta:sync": (data) => {
    // Process client updates
    const clientMac = data.mac;
    const connectionState = data.state ? "connected" : "disconnected";
    console.log(`Client ${clientMac} is now ${connectionState}`);
  },
  "device:sync": (data) => {
    // Process device updates
    console.log(`Device ${data.name} (${data.mac}) updated: ${data.state}`);
  },
  alarm: (data) => {
    // Process alarms
    console.log(`ALARM: ${data.msg} (Key: ${data.key}, Time: ${data.time})`);
  },
};

// In the message handler
ws.on("message", (rawData) => {
  try {
    const event = JSON.parse(rawData);

    if (event.data && event.data.meta && event.data.meta.message) {
      const msgType = event.data.meta.message;

      // Call the appropriate handler if it exists
      if (eventHandlers[msgType]) {
        eventHandlers[msgType](event.data.data);
      } else {
        console.log(`Unhandled event type: ${msgType}`);
      }
    }
  } catch (error) {
    console.error("Error processing WebSocket message:", error);
  }
});
```
