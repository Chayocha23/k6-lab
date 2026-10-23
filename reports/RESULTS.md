## Step 0.1: HTTP Request Duration

- **`http_req_duration`:** 283.16 ms

## Step 0.2: Request Rate and p(95)

- **Total requests:** 77
- **Request rate:** 3.63 requests/second
- **p(95):** 293.54 ms

**Why the rate stays below 5 requests per second:**

Although 5 virtual users run for 20 seconds, each iteration includes a 1-second think time (`sleep(1)`) plus the HTTP response time (approximately 289 ms). Therefore, each iteration takes around 1.32 seconds on average, meaning each virtual user completes fewer than one request per second. As a result, the overall request rate stays below 5 requests per second.