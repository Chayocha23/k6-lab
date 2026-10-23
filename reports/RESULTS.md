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

## Step 0.4: Pizza API

- **Total requests:** 1
- **HTTP request duration:** 303.78 ms
- **Average iteration duration:** 1.92 seconds
- **Checks passed:** 3/3 (100%)
- **Checks failed:** 0/3 (0%)

The test sends one HTTP request to the Pizza API and verifies three conditions: the response status is 200, the pizza has a name, and every ingredient is vegetarian. All three checks pass, and the HTTP request completes in 303.78 ms. The entire iteration takes 1.92 seconds.

## Step 0.5: A Request That Must Be Refused

- **Total requests:** 1
- **HTTP status:** 401 Unauthorized
- **HTTP request duration:** 282.72 ms
- **Checks passed:** 1/1 (100%)
- **Checks failed:** 0/1 (0%)
- **HTTP request failure rate:** 0.00%
- **Average iteration duration:** 1.89 seconds

The test sends a POST request to the Pizza API without a token and expects a 401 Unauthorized response. The check passes because the API correctly refuses the request. The `http.expectedStatuses(401)` configuration tells k6 to treat status 401 as an expected response, so `http_req_failed` remains at 0.00%. This demonstrates a basic security test using a performance testing tool.