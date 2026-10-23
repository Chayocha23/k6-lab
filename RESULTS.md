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

## Part A: A User Journey and the Load Test Types

### Step A1: The User Journey and a Smoke Test

**Test objective:** Verify that one user can complete the QuickPizza journey successfully before running larger load tests.

The `journey.js` script defines a user journey with three groups:

1. **Home:** Open the home page and verify that the response status is 200.
2. **Quotes:** Retrieve the quotes and verify that the response is successful and contains at least one quote.
3. **Recommendation:** Request a vegetarian pizza and verify that the response status is 200 and the pizza has a name.

The `a1-smoke.js` script runs this journey with one virtual user (VU) for 30 seconds. The test also defines thresholds requiring a check success rate above 99% and an HTTP request failure rate below 1%.

**Test results:**

| Metric | Result |
|---|---:|
| Virtual users | 1 |
| Configured test duration | 30 seconds |
| Completed iterations | 5 |
| Total HTTP requests | 15 |
| Checks passed | 20/20 (100%) |
| Checks failed | 0/20 (0%) |
| HTTP request failure rate | 0.00% |
| Average HTTP request duration | 469.25 ms |
| 95th percentile HTTP request duration (p(95)) | 1.13 s |
| Maximum HTTP request duration | 1.21 s |
| Thresholds | All passed |

**Conclusion:**

The smoke test passed. All 20 checks succeeded, the HTTP request failure rate was 0%, and both configured thresholds were satisfied. The results indicate that the user journey worked successfully under this light-load scenario, so the test can proceed to larger load tests.

The run completed five iterations and made 15 HTTP requests, corresponding to three requests per journey. The reported total runtime was approximately 33.5 seconds, including the configured 30-second execution period and shutdown.

### Step A2: The Average-Load Test and an HTML Report

**Test objective:** Evaluate the QuickPizza user journey under an average load of up to three virtual users and generate an HTML report for reviewing performance throughout the test.

The `a2-average.js` script reuses the journey defined in `journey.js`. The test ramps up to three virtual users over 30 seconds, maintains three users for two minutes, and then ramps down to zero users over 30 seconds.

The test defines four thresholds:
- Overall check success rate must be greater than 99%.
- HTTP request failure rate must be below 1%.
- The home page's p(95) response time must be below 1,000 ms.
- The recommendation endpoint's p(95) response time must be below 1,500 ms.

**Test results:**

| Metric | Result |
|---|---:|
| Maximum virtual users | 3 |
| Total test duration | Approximately 3 minutes |
| Completed iterations | 79 |
| Total HTTP requests | 237 |
| Request rate | 1.31 requests/second |
| Checks passed | 316/316 (100%) |
| Checks failed | 0/316 (0%) |
| HTTP request failure rate | 0.00% |
| Average HTTP request duration | 347.96 ms |
| 95th percentile HTTP request duration (p(95)) | 420.38 ms |
| Maximum HTTP request duration | 1.7 s |
| Home page p(95) | 969.45 ms |
| Recommendation p(95) | 428.71 ms |
| Thresholds | All passed |

**Conclusion:**

The average-load test passed all four configured thresholds. All 316 checks succeeded, and none of the 237 HTTP requests failed. The home page's p(95) response time was 969.45 ms, below its 1,000 ms threshold, while the recommendation endpoint's p(95) was 428.71 ms, below its 1,500 ms threshold.

The results indicate that the user journey met the defined performance requirements under a load of up to three virtual users during this run.

The command was configured to export an HTML report to `reports/a2-average.html`. Verify that the report file exists and opens successfully before considering the report deliverable complete.

### Step A3: A Stress Test

**Test objective:** Evaluate the QuickPizza user journey under a higher load of up to five virtual users and compare its response-time performance with the average-load test in Step A2.

The `a3-stress.js` script is based on `a2-average.js`. It ramps up to five virtual users over 30 seconds, maintains five users for two minutes, and then ramps down to zero users over 30 seconds. The test uses the same thresholds as Step A2 to check overall test quality, HTTP request failures, and response times for the home page and recommendation endpoint.

**Test results:**

| Metric | Result |
|---|---:|
| Maximum virtual users | 5 |
| Completed iterations | 128 |
| Total HTTP requests | 384 |
| Request rate | 2.08 requests/second |
| Checks passed | 510/512 (99.60%) |
| Checks failed | 2/512 (0.39%) |
| HTTP request failure rate | 0.26% (1/384) |
| Average HTTP request duration | 355.78 ms |
| 95th percentile HTTP request duration (p(95)) | 401.12 ms |
| Maximum HTTP request duration | 10.28 s |
| Home page p(95) | 300.5 ms |
| Recommendation p(95) | 410.02 ms |
| Thresholds | All passed |

**Comparison with Step A2:**

The request rate increased from 1.31 requests/second in Step A2 to 2.08 requests/second in Step A3. The overall p(95) response time decreased slightly from 420.38 ms to 401.12 ms. However, the maximum response time increased from 1.7 seconds to 10.28 seconds, and the HTTP request failure rate increased from 0.00% to 0.26%.

**Conclusion:**

All configured thresholds passed during the stress test. However, one HTTP request failed, causing two checks related to the recommendation response to fail. The maximum response time of 10.28 seconds also indicates a slow outlier, even though the overall p(95) remained below the endpoint-specific thresholds. These results show why response-time percentiles, maximum response time, request failures, and checks should be considered together when evaluating performance under higher load.

### Step A4: A Spike Test

**Test objective:** Observe how the QuickPizza application responds to a sudden increase in virtual users and determine whether response times rise during the spike and how long they take to return to their earlier level.

The `a4-spike.js` script uses six stages:
1. Maintain 1 virtual user for 30 seconds.
2. Ramp up to 5 virtual users over 5 seconds.
3. Maintain 5 virtual users for 30 seconds.
4. Ramp down to 1 virtual user over 5 seconds.
5. Maintain 1 virtual user for 1 minute.
6. Ramp down to 0 users over 10 seconds.

**Test results:**

| Metric | Result |
|---|---:|
| Maximum virtual users | 5 |
| Total test duration | Approximately 2 minutes 20 seconds |
| Completed iterations | 49 |
| Total HTTP requests | 147 |
| Request rate | 1.04 requests/second |
| Checks passed | 196/196 (100%) |
| Checks failed | 0/196 (0%) |
| HTTP request failure rate | 0.00% |
| Average HTTP request duration | 323.26 ms |
| 95th percentile HTTP request duration (p(95)) | 392.87 ms |
| Maximum HTTP request duration | 1.14 s |
| Home page p(95) | 297.97 ms |
| Recommendation p(95) | 413.27 ms |
| Thresholds | All passed |

**Spike and recovery analysis:**

The test was designed to introduce a sudden increase from 1 to 5 virtual users between approximately 30 and 35 seconds into the run. The load remained at 5 users for 30 seconds before dropping back to 1 user between approximately 65 and 70 seconds.

The terminal summary shows an overall p(95) response time of 392.87 ms and a maximum response time of 1.14 seconds. However, these aggregate values do not establish exactly how response time changed during the spike or how long recovery took.

Use the timeline in `reports/a4-spike.html` to identify the response-time change during the spike and when it returns to approximately its pre-spike level.

**Conclusion:**

All configured thresholds passed, all 196 checks succeeded, and none of the 147 HTTP requests failed. The overall results met the defined test requirements. The response-time impact and recovery period during the spike should be determined from the HTML report's time-series graphs.

## Part A results

| Step | Test type | Users | Req/s | p(95) | Max | Failed | Checks |
|---|---|---:|---:|---:|---:|---:|---:|
| A1 | Smoke | 1 | 0.45 | 1.13s | 1.21s | 0.00% | 100.00% |
| A2 | Average load | 3 | 1.31 | 420.38ms | 1.7s | 0.00% | 100.00% |
| A3 | Stress | 5 | 2.08 | 401.12ms | 10.28s | 0.26% | 99.60% |
| A4 | Spike | 5 | 1.04 | 392.87ms | 1.14s | 0.00% | 100.00% |
| A5 | Soak | 2 | | | | | |

