# ScraperAPI API Tutorial: How to Get Started, Send Your First Request, Handle JavaScript & Bot-Protected Sites — Plus Every Pricing Plan Explained (With Real Credit Math and Code Examples)

If you've spent more than five minutes trying to scrape data from a modern website, you already know the drill. You write a clean `requests.get()` call, run it, and get back a `403 Forbidden`, a CAPTCHA page, or worse — a honeypot response that looks like real HTML but is full of fake data meant to fool you. It's frustrating in a very specific way, because the *idea* of web scraping is dead simple. It's all the invisible machinery around it — proxies, rotating IPs, Cloudflare checks, JavaScript rendering — that turns a ten-line script into a week-long rabbit hole.

That's what ScraperAPI is built to solve. You send it a URL. It sends back the HTML. All the messy proxy management, CAPTCHA solving, and retry logic happens on their end, not yours.

This guide covers the ScraperAPI API tutorial from scratch — from signing up and getting your API key, to sending synchronous requests, handling JavaScript-heavy pages, running bulk async jobs, and pulling structured JSON from Amazon or Google without writing a single parser. Plus a full breakdown of every pricing plan with the actual math on what credits buy you at each tier.

---

## **What ScraperAPI Actually Does (In Plain Terms)**

ScraperAPI sits between your code and the target website. Instead of making a direct HTTP request, you forward the request through ScraperAPI's infrastructure. Their system picks the best proxy from a pool of 40 million+ IPs across 50+ countries, applies the right headers and browser fingerprint, handles CAPTCHAs if they come up, and retries automatically if the first attempt fails.

From your code's perspective, you're just hitting one API endpoint and getting HTML back. The difference is that the HTML actually works — no blocks, no fake pages, no rotating proxy management on your end.

Beyond the core scraping API, ScraperAPI also offers:

- **Async Scraper Service** — submit millions of jobs without timeouts; get results via webhook
- **Structured Data Endpoints** — clean JSON from Amazon, Google, Walmart, eBay, and Redfin without custom parsers
- **DataPipeline** — schedule and automate scraping workflows with no code
- **SDK support** — Python, Node.js, Ruby, PHP, Java, and Go

They process over 36 billion API requests per month and serve over 10,000 companies including Deloitte, Sony, and Alibaba. It's a mature infrastructure, not a side project.

👉 [Start your free trial — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)

---

## **Step 1: Sign Up and Get Your API Key**

Everything starts with an API key. Head to the ScraperAPI signup page, create a free account, and you'll get 1,000 free credits every month — plus a 7-day trial with 5,000 credits to properly test your actual targets at scale.

Once you're in your dashboard, your API key is on the main screen. Copy it. Every request you send to ScraperAPI will include this key as an authentication parameter.

No credit card required for the trial. If you decide it doesn't work for your use case, you've lost nothing.

👉 [Create your free ScraperAPI account](https://www.scraperapi.com/signup?fp_ref=coupons)

---

## **Step 2: Your First ScraperAPI Request**

There are six ways to make requests to ScraperAPI:

1. Via the main API endpoint (`https://api.scraperapi.com`)
2. Via the Async API endpoint (`https://async.scraperapi.com`)
3. Via proxy port (`http://scraperapi:APIKEY@proxy-server.scraperapi.com:8001`)
4. Via Structured Data Endpoints (`https://api.scraperapi.com/structured/`)
5. Via DataPipeline
6. Via the Python/Node SDK

For most tutorials, you'll start with the direct API endpoint. Here's the simplest possible Python implementation:

python
import requests
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY_HERE'
target_url = 'https://quotes.toscrape.com/page/1/'

params = {'api_key': API_KEY, 'url': target_url}
response = requests.get('http://api.scraperapi.com/', params=urlencode(params))

print(response.text)


That's it. You pass your API key and the URL you want to scrape. ScraperAPI handles the rest — proxy selection, headers, retries — and returns the HTML.

**One important note on timeouts**: ScraperAPI will spend up to 60 seconds trying different proxy and header configurations to get a successful response. If you set your timeout shorter than that, you'll cut the process short and get more failures. Either remove the timeout entirely or set it to at least 70 seconds.

---

## **Step 3: Building in Retry Logic**

Even with a high success rate, some requests will fail. ScraperAPI returns a `500` status code for failed requests and doesn't charge credits for them. Building in a retry loop is straightforward:

python
import requests
from bs4 import BeautifulSoup
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY_HERE'
NUM_RETRIES = 3
list_of_urls = [
    'http://quotes.toscrape.com/page/1/',
    'http://quotes.toscrape.com/page/2/'
]

scraped_quotes = []

for url in list_of_urls:
    params = {'api_key': API_KEY, 'url': url}
    for _ in range(NUM_RETRIES):
        try:
            response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
            if response.status_code in [200, 404]:
                break
        except requests.exceptions.ConnectionError:
            response = ''

    if response.status_code == 200:
        soup = BeautifulSoup(response.text, "html.parser")
        quotes_sections = soup.find_all('div', class_="quote")
        for block in quotes_sections:
            quote = block.find('span', class_='text').text
            author = block.find('small', class_='author').text
            scraped_quotes.append({'quote': quote, 'author': author})

print(scraped_quotes)


Three retries is enough to handle 99.9% of cases according to ScraperAPI's own data. The 404 in the `status_code` check is intentional — a proper 404 page from the target site counts as a successful request (the URL existed, the site responded).

---

## **Step 4: Scaling with Multi-Threaded Requests**

When you move from testing to production, you'll want to run requests concurrently. ScraperAPI's plan tiers define how many concurrent threads you have — the Hobby plan gives you 20, the Startup plan gives 50, Business gives 100, and so on.

Here's how to wire up concurrent scraping using Python's `concurrent.futures`:

python
import requests
from bs4 import BeautifulSoup
import concurrent.futures
import csv
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY_HERE'
NUM_RETRIES = 3
NUM_THREADS = 5

list_of_urls = [f'http://quotes.toscrape.com/page/{i}/' for i in range(1, 11)]
scraped_quotes = []

def scrape_url(url):
    params = {'api_key': API_KEY, 'url': url}
    for _ in range(NUM_RETRIES):
        try:
            response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
            if response.status_code in [200, 404]:
                break
        except requests.exceptions.ConnectionError:
            response = ''

    if response.status_code == 200:
        soup = BeautifulSoup(response.text, "html.parser")
        for block in soup.find_all('div', class_="quote"):
            scraped_quotes.append({
                'quote': block.find('span', class_='text').text,
                'author': block.find('small', class_='author').text
            })

with concurrent.futures.ThreadPoolExecutor(max_workers=NUM_THREADS) as executor:
    executor.map(scrape_url, list_of_urls)

print(scraped_quotes)


Set `NUM_THREADS` to match your plan's concurrent thread limit. Running more threads than your plan allows will just result in queued requests, not errors.

---

## **Step 5: Handling JavaScript-Heavy Pages**

Modern single-page applications render their content via JavaScript — a basic `requests.get()` call will get you an empty shell. ScraperAPI solves this with the `render=true` parameter, which fires up a headless Chromium instance to fully render the page before returning the HTML.

python
import requests
from urllib.parse import urlencode

API_KEY = 'YOUR_API_KEY_HERE'
params = {
    'api_key': API_KEY,
    'url': 'https://your-js-heavy-target.com/products',
    'render': 'true'
}

response = requests.get('http://api.scraperapi.com/', params=urlencode(params))
print(response.text)


**Credit cost note**: The `render=true` flag adds 10 credits per request on top of the base domain cost. For a standard site that normally costs 1 credit per request, you're now spending 11. For an Amazon product page (base cost 5 credits), you're spending 15. Plan accordingly.

Other useful parameters you can add:

- `country_code=us` — route through a specific country's proxies (geotargeting)
- `premium=true` — use residential proxies from a premium pool (+10 credits)
- `device_type=mobile` — spoof a mobile browser fingerprint (no extra cost)
- `session_number=123` — maintain the same IP across multiple requests in a session
- `autoparse=true` — auto-parse common HTML elements (no extra cost)

---

## **Step 6: Using the Async API for High-Volume Jobs**

The synchronous API is great for real-time scraping where you need results immediately. But when you're submitting tens of thousands of URLs — a full product catalog, a bulk SERP run, a weekly competitive intelligence sweep — the synchronous model creates bottlenecks. You're waiting on each request before sending the next.

The Async API flips this: you submit all your jobs at once, and ScraperAPI processes them in parallel and returns results to a webhook endpoint (or you poll a status URL). This massively improves throughput and success rate on difficult sites.

**Submitting a single async job:**

python
import requests

initial_request = requests.post(
    url='https://async.scraperapi.com/jobs',
    json={
        'apiKey': 'YOUR_API_KEY',
        'url': 'https://quotes.toscrape.com/'
    }
)
print(initial_request.text)
# Returns: {"id": "...", "status": "running", "statusUrl": "https://async.scraperapi.com/jobs/..."}


**Checking job status:**

python
import requests

r = requests.get("https://async.scraperapi.com/jobs/YOUR_JOB_ID")
result = r.json()

if result['status'] == 'finished':
    html_body = result['response']['body']
    print(html_body[:500])


**Submitting batch jobs (multiple URLs at once):**

python
import requests

urls = [f'https://quotes.toscrape.com/page/{i}/' for i in range(1, 11)]

batch_job = requests.post(
    url='https://async.scraperapi.com/batchjobs',
    json={
        'apiKey': 'YOUR_API_KEY',
        'urls': urls,
        'apiParams': {
            'render': False,
            'country_code': 'us'
        }
    }
)
print(batch_job.json())


The Async API shines when success rate matters more than speed — it gives ScraperAPI's system more time to retry with different proxy configurations and bypass methods. For recurring jobs, scheduled scraping, or anything that touches difficult targets, this is the recommended approach.

---

## **Step 7: Structured Data Endpoints — Skip the Parser**

Writing an HTML parser for Amazon product pages is a full-day job. Writing one that stays working when Amazon changes its page structure is an ongoing maintenance nightmare. ScraperAPI's Structured Data Endpoints (SDEs) solve this: they return clean, pre-parsed JSON for 18 endpoints across five major platforms.

| Platform | Available Endpoints |
| --- | --- |
| Amazon | Product details (by ASIN), Search results, Competitor offers |
| Google | SERP, Shopping, Maps, News, Jobs |
| Walmart | Product, Search, Category, Reviews |
| eBay | Product, Search |
| Redfin | Search, Agent Details, Rental Properties, For Sale |

**Example: Pull Amazon product data by ASIN**

python
import requests

API_KEY = 'YOUR_API_KEY'
asin = 'B08N5WRWNW'  # Replace with your target ASIN

response = requests.get(
    f'https://api.scraperapi.com/structured/amazon/product',
    params={
        'api_key': API_KEY,
        'asin': asin,
        'country': 'us'
    }
)

product = response.json()
print(product['name'])
print(product['price'])
print(product['rating'])


The response includes 18+ fields: price, title, rating, review count, Best Seller Rank, images, seller info, variants, and more. Supports 21 regional Amazon marketplaces. Same logic applies to Google SERP, Walmart, and the other platforms.

SDEs are available on all plans including the free tier.

👉 [Try Structured Data Endpoints with your free trial](https://www.scraperapi.com/signup?fp_ref=coupons)

---

## **Understanding the Credit System (Read This Before Picking a Plan)**

This is the part most tutorials skip over, and it's the part that causes people to run out of credits unexpectedly.

ScraperAPI bills by credits, not by raw requests. Every request costs credits, and the cost depends on two variables: the target domain and the parameters you enable.

**Domain-based multipliers (automatic — you can't opt out):**

| Domain Type | Credits per Request | Examples |
| --- | --- | --- |
| Standard HTML page | 1 | Blogs, news sites, most websites |
| E-commerce | 5 | Amazon, eBay, Walmart |
| Search engines | 25 | Google, Bing |
| Social media | 30 | LinkedIn |

**Parameter-based costs (optional — you control these):**

| Parameter | Extra Credits |
| --- | --- |
| `render=true` | +10 |
| `premium=true` | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` | +30 (paid plans only) |
| Cloudflare / DataDome bypass | +10 (auto-applied when detected) |
| `premium=true` + `render=true` combined | +25 (not +20) |
| `ultra_premium=true` + `render=true` combined | +75 (not +40) |

That last point is critical: combining features costs more than the sum of individual costs. It's non-linear and not prominently flagged — it's what catches people off guard.

Also worth knowing: credits do **not** roll over. They expire at the end of your billing cycle. And Pay-As-You-Go (the option to keep scraping after you hit your credit limit) is only available on the Scaling plan and above.

---

## **Full ScraperAPI Pricing Plan Comparison**

All plans include: JavaScript rendering, premium proxies, automatic retries, unlimited bandwidth, CAPTCHA solving, and a 99.9% uptime guarantee. Annual billing saves 10% across the board.

| Plan | Monthly Price | Annual (per mo) | API Credits | Concurrent Threads | Geotargeting | Pay-As-You-Go | Get Started |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Free** | $0 | — | 1,000 | 5 | No | No | [Start Free](https://www.scraperapi.com/signup?fp_ref=coupons) |
| **Hobby** | $49 | $44.10 | 100,000 | 20 | US & EU only | No | [Get Hobby](https://www.scraperapi.com/signup?fp_ref=coupons) |
| **Startup** | $149 | $134.10 | 1,000,000 | 50 | US & EU only | No | [Get Startup](https://www.scraperapi.com/signup?fp_ref=coupons) |
| **Business** | $299 | $269.10 | 3,000,000 | 100 | Global (50+ countries) | No | [Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Scaling** ⭐ | $475 | $427.50 | 5,000,000 | 200 | Global | Yes | [Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Professional** | $975 | $877.50 | 10,500,000 | 300 | Global | Yes | [Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Advanced** | $1,975 | $1,777.50 | 21,500,000 | 500 | Global | Yes | [Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| **Enterprise** | Custom | Custom | 22M+ | 500+ | Global | Yes | [Contact Sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

**Which plan is right for your actual workload?**

The key question isn't "how many credits does it say?" — it's "how many *requests* does it actually buy me for my specific targets?" Run the math with your domain multipliers first.

- **Free / Hobby** — Testing the integration, personal projects, low-volume scraping of standard websites. Hobby's 20 concurrent threads is the main constraint.
- **Startup** — The sweet spot for freelancers and small teams. 1M credits, 50 threads, US/EU geo. Enough for most recurring scraping workflows if you're not hitting Amazon or Google heavily.
- **Business** — The first tier with global geotargeting. If your project needs data from Asia-Pacific, Latin America, or anywhere outside US/EU, this is your entry point.
- **Scaling** — The most popular plan for a reason. 5M credits, 200 threads, global geo, and Pay-As-You-Go. You won't get cut off mid-cycle if you hit your limit.
- **Professional / Advanced** — For teams running continuous, business-critical pipelines where volume and concurrency are the main levers.
- **Enterprise** — Custom pricing, dedicated Slack support, and SLA guarantees. Contact sales.

---

## **Where ScraperAPI Works Well (And Where It Doesn't)**

Based on independent benchmarks from April 2026 (Scrapeway), here's how ScraperAPI performs by target:

| Target Site | Success Rate | Notes |
| --- | --- | --- |
| Zillow | 100% | Excellent for real estate data |
| Etsy | 99% | Very reliable |
| Amazon | 98% | Strong — SDEs make it even better |
| LinkedIn | 95% | Works but expensive (30 credits/req) |
| Walmart | 93% | Reliable with SDEs |
| Indeed | 90% | Usable for job data |
| Instagram | 0% | Doesn't work |
| Twitter/X | 0% | Doesn't work |
| Booking.com | 0% | Doesn't work |

The pattern is clear: ScraperAPI is strong on e-commerce, real estate, and search engines — sites that have structured, publicly accessible data. Social media platforms, login-required pages, and heavily JS-protected travel sites are weak spots. For sites requiring login, ScraperAPI's terms of service also prohibit scraping behind authentication walls.

---

## **Practical Tips Before You Run Your First Production Job**

A few things that experienced ScraperAPI users figure out the hard way:

**Check the domain cost before running at scale.** Use the Domain Multiplier tool in your dashboard, or hit the cost API endpoint directly: `https://api.scraperapi.com/account/urlcost?api_key=YOUR_KEY&url=YOUR_URL`. This tells you exactly what a given URL will cost before you burn through your credits.

**Don't auto-enable premium features.** `render=true`, `premium=true`, and `ultra_premium=true` are opt-in. ScraperAPI will not automatically enable JS rendering — you have to ask for it. Start without these flags and only add them if your success rate is low.

**Monitor your credit consumption daily for the first week.** There are no proactive usage alerts — no email or push notification when you're running low. You have to check your dashboard manually. New users often get burned by this.

**Set retries to at least 3.** ScraperAPI refunds credits on failed requests, so retries cost you nothing extra. Three retries gets you to 99.9% success on most targets.

**Use the Async API for batch jobs.** If you're submitting more than a few hundred URLs at a time, the Async API is more reliable and handles failures gracefully. Use it for any recurring or scheduled workload.

---

## **Common Questions About the ScraperAPI API Tutorial**

**Do I need my own proxies?** No. ScraperAPI manages a pool of 40 million+ residential and datacenter IPs. You don't need to buy, rotate, or manage proxies yourself.

**Does it handle CAPTCHAs?** Yes. Basic CAPTCHA solving is built in. For sites with advanced bot protection like Cloudflare or DataDome, ScraperAPI applies an additional bypass (+10 credits per request) automatically when the protection is detected.

**Can I use it with languages other than Python?** Yes. ScraperAPI supports cURL, Node.js, PHP, Ruby, Java, and Go — full documentation and code examples available in their docs.

**What if I run out of credits mid-month?** On Hobby, Startup, and Business plans: you're cut off until the billing cycle resets (or you upgrade). On Scaling and above: Pay-As-You-Go kicks in and you keep scraping at a fixed per-credit rate.

**Is there a refund policy?** Yes — 7-day no-questions-asked refund policy. Contact support if you're unhappy for any reason.

---

## **Getting Started Is Genuinely Simple**

There's a lot of conceptual territory in a ScraperAPI API tutorial — credits, multipliers, async vs sync, structured endpoints, proxy modes — but the actual first step is three lines of Python and an API key.

Start with the free trial. Test your actual target URLs. Check the credit costs using the Domain Multiplier. Run the math for your monthly volume. Then pick a plan based on what you'll realistically spend, not the headline credit number.

The infrastructure does what it promises. The setup takes minutes. The only thing that trips people up is the credit math — and now you've got the full picture.

👉 [Start your free 7-day trial — 5,000 credits, no credit card required](https://www.scraperapi.com/?fp_ref=coupons)
