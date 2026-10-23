## Step 0.1: HTTP Request Duration

- **`http_req_duration`:** 283.16 ms

## Step 0.2: Request Rate and p(95)

- **Total requests:** 77
- **Request rate:** 3.63 requests/second
- **p(95):** 293.54 ms

**Why the rate stays below 5 requests per second:**

Although 5 virtual users run for 20 seconds, each iteration includes a 1-second think time (`sleep(1)`) plus the HTTP response time (approximately 289 ms). Therefore, each iteration takes around 1.32 seconds on average, meaning each virtual user completes fewer than one request per second. As a result, the overall request rate stays below 5 requests per second.

## Step 0.3: Fixed Iterations

- **Total iterations:** 20
- **Virtual users (VUs):** 5
- **Total execution time:** 9.2 seconds
- **Average iteration duration:** 1.79 seconds

The 20 iterations are shared among 5 virtual users. The test finishes in 9.2 seconds, with an average iteration duration of 1.79 seconds.