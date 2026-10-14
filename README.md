# Web Crawler Roadmap — From Zero to Production

> A complete, practical, and professional roadmap to master web crawling — from HTTP fundamentals to distributed, search-engine-scale systems.

[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

[![Stars](https://img.shields.io/github/stars/your-username/web-crawler-roadmap?style=social)](#)

---

## Table of Contents

1. [Introduction](#-introduction)
2. [Legal & Ethical Foundations](#-legal--ethical-foundations)
3. [Networking & Web Fundamentals](#-networking--web-fundamentals)
4. [HTTP Client Mastery](#-http-client-mastery)
5. [HTML Parsing & Data Extraction](#-html-parsing--data-extraction)
6. [Crawl Architecture Basics](#-crawl-architecture-basics)
7. [Robots, Sitemaps & Politeness](#-robots-sitemaps--politeness)
8. [Concurrency & Async Crawling](#-concurrency--async-crawling)
9. [Deduplication & URL Management](#-deduplication--url-management)
10. [Storage & Persistence](#-storage--persistence)
11. [Dynamic Sites & Headless Browsers](#-dynamic-sites--headless-browsers)
12. [API Reverse Engineering](#-api-reverse-engineering)
13. [Authentication, Sessions & Cookies](#-authentication-sessions--cookies)
14. [Proxies & Distributed Fetching](#-proxies--distributed-fetching)
15. [Anti-Bot Systems & Defenses](#-anti-bot-systems--defenses)
16. [Distributed Crawling at Scale](#-distributed-crawling-at-scale)
17. [Scheduling, Prioritization & Freshness](#-scheduling-prioritization--freshness)
18. [Data Quality & Content Processing](#-data-quality--content-processing)
19. [Search Indexing & Ranking](#-search-indexing--ranking)
20. [Monitoring, Observability & Ops](#-monitoring-observability--ops)
21. [Performance & Cost Optimization](#-performance--cost-optimization)
22. [Security & Compliance](#-security--compliance)
23. [Capstone Projects](#-capstone-projects)
24. [Recommended Stack](#-recommended-stack)
25. [Learning Resources](#-learning-resources)
26. [Contributing](#-contributing)
27. [License](#-license)

---

## Introduction

A **web crawler** (or spider) is a program that systematically browses the web to discover and collect data. It powers search engines, price comparison tools, news aggregators, ML datasets, and market intelligence platforms.

This roadmap is a **workflow-driven guide**. Each section builds on the previous one. You can follow it top-to-bottom to go from zero to production-grade crawler engineering.

**Who this is for:**

- Backend engineers moving into data engineering
- Students building portfolio projects
- Data scientists who need custom datasets
- Anyone building search, aggregation, or monitoring systems

**How to use this repo:**

- Star it if it helps
- Fork it and track your progress
- Check off sections as you complete them
- Build the mini-projects in each section

---

## Legal & Ethical Foundations

> **Non-negotiable.** Read this before writing a single line of crawler code.

### Key Concepts

| Topic | Why It Matters |
|---|---|
| `robots.txt` | Machine-readable crawling rules set by site owners |
| Terms of Service (ToS) | Contractual rules — violating them can get you banned or sued |
| Copyright | Scraped content may be protected; redistribution is illegal |
| Personal Data (GDPR, CCPA) | Scraping PII has strict legal consequences |
| CFAA (US) / Computer Misuse Act (UK) | Unauthorized access can be criminal |
| Rate Limiting | Overloading a server is a form of DoS |
| Public APIs | Always prefer official APIs when available |

### Golden Rules

1. **Identify your bot** — set a real `User-Agent` with contact info:
   ```
   MyBot/1.0 (+https://example.com/bot; contact@example.com)
   ```
2. **Read `robots.txt`** before crawling any domain.
3. **Respect rate limits** — start at 1 req/sec per domain.
4. **Prefer APIs** over HTML scraping.
5. **Cache aggressively** to avoid re-fetching.
6. **Never bypass anti-bot systems** without written permission.
7. **Get explicit authorization** before load testing or aggressive crawling.
8. **Store only what you need.** Delete PII you don't require.

### Example: Parsing `robots.txt` (Python)

```python
from urllib.robotparser import RobotFileParser

rp = RobotFileParser()
rp.set_url("https://example.com/robots.txt")
rp.read()

user_agent = "MyBot/1.0"
url = "https://example.com/private/page"

if rp.can_fetch(user_agent, url):
    print("Allowed to crawl")
else:
    print("Blocked by robots.txt")
```

### Example: `robots.txt` Syntax

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/
Crawl-delay: 1

Sitemap: https://example.com/sitemap.xml
```

---

## Networking & Web Fundamentals

Before crawling, understand what actually happens when you type a URL.

### The Request Lifecycle

1. **DNS Resolution** — domain → IP address
2. **TCP Handshake** — SYN → SYN-ACK → ACK
3. **TLS Handshake** — certificate exchange, key negotiation
4. **HTTP Request** — method, path, headers, body
5. **Server Processing** — routing, DB, rendering
6. **HTTP Response** — status, headers, body
7. **Connection Close / Keep-Alive**

### Essential HTTP Status Codes

| Code | Meaning | Crawler Action |
|---|---|---|
| 200 | OK | Parse and continue |
| 301 | Permanent Redirect | Follow, update URL |
| 302 | Temporary Redirect | Follow, keep original |
| 304 | Not Modified | Use cached version |
| 401 | Unauthorized | Auth required |
| 403 | Forbidden | Likely blocked |
| 404 | Not Found | Skip |
| 429 | Too Many Requests | Back off, retry |
| 500 | Server Error | Retry with backoff |
| 503 | Service Unavailable | Retry with backoff |

### Key Headers

```http
GET /page HTTP/1.1
Host: example.com
User-Agent: MyBot/1.0 (+https://example.com/bot)
Accept: text/html,application/xhtml+xml
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: session=abc123
Referer: https://example.com/
```

### URL Anatomy

```
https://user:pass@example.com:8080/path/page?query=1&sort=asc#section
└─┬─┘   └──┬───┘ └────┬────┘└┬┘└────┬────┘└────┬─────┘└──┬──┘
scheme   auth      host    port  path        query     fragment
```

### URL Normalization Rules

| Input | Normalized |
|---|---|
| `HTTP://Example.com/` | `http://example.com/` |
| `https://example.com/a/../b` | `https://example.com/b` |
| `https://example.com/?b=2&a=1` | `https://example.com/?a=1&b=2` |
| `https://example.com/page#top` | `https://example.com/page` |
| `https://example.com/page/` | `https://example.com/page` |

---

## HTTP Client Mastery

The HTTP client is your crawler's engine. Choose based on language and concurrency needs.

### Python

```python
# requests — sync, simple
import requests

headers = {"User-Agent": "MyBot/1.0 (+https://example.com/bot)"}
resp = requests.get("https://example.com", headers=headers, timeout=10)
resp.raise_for_status()
html = resp.text
```

```python
# httpx — supports sync + async, HTTP/2
import httpx

with httpx.Client(http2=True, timeout=10) as client:
    resp = client.get("https://example.com")
    print(resp.status_code)
```

```python
# aiohttp — async, high throughput
import aiohttp
import asyncio

async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()

asyncio.run(fetch("https://example.com"))
```

### Node.js

```js
// axios — simple
const axios = require('axios');

const { data } = await axios.get('https://example.com', {
  headers: { 'User-Agent': 'MyBot/1.0' },
  timeout: 10000,
});
```

```js
// undici — modern, fast
import { request } from 'undici';

const { statusCode, body } = await request('https://example.com');
const html = await body.text();
```

### Go

```go
resp, err := http.Get("https://example.com")
if err != nil { log.Fatal(err) }
defer resp.Body.Close()
body, _ := io.ReadAll(resp.Body)
```

### Retry with Exponential Backoff

```python
import time
import requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

session = requests.Session()
retries = Retry(
    total=5,
    backoff_factor=1,
    status_forcelist=[429, 500, 502, 503, 504],
    allowed_methods=["GET", "HEAD"],
)
session.mount("https://", HTTPAdapter(max_retries=retries))

resp = session.get("https://example.com")
```

### Timeout & Error Handling

```python
try:
    resp = requests.get(url, timeout=(3.05, 10))  # (connect, read)
    resp.raise_for_status()
except requests.Timeout:
    print("Timed out")
except requests.HTTPError as e:
    print(f"HTTP error: {e.response.status_code}")
except requests.RequestException as e:
    print(f"Request failed: {e}")
```

---

## HTML Parsing & Data Extraction

Raw HTML is noise. Parsing turns it into structured data.

### BeautifulSoup (Python)

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "lxml")

title = soup.select_one("h1.title").get_text(strip=True)
links = [a["href"] for a in soup.select("a[href]")]
price = soup.find("span", class_="price").text
```

### lxml + XPath (Python)

```python
from lxml import html as lxml_html

tree = lxml_html.fromstring(html)
titles = tree.xpath("//h2[@class='title']/text()")
links = tree.xpath("//a/@href")
```

### Cheerio (Node.js)

```js
const cheerio = require('cheerio');
const $ = cheerio.load(html);

$('h1.title').text();
$('a').each((i, el) => console.log($(el).attr('href')));
```

### Structured Data

| Format | Use Case |
|---|---|
| JSON-LD | `<script type="application/ld+json">` — schema.org data |
| Microdata | `itemprop` attributes in HTML |
| Open Graph | `og:title`, `og:image` for social sharing |
| Twitter Cards | `twitter:title`, `twitter:image` |
| RSS / Atom | Feeds with dates and entries |
| Sitemaps | XML list of URLs for crawlers |

```python
import json
from bs4 import BeautifulSoup

soup = BeautifulSoup(html, "lxml")
for script in soup.find_all("script", type="application/ld+json"):
    data = json.loads(script.string)
    print(data.get("name"), data.get("price"))
```

### CSS Selectors vs XPath

| Task | CSS | XPath |
|---|---|---|
| By class | `.title` | `//*[@class='title']` |
| By id | `#main` | `//*[@id='main']` |
| By attribute | `a[href]` | `//a[@href]` |
| Contains text | — | `//*[contains(text(),'Hi')]` |
| Parent | — | `//span/..` |
| Sibling | `h1 + p` | `//h1/following-sibling::p[1]` |

---

## Crawl Architecture Basics

### Minimal Crawler Loop

```python
from collections import deque
from urllib.parse import urljoin, urlparse
import requests
from bs4 import BeautifulSoup

def crawl(seed, max_pages=100):
    seen = set()
    queue = deque([seed])
    results = []

    while queue and len(seen) < max_pages:
        url = queue.popleft()
        if url in seen:
            continue
        seen.add(url)

        try:
            resp = requests.get(url, timeout=10)
            if resp.status_code != 200:
                continue
        except requests.RequestException:
            continue

        soup = BeautifulSoup(resp.text, "lxml")
        results.append({
            "url": url,
            "title": soup.title.string if soup.title else None,
        })

        for a in soup.select("a[href]"):
            link = urljoin(url, a["href"])
            if urlparse(link).netloc == urlparse(seed).netloc:
                queue.append(link)

    return results
```

### Crawler Components

```
┌─────────────┐    ┌──────────┐    ┌──────────┐    ┌────────┐
│  Scheduler  │───▶│ Fetcher  │───▶│  Parser  │───▶│ Storage│
└─────────────┘    └──────────┘    └──────────┘    └────────┘
       ▲                                                  │
       │              ┌──────────────┐                    │
       └──────────────│ URL Frontier │◀───────────────────┘
                      └──────────────┘
```

| Component | Responsibility |
|---|---|
| Scheduler | Decides what to crawl next |
| Frontier | Priority queue of pending URLs |
| Fetcher | Performs HTTP requests |
| Parser | Extracts links and data |
| Deduplicator | Removes already-seen URLs |
| Storage | Persists results and state |
| Monitor | Tracks health and metrics |

---

## Robots, Sitemaps & Politeness

### Parsing `robots.txt` Per Domain

```python
from urllib.robotparser import RobotFileParser
from urllib.parse import urlparse
from functools import lru_cache

@lru_cache(maxsize=1024)
def get_robots(domain):
    rp = RobotFileParser()
    rp.set_url(f"https://{domain}/robots.txt")
    try:
        rp.read()
    except Exception:
        pass
    return rp

def can_fetch(url, ua="MyBot/1.0"):
    domain = urlparse(url).netloc
    return get_robots(domain).can_fetch(ua, url)
```

### Sitemaps

```python
import requests
from bs4 import BeautifulSoup

def parse_sitemap(url):
    resp = requests.get(url)
    soup = BeautifulSoup(resp.content, "xml")
    return [loc.text for loc in soup.find_all("loc")]
```

### Politeness Strategy

| Rule | Value |
|---|---|
| Min delay per domain | 1 second |
| Max concurrent per domain | 1–3 |
| Respect `Crawl-delay` | Always |
| Off-peak crawling | Preferred |
| Backoff on 429/503 | Exponential |
| Honor `Retry-After` header | Always |

```python
import time
from urllib.parse import urlparse

last_request = {}

def polite_fetch(url, delay=1.0):
    domain = urlparse(url).netloc
    now = time.time()
    if domain in last_request:
        wait = delay - (now - last_request[domain])
        if wait > 0:
            time.sleep(wait)
    last_request[domain] = time.time()
    return requests.get(url)
```

---

## Concurrency & Async Crawling

### Thread Pool (Python)

```python
from concurrent.futures import ThreadPoolExecutor
import requests

def fetch(url):
    return requests.get(url, timeout=10).text

urls = [f"https://example.com/page/{i}" for i in range(100)]

with ThreadPoolExecutor(max_workers=20) as executor:
    results = list(executor.map(fetch, urls))
```

### Async with `asyncio` + `aiohttp`

```python
import asyncio
import aiohttp

async def fetch(session, url, sem):
    async with sem:
        try:
            async with session.get(url, timeout=10) as resp:
                return url, await resp.text()
        except Exception as e:
            return url, None

async def crawl(urls, concurrency=20):
    sem = asyncio.Semaphore(concurrency)
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, u, sem) for u in urls]
        return await asyncio.gather(*tasks)

urls = [f"https://example.com/page/{i}" for i in range(1000)]
results = asyncio.run(crawl(urls))
```

### Worker Pool with Queue

```python
import asyncio
from asyncio import Queue

async def worker(name, queue):
    while True:
        url = await queue.get()
        try:
            # fetch and parse
            await asyncio.sleep(0.1)
        finally:
            queue.task_done()

async def main(urls):
    queue = Queue()
    for u in urls:
        await queue.put(u)

    workers = [asyncio.create_task(worker(f"w{i}", queue)) for i in range(20)]
    await queue.join()
    for w in workers:
        w.cancel()

asyncio.run(main([f"https://example.com/{i}" for i in range(1000)]))
```

### Concurrency Comparison

| Model | Pros | Cons |
|---|---|---|
| Threads | Simple, works with sync libs | GIL limits CPU-bound work |
| Processes | True parallelism | High memory, IPC overhead |
| `asyncio` | Massive concurrency, low memory | Async-only libraries |
| Go goroutines | Simple + fast | Different language |
| Node.js async | Event-loop friendly | Single-threaded CPU |

---

## Deduplication & URL Management

### Exact Dedup with Set

```python
seen = set()

def add_url(url):
    if url in seen:
        return False
    seen.add(url)
    return True
```

### Normalized URL Hash

```python
import hashlib
from urllib.parse import urlsplit, urlunsplit, parse_qsl, urlencode

def normalize(url):
    parts = urlsplit(url)
    query = urlencode(sorted(parse_qsl(parts.query)))
    path = parts.path.rstrip("/") or "/"
    return urlunsplit((
        parts.scheme.lower(),
        parts.netloc.lower(),
        path,
        query,
        "",
    ))

def url_hash(url):
    return hashlib.sha256(normalize(url).encode()).hexdigest()
```

### Bloom Filter (Memory Efficient)

```python
from pybloom_live import BloomFilter

bf = BloomFilter(capacity=10_000_000, error_rate=0.001)

if url not in bf:
    bf.add(url)
    # crawl it
```

### Content Deduplication

| Algorithm | Use Case |
|---|---|
| MD5 / SHA-256 | Exact content match |
| SimHash | Near-duplicate detection |
| MinHash + LSH | Large-scale near-dup |
| TLSH / ssdeep | Fuzzy hashing |

```python
from simhash import Simhash

def fingerprint(text):
    return Simhash(text.split()).value

h1 = fingerprint("hello world")
h2 = fingerprint("hello worlds")
print(bin(h1 ^ h2).count("1"))  # Hamming distance
```

---

## Storage & Persistence

### When to Use What

| Storage | Best For |
|---|---|
| SQLite | Local dev, small crawls |
| PostgreSQL | Structured data, queries, joins |
| MongoDB | Flexible documents, nested data |
| Redis | Queues, dedup, caching |
| S3 / GCS | Raw HTML, large blobs |
| Elasticsearch | Search, full-text queries |
| Parquet | Analytics, ML pipelines |

### PostgreSQL Schema Example

```sql
CREATE TABLE pages (
    id           BIGSERIAL PRIMARY KEY,
    url          TEXT UNIQUE NOT NULL,
    url_hash     CHAR(64) NOT NULL,
    status       INT,
    title        TEXT,
    content      TEXT,
    fetched_at   TIMESTAMPTZ DEFAULT NOW(),
    domain       TEXT
);

CREATE INDEX idx_pages_domain ON pages(domain);
CREATE INDEX idx_pages_fetched ON pages(fetched_at DESC);
```

### Python Insert Example

```python
import psycopg2
from psycopg2.extras import execute_batch

conn = psycopg2.connect("dbname=crawler user=postgres")
cur = conn.cursor()

rows = [
    ("https://example.com/1", "hash1", 200, "Title 1"),
    ("https://example.com/2", "hash2", 200, "Title 2"),
]

execute_batch(cur, """
    INSERT INTO pages (url, url_hash, status, title)
    VALUES (%s, %s, %s, %s)
    ON CONFLICT (url) DO NOTHING
""", rows)

conn.commit()
```

### Checkpointing & Resume

```python
import json

def save_state(state, path="crawl_state.json"):
    with open(path, "w") as f:
        json.dump(state, f)

def load_state(path="crawl_state.json"):
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError:
        return {"seen": [], "queue": []}
```

---

## Dynamic Sites & Headless Browsers

Modern sites render with JavaScript. Static HTTP fetches only see the shell.

### Playwright (Python)

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com", wait_until="networkidle")
    page.wait_for_selector(".product-card")
    items = page.query_selector_all(".product-card")
    for item in items:
        print(item.inner_text())
    browser.close()
```

### Puppeteer (Node.js)

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ headless: 'new' });
  const page = await browser.newPage();
  await page.goto('https://example.com', { waitUntil: 'networkidle2' });
  await page.waitForSelector('.product-card');
  const items = await page.$$eval('.product-card', els =>
    els.map(e => e.innerText)
  );
  console.log(items);
  await browser.close();
})();
```

### Intercepting Network Requests

```python
def handle_response(response):
    if "/api/products" in response.url:
        print(response.json())

page.on("response", handle_response)
page.goto("https://example.com")
```

### Best Practices

- Use `wait_for_selector`, not `sleep`.
- Block images, fonts, and CSS for speed:
  ```python
  page.route("**/*.{png,jpg,css,woff2}", lambda r: r.abort())
  ```
- Reuse browser contexts.
- Prefer APIs when available.
- Set realistic viewport and user agent.
- Use headless mode in production.

### When to Avoid Headless

- If the site has an API, hit it directly.
- If the content is in the initial HTML, use static fetch.
- If the site blocks headless, use a stealth plugin *only with permission*.

---

## API Reverse Engineering

Most SPAs fetch data via XHR/fetch. Find the API and skip the DOM.

### Discovery

1. Open DevTools → **Network** tab.
2. Filter by **Fetch/XHR**.
3. Reload and inspect requests.
4. Look for JSON responses.

### Example: Paginated JSON API

```python
import requests

def fetch_page(page=1):
    url = "https://example.com/api/products"
    params = {"page": page, "per_page": 50}
    headers = {"User-Agent": "MyBot/1.0"}
    resp = requests.get(url, params=params, headers=headers)
    return resp.json()

all_items = []
for page in range(1, 11):
    data = fetch_page(page)
    all_items.extend(data["items"])
```

### GraphQL

```python
query = """
query Products($first: Int!) {
  products(first: $first) {
    edges { node { id name price } }
  }
}
"""

resp = requests.post(
    "https://example.com/graphql",
    json={"query": query, "variables": {"first": 50}},
    headers={"Content-Type": "application/json"},
)
data = resp.json()
```

### Common API Patterns

| Pattern | Example |
|---|---|
| Pagination | `?page=2&per_page=50` |
| Cursor | `?after=cursor_token` |
| Offset | `?offset=100&limit=50` |
| Sort | `?sort=price&order=asc` |
| Filter | `?category=books&min_price=10` |
| Auth | `Authorization: Bearer <token>` |

---

## Authentication, Sessions & Cookies

### Session with `requests`

```python
import requests

session = requests.Session()
session.headers.update({"User-Agent": "MyBot/1.0"})

# Login
login_data = {"username": "user", "password": "pass"}
session.post("https://example.com/login", data=login_data)

# Authenticated request
resp = session.get("https://example.com/dashboard")
```

### Token-Based Auth

```python
headers = {"Authorization": "Bearer eyJhbGciOi..."}
resp = requests.get("https://api.example.com/me", headers=headers)
```

### Cookie Persistence

```python
import pickle

# Save cookies
with open("cookies.pkl", "wb") as f:
    pickle.dump(session.cookies, f)

# Load cookies
with open("cookies.pkl", "rb") as f:
    session.cookies.update(pickle.load(f))
```

### OAuth 2.0 Flow (High-Level)

1. Redirect user to provider.
2. User authorizes.
3. Provider redirects back with `code`.
4. Exchange `code` for `access_token`.
5. Use token in `Authorization` header.
6. Refresh with `refresh_token` when expired.

---

## Proxies & Distributed Fetching

> **Legal note:** Use only for authorized testing. Never bypass rate limits on systems you don't own.

### Proxy Types

| Type | Anonymity | Speed | Cost |
|---|---|---|---|
| Datacenter | Low | High | Low |
| Residential | High | Medium | High |
| Mobile | Very High | Medium | Very High |
| ISP | High | High | Medium |
| Tor | Very High | Very Low | Free |

### Using Proxies with `requests`

```python
proxies = {
    "http": "http://user:pass@proxy.example.com:8080",
    "https": "https://user:pass@proxy.example.com:8080",
}
resp = requests.get("https://example.com", proxies=proxies, timeout=10)
```

### Rotating Proxies

```python
import random

proxy_pool = [
    "http://p1.example.com:8080",
    "http://p2.example.com:8080",
    "http://p3.example.com:8080",
]

def get_proxy():
    return {"http": random.choice(proxy_pool), "https": random.choice(proxy_pool)}

resp = requests.get("https://example.com", proxies=get_proxy())
```

### Proxy Rotation Strategy

- **Per request**: highest anonymity, most expensive.
- **Per session**: sticky sessions, good for auth.
- **Per domain**: avoid mixing.
- **Backoff on failure**: never hammer a dead proxy.

---

## Anti-Bot Systems & Defenses

### Common Defenses

| System | Detection Method |
|---|---|
| Cloudflare | JS challenge, TLS fingerprint, behavioral |
| Akamai | Bot detection, device fingerprint |
| PerimeterX / HUMAN | Browser fingerprint, ML models |
| DataDome | Real-time behavioral analysis |
| reCAPTCHA / hCaptcha | Human challenge |
| Rate limiting | Per IP, per token, per route |
| Honeypots | Hidden links only bots follow |
| TLS fingerprinting | JA3/JA4 hashes |

### Detection Vectors

- **Headers**: missing `Accept-Language`, `Sec-*` headers
- **TLS**: Python `requests` has a distinct JA3 signature
- **Behavior**: no mouse movement, instant clicks
- **Timing**: too fast, too consistent
- **Fingerprint**: `navigator.webdriver`, canvas, WebGL

### Ethical Response

1. **Prefer the API.**
2. **Slow down.**
3. **Rotate User-Agents** only for authorized testing.
4. **Use official partnership programs.**
5. **Respect blocks** — move on.

> Bypassing anti-bot systems without authorization can violate the CFAA, ToS, and local laws. This section is for defense awareness and authorized testing only.

---

## Distributed Crawling at Scale

### Architecture

```
                    ┌─────────────┐
                    │  Scheduler  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────┐
        │ Fetcher1 │ │ Fetcher2 │ │ Fetcher3 │
        └────┬─────┘ └────┬─────┘ └────┬─────┘
             │            │            │
             └────────────┼────────────┘
                          ▼
                   ┌─────────────┐
                   │   Kafka /   │
                   │   Redis     │
                   └──────┬──────┘
                          ▼
                   ┌─────────────┐
                   │   Parser    │
                   └──────┬──────┘
                          ▼
                   ┌─────────────┐
                   │   Storage   │
                   └─────────────┘
```

### Key Components

| Component | Tool |
|---|---|
| Message queue | Kafka, RabbitMQ, SQS |
| Distributed cache | Redis, Memcached |
| Coordination | Zookeeper, etcd |
| Container orchestration | Kubernetes |
| Object storage | S3, GCS |
| Metrics | Prometheus, Grafana |
| Logs | ELK, Loki |

### Kubernetes Deployment Sketch

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: crawler-worker
spec:
  replicas: 10
  selector:
    matchLabels:
      app: crawler
  template:
    metadata:
      labels:
        app: crawler
    spec:
      containers:
      - name: worker
        image: myorg/crawler:1.0
        env:
        - name: REDIS_URL
          value: "redis://redis:6379"
        - name: KAFKA_BROKERS
          value: "kafka:9092"
```

### Distributed Dedup with Redis

```python
import redis

r = redis.Redis(host="redis", port=6379)

def is_new(url_hash):
    # SETNX returns True if key was set
    return r.setnx(f"seen:{url_hash}", 1)
```

### Sharding by Domain

```python
import hashlib

def shard_for(url):
    domain = urlparse(url).netloc
    return int(hashlib.md5(domain.encode()).hexdigest(), 16) % NUM_SHARDS
```

---

## ⏰ Scheduling, Prioritization & Freshness

### Priority Queue

```python
import heapq
import time

class Frontier:
    def __init__(self):
        self.heap = []
        self.counter = 0

    def push(self, url, priority=0):
        # Lower priority value = higher importance
        heapq.heappush(self.heap, (priority, self.counter, url))
        self.counter += 1

    def pop(self):
        return heapq.heappop(self.heap)[2]
```

### Priority Signals

| Signal | Weight |
|---|---|
| Homepage / top-level | High |
| Sitemap URL | High |
| Recently updated | High |
| High PageRank | Medium |
| Deep link | Low |
| Known low-quality | Skip |

### Recrawl Policies

| Page Type | Recrawl Frequency |
|---|---|
| News homepage | Every 5 min |
| News article | Every 1 hour |
| Blog post | Every 1 day |
| Product page | Every 1 hour |
| Documentation | Every 1 week |
| Static page | Every 1 month |

### Change Detection

```python
import hashlib

def content_hash(text):
    return hashlib.sha256(text.encode()).hexdigest()

# On each recrawl, compare with previous hash
if new_hash != old_hash:
    # page changed → update, re-index, notify
    pass
```

---

## Data Quality & Content Processing

### Cleaning Pipeline

1. **Extract main content** — remove nav, ads, footers.
2. **Normalize whitespace** and encoding.
3. **Language detection** — `langdetect`, `fastText`.
4. **Boilerplate removal** — `trafilatura`, `readability-lxml`.
5. **PII scrubbing** — emails, phones, IDs.
6. **Deduplication** — exact + near-dup.
7. **Classification** — topic, sentiment, intent.
8. **Entity extraction** — people, orgs, locations.

### Content Extraction with Trafilatura

```python
import trafilatura

downloaded = trafilatura.fetch_url("https://example.com/article")
text = trafilatura.extract(downloaded)
```

### Language Detection

```python
from langdetect import detect

lang = detect("Bonjour tout le monde")
print(lang)  # "fr"
```

### PII Scrubbing

```python
import re

def scrub_pii(text):
    text = re.sub(r"\b[\w.]+@[\w.]+\b", "[EMAIL]", text)
    text = re.sub(r"\b\d{3}-\d{2}-\d{4}\b", "[SSN]", text)
    text = re.sub(r"\b\d{10,}\b", "[NUMBER]", text)
    return text
```

### Schema for Extracted Records

```json
{
  "url": "https://example.com/article",
  "title": "Example Article",
  "author": "Jane Doe",
  "published_at": "2024-01-15T10:00:00Z",
  "language": "en",
  "content": "Full text...",
  "summary": "Short summary...",
  "tags": ["tech", "ai"],
  "entities": {"people": ["Jane Doe"], "orgs": ["OpenAI"]},
  "content_hash": "sha256:...",
  "fetched_at": "2024-01-15T10:05:00Z"
}
```

---

## Search Indexing & Ranking

### Inverted Index Basics

```
doc1: "the quick brown fox"
doc2: "the lazy brown dog"

index:
  the   → [doc1, doc2]
  quick → [doc1]
  brown → [doc1, doc2]
  fox   → [doc1]
  lazy  → [doc2]
  dog   → [doc2]
```

### BM25 Scoring

```python
from rank_bm25 import BM25Okapi

corpus = [
    "the quick brown fox".split(),
    "the lazy brown dog".split(),
]
bm25 = BM25Okapi(corpus)
scores = bm25.get_scores("brown fox".split())
```

### PageRank (Simplified)

```python
import networkx as nx

G = nx.DiGraph()
G.add_edges_from([("a", "b"), ("b", "c"), ("c", "a"), ("d", "a")])
pr = nx.pagerank(G, alpha=0.85)
print(pr)
```

### Elasticsearch Indexing

```python
from elasticsearch import Elasticsearch

es = Elasticsearch("http://localhost:9200")

es.index(index="pages", id=1, document={
    "url": "https://example.com/1",
    "title": "Hello World",
    "content": "This is a test page.",
})

es.indices.refresh(index="pages")

resp = es.search(index="pages", query={"match": {"content": "test"}})
```

### Hybrid Ranking

```
final_score = 0.5 * bm25_score
            + 0.3 * pagerank_score
            + 0.1 * freshness_score
            + 0.1 * quality_score
```

---

## Monitoring, Observability & Ops

### Metrics to Track

| Metric | Why |
|---|---|
| Requests/sec | Throughput |
| Success rate | Health |
| Error rate by code | Debug |
| Latency p50/p95/p99 | Performance |
| Queue depth | Backlog |
| URLs discovered | Coverage |
| URLs crawled | Progress |
| Duplicates rate | Efficiency |
| Bytes fetched | Cost |
| Retries | Network issues |

### Prometheus + Grafana

```python
from prometheus_client import Counter, Histogram, start_http_server

REQUESTS = Counter("crawler_requests_total", "Total requests", ["status"])
LATENCY = Histogram("crawler_latency_seconds", "Request latency")

@LATENCY.time()
def fetch(url):
    resp = requests.get(url)
    REQUESTS.labels(status=resp.status_code).inc()
    return resp

start_http_server(8000)
```

### Structured Logging

```python
import structlog

log = structlog.get_logger()

log.info("fetch", url=url, status=200, latency_ms=123)
log.error("fetch_failed", url=url, error=str(e))
```

### Alerting Rules

- Error rate > 5% for 5 minutes
- Queue depth > 100k
- p99 latency > 5s
- Zero requests for 60s

---

## Performance & Cost Optimization

### Techniques

| Technique | Benefit |
|---|---|
| HTTP/2 + keep-alive | Fewer connections |
| Compression (gzip, brotli) | Less bandwidth |
| Conditional requests (`If-Modified-Since`) | Skip unchanged |
| Caching | Avoid re-fetch |
| Async I/O | Higher concurrency |
| Connection pooling | Lower overhead |
| DNS caching | Faster resolution |
| Content truncation | Less storage |
| Cold storage (S3 Glacier) | Cheaper archive |

### Conditional Requests

```python
headers = {"If-Modified-Since": last_fetched_http_date}
resp = requests.get(url, headers=headers)

if resp.status_code == 304:
    pass  # use cached version
else:
    # update cache
    pass
```

### Cost Model

```
cost = (bandwidth_gb * bw_price)
     + (storage_gb * storage_price)
     + (compute_hours * compute_price)
     + (proxy_requests * proxy_price)
```

Track per-domain cost to identify expensive targets.

---

## Security & Compliance

### Crawler Security

- Never execute untrusted JavaScript outside a sandbox.
- Validate all URLs (SSRF protection).
- Limit redirect depth.
- Cap response size.
- Sanitize extracted HTML.
- Rotate secrets, never commit them.

### SSRF Protection

```python
import ipaddress
from urllib.parse import urlparse
import socket

BLOCKED = [
    ipaddress.ip_network("10.0.0.0/8"),
    ipaddress.ip_network("127.0.0.0/8"),
    ipaddress.ip_network("169.254.0.0/16"),
    ipaddress.ip_network("172.16.0.0/12"),
    ipaddress.ip_network("192.168.0.0/16"),
]

def is_safe(url):
    host = urlparse(url).hostname
    ip = ipaddress.ip_address(socket.gethostbyname(host))
    return not any(ip in net for net in BLOCKED)
```

### Compliance Checklist

- [ ] `robots.txt` respected
- [ ] ToS reviewed
- [ ] Rate limits enforced
- [ ] PII scrubbed
- [ ] Data retention policy
- [ ] User-Agent identifies bot
- [ ] Contact email in UA
- [ ] Opt-out mechanism
- [ ] Data deletion on request
- [ ] Audit logs

---

## Capstone Projects

### Project 1 — Personal News Aggregator

- Crawl 10 news sites.
- Extract articles + metadata.
- Store in PostgreSQL.
- Serve via FastAPI + simple UI.
- Schedule hourly recrawls.

### Project 2 — Price Tracker

- Crawl e-commerce product pages.
- Track price over time.
- Alert on drops.
- Dashboard with charts.

### Project 3 — Mini Search Engine

- Crawl a domain (e.g., docs.python.org).
- Build inverted index + PageRank.
- Serve search UI with BM25 ranking.
- Deploy on a VPS.

### Project 4 — Distributed Crawler

- Deploy on Kubernetes.
- Use Kafka + Redis + PostgreSQL.
- Crawl 1M+ pages.
- Monitor with Prometheus + Grafana.

### Project 5 — ML Dataset Builder

- Crawl a niche (recipes, jobs, papers).
- Clean and deduplicate.
- Extract structured entities.
- Export to Parquet.
- Publish on Hugging Face.

---

## Recommended Stack

| Layer | Options |
|---|---|
| Language | Python, Go, Node.js |
| HTTP | `httpx`, `aiohttp`, `requests`, `undici` |
| Parsing | `lxml`, `BeautifulSoup`, `Cheerio` |
| Browser | Playwright, Puppeteer |
| Framework | Scrapy, Crawlee, Colly |
| Queue | Redis, Kafka, RabbitMQ, SQS |
| Storage | PostgreSQL, MongoDB, S3 |
| Search | Elasticsearch, Meilisearch, Typesense |
| Orchestration | Docker, Kubernetes |
| Monitoring | Prometheus, Grafana, Loki |
| Proxy | Bright Data, Oxylabs, ScraperAPI |

---

## Learning Resources

### Books

- *Web Scraping with Python* — Ryan Mitchell
- *Mining the Social Web* — Matthew Russell
- *Introduction to Information Retrieval* — Manning et al.
- *Designing Data-Intensive Applications* — Martin Kleppmann

### Papers

- *The Anatomy of a Large-Scale Hypertextual Web Search Engine* — Brin & Page
- *Mercator: A Scalable, Extensible Web Crawler* — Heydon & Najork
- *Efficient Crawling Through URL Ordering* — Cho et al.

### Tools

- [Scrapy](https://scrapy.org/)
- [Crawlee](https://crawlee.dev/)
- [Colly](https://go-colly.org/)
- [Playwright](https://playwright.dev/)
- [Trafilatura](https://trafilatura.readthedocs.io/)
- [Apache Nutch](https://nutch.apache.org/)
- [Heritrix](https://github.com/internetarchive/heritrix3)

### Communities

- r/webscraping
- Scrapy Discord
- Indie Hackers — Data
- Hacker News — scraping threads

---

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

1. Fork the repo.
2. Create a feature branch.
3. Commit your changes.
4. Open a pull request.

---

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

## Support

If this roadmap helped you, please:

- Star the repo
- Share on Twitter / LinkedIn
- Open an issue with feedback
- Fork and build something cool

---

> **Disclaimer:** This roadmap is for educational purposes. Always respect `robots.txt`, terms of service, copyright, and local laws. Never use these techniques to attack, overload, or bypass protections on systems you do not own or have explicit permission to test.
