# Network Foundations – Understanding Modern Network Communication

**Name:** Murali Manohara
**Role:** AI Engineer Trainee

## Introduction

This project explains how a Task Management Web Application communicates over a network. The app lets users log in, create tasks, get real-time updates, and receive background notifications. It covers TCP/IP, protocol selection, latency, secure communication, VPN, distributed systems, and event-driven architecture.

## Task 1: TCP/IP Communication Flow

When a user opens the application (e.g. `https://taskapp.example.com`), the following happens:

- The browser resolves the domain name to a server IP address using DNS.
- The request leaves the user's device and passes through the local router.
- The router forwards the traffic across the Internet, possibly through several intermediate routers.
- The application server receives and processes the request.
- The server sends a response back through the same path, and the browser renders it (HTML, CSS, JS, images).

TCP/IP is the set of protocols that makes this possible.

- **IP (Internet Protocol)** handles addressing and routing – it makes sure data reaches the correct destination.
- **TCP (Transmission Control Protocol)** provides reliable, ordered delivery. It establishes a connection before data is exchanged, and can retransmit lost packets. It also handles error detection and flow/congestion control.

In short: the browser sends a request from the device, through the router and the Internet, to the server. IP addresses and routes the data, while TCP makes sure it arrives reliably and in order. The server processes the request and returns a response the same way.

## Task 2: Choosing the Right Communication Protocol

Different parts of the application have different needs, so different protocols are used:

| Scenario | Protocol |
|---|---|
| Opening the website securely | HTTPS |
| Live task status updates | WebSocket |
| Internal communication between backend services | gRPC |
| Downloading a public document | HTTP |

- **HTTPS** is used for the main site because login credentials, user data, and task information should be encrypted in transit.
- **WebSocket** is used for live updates because it keeps a persistent connection open, so the server can push a status change (e.g. "Processing" to "Completed") to the browser immediately, instead of the browser repeatedly polling with new requests.
- **gRPC** is used between backend services (e.g. User Service calling Task Service) because it is efficient for service-to-service calls and commonly runs over HTTP/2.
- **HTTP** is used for downloading a public document, since it is a simple request-response transfer and the document itself isn't sensitive. In a real production system HTTPS would still be preferred even here, but of the four options given, HTTP fits this case best.

## Task 3: Network Performance

The application takes 5 seconds to display task updates. This is mainly a **latency** issue, not a throughput issue (assuming the update itself is small).

Latency is the delay before data reaches its destination and a response comes back — the 5-second wait fits this. Throughput is how much data can be transferred in a given time (e.g. MB/s), which matters more for large transfers, not a small status update.

Two ways to improve this:

1. **Reduce unnecessary round trips** — replace repeated "is it done yet?" polling with a persistent WebSocket connection so the server can push the update as soon as it happens.
2. **Reduce the amount of data sent** — instead of sending the full task object every time, send only what changed (e.g. just the task ID and new status).

Note: a 5-second delay isn't automatically a network problem — it could also come from server processing or database queries, but since the question asks about network performance specifically, latency is the appropriate answer here.

## Task 4: Secure Communication

The application should use HTTPS (HTTP over TLS) instead of plain HTTP, because it handles sensitive data such as login credentials and task information. TLS provides three things:

- **Confidentiality** — encrypts traffic so it can't easily be read if intercepted.
- **Integrity** — allows detection if data was modified in transit.
- **Authentication** — certificates let the browser verify it's talking to the real server, protecting against impersonation.

A VPN, on the other hand, is used when a company needs to give employees or systems secure access to private internal resources over an untrusted network (e.g. someone working from home connecting to an internal task server or database). HTTPS protects one application's connection; a VPN protects access to an entire private network.

## Task 5: Modern Application Architecture

The application is split into a User Service, Task Service, and Notification Service. This makes it a **distributed system**, since these are separate services that communicate with each other over a network to get work done: the User Service handles login/user info, the Task Service handles creating and updating tasks, and the Notification Service handles sending emails.

The notification process is also **event-driven**. When a task is created, the Task Service can emit a `TaskCreated` event. The Notification Service listens for this event and sends an email in response, without needing to be called directly every time a task is created. This decouples task creation from the notification logic.

## Conclusion

This project covers how the Task Management Web Application communicates end-to-end. IP and TCP move data reliably between the browser and server. HTTPS/TLS secures that communication, while WebSocket and gRPC are used where real-time updates or internal service calls are needed. The 5-second update delay is a latency concern that can be improved by cutting round trips and payload size. Finally, since the app is built from independent services that react to events like `TaskCreated`, it is both a distributed system and an event-driven one.

## Architecture Diagram

The diagram below summarizes the complete request/response flow, protocol choices, and service architecture described above.

![Architecture Diagram](images/architecture-diagram.png)
