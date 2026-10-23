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

## Step 0.7: From a Requirement to Thresholds

### Run 1: Original Requirement

**Requirement:** With 3 users sending requests concurrently for 30 seconds, the API must answer 95% of requests within 1 second, 99% within 1.5 seconds, and have a request failure rate below 1%.

- **Virtual users:** 3
- **Test duration:** 30 seconds
- **Total HTTP requests:** 66
- **p(95):** 392.4 ms — PASS (`p(95)<1000`)
- **p(99):** 397.48 ms — PASS (`p(99)<1500`)
- **HTTP request failure rate:** 0.00% — PASS (`rate<0.01`)
- **Exit code:** 0

All three performance requirements pass. The measured p(95) and p(99) are both below their respective limits, and no HTTP requests fail.

### Run 2: Stricter p(95) Requirement

**Changed requirement:** The API must answer 95% of requests within 300 ms instead of 1 second. The other requirements remain unchanged.

- **Virtual users:** 3
- **Test duration:** 30 seconds
- **Total HTTP requests:** 66
- **p(95):** 400.83 ms — FAIL (`p(95)<300`)
- **p(99):** 408.34 ms — PASS (`p(99)<1500`)
- **HTTP request failure rate:** 0.00% — PASS (`rate<0.01`)
- **Exit code:** 99

The test fails because the measured p(95) of 400.83 ms exceeds the stricter 300 ms limit. The p(99) and HTTP request failure rate requirements still pass. k6 exits with code 99 because the `http_req_duration` threshold is violated.

## Step 0.8: Bug Hunt Checkpoint

### Investigation

The `explore.js` script was used to inspect the API's available ingredients and tools. The API returned ingredient names such as `Pepperoni` and tool names such as `Knife`.

### Verification After Fix

- **Virtual users:** 5
- **Test duration:** 20 seconds
- **Total HTTP requests:** 74
- **Request rate:** 3.49 requests/second
- **HTTP request duration (p95):** 405.66 ms
- **HTTP request failure rate:** 0.00%
- **Checks passed:** 100% (222/222)
- **Checks failed:** 0 (0%)

All three checks passed: the response status was 200, the pizza contained no pepperoni, and it did not require a knife. The test results confirm that all 74 responses satisfied the defined checks during this run.

### Why Testing Under Load Helps

A single manual request may return a pizza that satisfies the customer's requirements by chance. Repeated requests allow the checks to examine many recommendations and increase the chance of detecting results that contain excluded ingredients or require excluded tools. In this run, all 74 responses satisfied the checks.

