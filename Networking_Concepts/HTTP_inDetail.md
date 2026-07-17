# 🌐 HTTP In Detail — TryHackMe Writeup

![Platform](https://img.shields.io/badge/Platform-TryHackMe-red?style=for-the-badge&logo=tryhackme)
![Category](https://img.shields.io/badge/Category-Web%20Fundamentals-blue?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-green?style=for-the-badge)

> **Room:** [HTTP In Detail](https://tryhackme.com/room/httpindetail) — TryHackMe
> **Author of writeup:** *Your Name Here*
> **Purpose:** Understanding how HTTP works under the hood — headers, methods, and status codes — which is foundational knowledge for web application penetration testing.

---

## 📑 Table of Contents

- [Introduction](#-introduction)
- [HTTP Headers](#-http-headers)
  - [Common Request Headers](#common-request-headers)
  - [Common Response Headers](#common-response-headers)
- [HTTP Methods](#-http-methods)
- [HTTP Status Codes](#-http-status-codes)
- [Quick Revision Cheat Sheet](#-quick-revision-cheat-sheet)
- [Key Takeaways](#-key-takeaways)

---

## 📝 Introduction

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the web. Every time a browser talks to a server, it exchanges **requests** and **responses**, each carrying **headers**, a **method**, and a **status code**. Understanding these three components in depth is essential for anyone getting into **web application security**, since most vulnerabilities (IDOR, broken auth, improper access control, etc.) are found by manipulating exactly these pieces.

This writeup documents my notes while completing the **HTTP In Detail** room on TryHackMe.

---

## 📦 HTTP Headers

Headers are additional bits of data sent along with an HTTP request or response. While no header is *strictly* required to make a request, without them, a website won't display or function correctly.

### Common Request Headers

*(Sent from the client/browser → server)*

| Header | Description |
|---|---|
| **Host** | Specifies which website is being requested when a server hosts multiple sites. Without it, the client gets the server's default site. |
| **User-Agent** | Identifies the browser and version making the request, so the server can format the response appropriately (some HTML/CSS/JS features are browser-specific). |
| **Content-Length** | Tells the server how much data to expect in the body of the request (e.g. when submitting a form), ensuring no data is missing. |
| **Accept-Encoding** | Informs the server which compression methods (e.g. gzip) the browser supports, allowing data to be transmitted more efficiently. |
| **Cookie** | Sends stored client-side data (like session IDs) back to the server on every request. |

### Common Response Headers

*(Sent from the server → client)*

| Header | Description |
|---|---|
| **Set-Cookie** | Instructs the browser to store a cookie, which will then be sent back on subsequent requests. |
| **Cache-Control** | Defines how long the browser should cache the response before requesting it again. |
| **Content-Type** | Tells the client what type of data is being returned — HTML, CSS, JS, image, PDF, video, etc. — so the browser knows how to render it. |
| **Content-Encoding** | Specifies the compression method used on the response body. |

---

## 🔧 HTTP Methods

HTTP methods indicate the **intended action** a client wants to perform on a resource.

| Method | Purpose |
|---|---|
| **GET** | Retrieve information from the web server. |
| **POST** | Submit data to the web server, potentially creating new records. |
| **PUT** | Submit data to update existing information on the server. |
| **DELETE** | Remove information/records from the server. |

> 💡 **Pentesting relevance:** Testing whether a server accepts unexpected methods (like `PUT` or `DELETE` on endpoints that shouldn't allow them) is a common step in identifying misconfigurations.

---

## 📊 HTTP Status Codes

Status codes tell the client the **result** of their request.

### ✅ 2xx — Success

| Code | Meaning |
|---|---|
| **200 OK** | The request was completed successfully. |
| **201 Created** | A new resource has been created (e.g. a new user or blog post). |

### 🔁 3xx — Redirection

| Code | Meaning |
|---|---|
| **301 Moved Permanently** | Redirects the client to a new URL / tells search engines the page has permanently moved. |
| **302 Found** | A temporary redirect — the resource may move back or change again later. |

### ⚠️ 4xx — Client Errors

| Code | Meaning |
|---|---|
| **400 Bad Request** | Something was wrong or missing in the request (e.g. a required parameter wasn't sent). |
| **401 Not Authorised** | Authentication (username/password, etc.) is required before accessing this resource. |
| **403 Forbidden** | Access to this resource is denied, regardless of login status. |
| **404 Page Not Found** | The requested page/resource does not exist. |
| **405 Method Not Allowed** | The resource doesn't support the HTTP method used (e.g. sending `GET` to an endpoint that expects `POST`). |

### 🔥 5xx — Server Errors

| Code | Meaning |
|---|---|
| **500 Internal Server Error** | The server encountered an unexpected error it couldn't handle properly. |
| **503 Service Unavailable** | The server is overloaded or down for maintenance and cannot handle the request. |

---

## ⚡ Quick Revision Cheat Sheet

```
REQUEST HEADERS          →  RESPONSE HEADERS
─────────────────────────────────────────────
Host                     →  Set-Cookie
User-Agent               →  Cache-Control
Content-Length           →  Content-Type
Accept-Encoding          →  Content-Encoding
Cookie

METHODS                  →  ACTION
─────────────────────────────────────────────
GET                      →  Read/Retrieve
POST                     →  Create
PUT                      →  Update
DELETE                   →  Remove

STATUS CODES             →  MEANING
─────────────────────────────────────────────
2xx  → Success           (200 OK, 201 Created)
3xx  → Redirection       (301 Permanent, 302 Temporary)
4xx  → Client Error      (400, 401, 403, 404, 405)
5xx  → Server Error      (500, 503)
```

---

## 🎯 Key Takeaways

- Headers carry **metadata** — they aren't the actual content, but they control how it's handled, cached, and rendered.
- HTTP **methods** define client intent — knowing which methods a server allows (and testing for insecure ones) is a common recon step in web pentesting.
- **Status codes** are the server's way of communicating outcomes — critical for debugging and for spotting misconfigurations during security assessments (e.g. a `405` revealing an unintended method is allowed).
- Mastering these fundamentals builds the base needed for more advanced topics like **HTTP request smuggling, IDOR, and broken authentication**.

---

<p align="center">
  <i>📌 Part of my TryHackMe learning journey — more writeups coming soon!</i>
</p>
