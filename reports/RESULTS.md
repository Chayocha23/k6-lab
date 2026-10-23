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

## Step 0.6: Thresholds

### Run 1: Default Thresholds

- **Virtual users:** 3
- **Test duration:** 30 seconds
- **Total HTTP requests:** 67
- **Request rate:** 2.14 requests/second
- **HTTP request duration (p95):** 392.68 ms
- **HTTP request failure rate:** 0.00%
- **Checks passed:** 100% (134/134)
- **Thresholds:** 3/3 passed
- **Exit code:** 0

All three thresholds pass. The p(95) response time of 392.68 ms is below the default limit of 1,500 ms. The HTTP request failure rate is 0.00%, and all checks pass.

### Run 2: Stricter Response-Time Threshold

Command: `k6 run -e P95=300 06-thresholds.js`

- **Virtual users:** 3
- **Test duration:** 30 seconds
- **Total HTTP requests:** 52
- **Request rate:** 1.67 requests/second
- **HTTP request duration (p95):** 388.81 ms
- **HTTP request failure rate:** 1.92%
- **Checks passed:** 98.07% (102/104)
- **Thresholds:** 0/3 passed
- **Exit code:** 99

The test fails because all three thresholds are violated. The p(95) response time of 388.81 ms exceeds the 300 ms limit, the HTTP request failure rate of 1.92% exceeds the 1% limit, and the check pass rate of 98.07% is below the required 99%.

During the second run, a connection error occurred while sending a request to the Pizza API. This caused a request failure and a script exception when the test attempted to parse a null response body as JSON. Therefore, the result includes a connection problem in addition to the stricter response-time threshold being violated.