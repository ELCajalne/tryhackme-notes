# How the Web Works 🌍

---

## Table of Contents
- [DNS in Detail](#dns-in-detail)
- [HTTP in Detail](#http-in-detail)
- [How Websites Work](#how-websites-work)

---

## DNS in Detail

**Domain Name System (DNS)** — Translates domain names into IP addresses so you don't have to remember numbers.

### Domain Hierarchy

| Level | Description | Example |
|---|---|---|
| TLD (Top-Level Domain) | Rightmost part of a domain | `.com`, `.org`, `.ph` |
| gTLD | Indicates purpose | `.com` (commercial), `.edu` (education), `.gov` (government) |
| ccTLD | Indicates geography | `.ca` (Canada), `.ph` (Philippines) |
| Second-Level Domain | The main domain name | `tryhackme` in `tryhackme.com` |
| Subdomain | Sits left of the Second-Level Domain | `admin` in `admin.tryhackme.com` |

- Second-Level Domain: max 63 characters, a-z, 0-9, hyphens only.
- Total domain length: 253 characters or less.

### DNS Record Types

| Record | Description |
|---|---|
| A Record | Resolves to an IPv4 address |
| AAAA Record | Resolves to an IPv6 address |
| CNAME Record | Resolves to another domain name (alias). Requires a second DNS lookup. |
| MX Record | Resolves to the mail server for the domain. Includes priority flag for backup servers. |
| TXT Record | Free text field. Used to verify domain ownership or authorize email servers (anti-spam). |

### DNS Request Process
1. Browser checks **local cache**.
2. Request goes to **Recursive DNS Server** (usually your ISP). Checks its own cache.
3. If not found → queries **Root DNS Servers** → redirects to correct TLD server.
4. **TLD Server** → points to the **Authoritative Server** (nameserver) for the domain.
5. **Authoritative DNS Server** returns the record. Recursive DNS Server caches it.
6. All records have a **TTL (Time To Live)** — seconds the response is cached locally.

---

## HTTP in Detail

### What is HTTP?
**HyperText Transfer Protocol** — Developed by Tim Berners-Lee in 1989–1991. Rules for communicating with web servers to transmit webpage data.

### What is HTTPS?
Secure version of HTTP. Data is encrypted — prevents interception and verifies server identity.

### URL Structure

| Component | Description |
|---|---|
| Scheme | Protocol (HTTP, HTTPS, FTP) |
| User | Username and password for authentication |
| Host | Domain name or IP address |
| Port | 80 for HTTP, 443 for HTTPS |
| Path | File or location of the resource |
| Query String | Extra data (e.g., `/blog?id=1`) |
| Fragment | Reference to a specific location on the page |

### Making a Request
Minimum request: `GET / HTTP/1.1`

Example full request:
```
GET / HTTP/1.1
Host: tryhackme.com
User-Agent: Mozilla/5.0 Firefox/87.0
Referer: https://tryhackme.com/
```

- Line 1: GET method, home page `/`, HTTP 1.1
- Line 2: Website we want
- Line 3: Browser software and version
- Line 4: Referring page
- Line 5: Blank line = end of request

Example response:
```
HTTP/1.1 200 OK
Server: nginx/1.15.8
Date: Fri, 09 Apr 2021 13:34:03 GMT
Content-Type: text/html
Content-Length: 98

<html>...</html>
```

### HTTP Methods

| Method | Description |
|---|---|
| GET | Retrieve information from a web server |
| POST | Submit data — potentially creating new records |
| PUT | Submit data to update existing information |
| DELETE | Delete information/records |

### HTTP Status Codes

| Range | Category |
|---|---|
| 100–199 | Informational Response |
| 200–299 | Success |
| 300–399 | Redirection |
| 400–499 | Client Errors |
| 500–599 | Server Errors |

| Code | Meaning |
|---|---|
| 200 | OK — Request successful |
| 201 | Created — New resource created |
| 301 | Moved Permanently |
| 302 | Found (Temporary Redirect) |
| 400 | Bad Request |
| 401 | Not Authorized |
| 403 | Forbidden |
| 404 | Page Not Found |
| 405 | Method Not Allowed |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

### Headers

**Request Headers:**

| Header | Description |
|---|---|
| Host | Which website to serve when server hosts multiple |
| User-Agent | Browser software and version |
| Content-Length | How much data to expect |
| Accept-Encoding | Compression methods the browser supports |
| Cookie | Data to help server remember your information |

**Response Headers:**

| Header | Description |
|---|---|
| Set-Cookie | Instructs browser to store a cookie |
| Cache-Control | How long to cache the response |
| Content-Type | Type of data returned (HTML, CSS, JS, PDF, etc.) |
| Content-Encoding | Compression method used |

### Cookies
Used for website authentication. The value is usually a **token** — a unique secret code, not a plain-text password.

---

## How Websites Work

Two major components:
- **Front-End (Client-Side)** — How your browser renders the website.
- **Back-End (Server-Side)** — Server that processes requests and returns responses.

### HTML
**HyperText Markup Language** — The structure of websites.

Key elements:
- `<!DOCTYPE html>` — Defines HTML5 document.
- `<html>` — Root element.
- `<head>` — Metadata.
- `<body>` — Visible content.
- `<h1>` — Heading. `<p>` — Paragraph. `<button>` — Button. `<img>` — Image.

Attributes:
- `class` — Styling (multiple elements can share).
- `id` — Unique identifier for one element.

### JavaScript
Makes pages interactive and controls page functionality.

### Sensitive Data Exposure
Occurs when sensitive clear-text information is left in frontend source code — credentials, hidden links, or config data in HTML/JavaScript comments.

> Always review page source code when assessing a web application.

### HTML Injection & XSS
Occurs when unfiltered user input is displayed on the page.

Example: `<script>alert('hacked!')</script>` in an unprotected input field = **XSS (Cross-Site Scripting)** attack.

Prevention:
- **Validation** — Check input matches expected format.
- **Sanitization** — Remove or escape dangerous characters (`<`, `>`, `'`, `"`).
- **Encoding** — Convert input into safe text browsers won't execute.

> General rule: **Never trust user input.**

### Web Infrastructure

| Component | Description |
|---|---|
| Load Balancer | Distributes traffic across multiple servers. Uses round-robin or weighted algorithms. Performs health checks. |
| CDN | Hosts static files across thousands of servers worldwide. Serves from nearest location. |
| Database | Stores and retrieves data for web applications. |
| WAF | Sits between requests and web server. Blocks attacks, verifies real browsers, applies rate limiting. |

### Web Servers
Common: **Apache**, **Nginx**, **IIS**, **NodeJS**.

Default root directories:
- Nginx/Apache (Linux): `/var/www/html`
- IIS (Windows): `C:\inetpub\wwwroot`

**Virtual Hosts** — One web server hosting multiple websites with different domain names.

**Static Content** — Never changes (images, CSS, JS).

**Dynamic Content** — Changes based on requests. Generated by backend scripting languages.
