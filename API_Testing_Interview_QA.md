# API Testing Interview Questions & Answers

---

## 1. Fundamentals

**Q: What is API testing and how is it different from UI testing?**
API testing verifies the business logic, data response, security, and performance at the API/service layer — without a UI. It's faster, more stable, and catches issues earlier since it doesn't depend on rendering. UI testing verifies the front-end behavior and user experience, and is slower and more brittle because it depends on layout/rendering changes.

**Q: Types of APIs (REST, SOAP, GraphQL) — key differences?**
- **REST**: Uses HTTP methods, resource-based URLs, lightweight (JSON/XML), stateless.
- **SOAP**: XML-based protocol, strict contract (WSDL), built-in error handling and security standards, heavier.
- **GraphQL**: Single endpoint, client specifies exactly what data it needs (no over-fetching/under-fetching), strongly typed schema.

**Q: HTTP methods and when to use each**
- **GET** – Retrieve data, no body, safe & idempotent.
- **POST** – Create a new resource, not idempotent.
- **PUT** – Replace/update an entire resource, idempotent.
- **PATCH** – Partially update a resource, may or may not be idempotent.
- **DELETE** – Remove a resource, idempotent.

**Q: Difference between PUT and PATCH?**
PUT replaces the whole resource (you must send the full object); PATCH updates only the specified fields. Calling PUT twice with the same payload gives the same result (idempotent); PATCH depends on the operation (e.g., "increment counter" is not idempotent).

**Q: Common HTTP status codes**
- **200** OK – success
- **201** Created – resource created (after POST)
- **204** No Content – success, nothing to return (after DELETE)
- **400** Bad Request – malformed/invalid input
- **401** Unauthorized – missing/invalid authentication
- **403** Forbidden – authenticated but not allowed
- **404** Not Found – resource doesn't exist
- **409** Conflict – e.g., duplicate resource
- **422** Unprocessable Entity – validation error
- **500** Internal Server Error – server-side failure
- **502** Bad Gateway – invalid response from upstream server
- **503** Service Unavailable – server overloaded/down

**Q: What is idempotency? Which methods are idempotent?**
An idempotent operation gives the same result no matter how many times it's called. GET, PUT, DELETE, HEAD are idempotent. POST and PATCH are generally not.

**Q: REST vs SOAP?**
| REST | SOAP |
|---|---|
| Architectural style | Protocol |
| JSON/XML | XML only |
| Faster, lightweight | Heavier, strict standards |
| Stateless | Can be stateful |
| No built-in security standard | Built-in WS-Security |

**Q: REST principles**
Statelessness (no client session stored on server), client-server separation, cacheable responses, uniform interface (resources identified by URIs), layered system, resource-based design.

**Q: Structure of an HTTP request/response**
- **Request**: Method + URL, Headers (Content-Type, Authorization, etc.), Body (payload), Query/Path params.
- **Response**: Status line (code + message), Headers, Body (data returned).

**Q: Path Parameter vs Query Parameter?**
- **Path parameter**: Part of the URL path, identifies a specific resource. E.g., `/users/{id}` → `/users/101`
- **Query parameter**: Appended after `?`, used for filtering/sorting/pagination. E.g., `/users?status=active&page=2`

---

## 2. Tool-Specific (Postman / REST Assured)

**Q: How do you write test scripts in Postman?**
Using the "Tests" tab with `pm.test()` and `pm.expect()`:
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has userId", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.userId).to.eql(1);
});
```

**Q: How do you chain requests in Postman?**
Extract a value from one response (e.g., an auth token or ID) and save it as an environment/global variable, then reference it in the next request using `{{variableName}}`.
```javascript
var jsonData = pm.response.json();
pm.environment.set("authToken", jsonData.token);
```

**Q: What is Collection Runner / Newman?**
Collection Runner executes a whole collection of requests in sequence, often with a data file (CSV/JSON) to run the same requests with different data sets. Newman is the CLI tool to run Postman collections — used to integrate API tests into CI/CD pipelines.

**Q: How do you handle authentication in Postman?**
Postman supports Basic Auth, Bearer Token, OAuth 2.0, API Key, etc., directly in the "Authorization" tab, or manually by setting the `Authorization` header. For OAuth2, Postman can even fetch and auto-refresh the access token.

**Q: Pre-request scripts — example use case?**
Scripts that run *before* the request is sent — commonly used to generate a timestamp, create a dynamic signature/hash, or fetch a fresh auth token so it's ready before the actual API call executes.

**Q: How do you parameterize data in Postman?**
Using a CSV or JSON data file with the Collection Runner — each row becomes one test iteration, and values are referenced as `{{columnName}}` in the request.

**Q: REST Assured — validating status code, body, headers**
```java
given()
    .header("Content-Type", "application/json")
.when()
    .get("/users/1")
.then()
    .statusCode(200)
    .header("Content-Type", equalTo("application/json"))
    .body("name", equalTo("John"));
```

**Q: How do you extract a value from one response to use in the next request?**
Parse the JSON response (using JsonPath in REST Assured, or `pm.response.json()` in Postman), store it in a variable, and pass it into the next request's headers/body/params.

---

## 3. Real-Time / Scenario-Based

**Q: You get a 500 error intermittently — how do you debug it?**
1. Check server/application logs for the stack trace at that timestamp.
2. Try to reproduce with the exact same payload/headers.
3. Check if it correlates with load (possible race condition or DB timeout).
4. Check if it's environment-specific (works in QA, fails in staging).
5. Check third-party/downstream service dependency failures.
6. Isolate: is it data-specific (a particular record causing it)?

**Q: How do you test an API that depends on another API that isn't ready yet?**
Use **mocking/stubbing** — tools like WireMock, Postman Mock Server, or MockServer to simulate the expected response, so you can test your own service's logic independently of the dependency.

**Q: How would you test a login API?**
- Valid username/password → 200 + token
- Invalid password → 401
- Non-existent username → 401/404 (without leaking which one is wrong — security)
- Empty/missing fields → 400
- SQL injection / script injection attempts → should be sanitized, not break the system
- Account locked after N failed attempts
- Token expiry and refresh flow
- Case sensitivity of username/email
- Special characters and long strings in password
- Concurrent login sessions (single-session vs multi-session policy)

**Q: How do you validate a large JSON response?**
Rather than checking each field manually, use **JSON Schema validation** to confirm structure, required fields, and data types, then spot-check specific business-critical values separately.

**Q: How would you test pagination in an API?**
- Default page/limit values
- First page, last page
- Page = 0 or negative
- Page number beyond available data (should return empty, not error)
- Page size boundary (min/max limits, e.g., limit=0, limit=10000)
- Total count/record consistency across pages
- Sorting consistency across pages (no duplicate/missing records)

**Q: How do you test API rate limiting?**
Send requests exceeding the defined limit within the time window, and verify you get a `429 Too Many Requests` response along with correct `Retry-After` or rate-limit headers (`X-RateLimit-Remaining`, etc.).

**Q: How do you test file upload APIs?**
- Valid file types and sizes
- Invalid/unsupported file types
- Oversized files (boundary + beyond limit)
- Empty file
- Corrupted file
- Duplicate filename handling
- Special characters in filename

**Q: API response time is slow — how do you flag it?**
Add response-time assertions in your test scripts (e.g., `pm.expect(pm.response.responseTime).to.be.below(2000)`), log/flag it, and escalate to performance testing (JMeter/Gatling) if it's a systemic issue rather than a one-off.

**Q: How do you handle flaky tests in API automation?**
- Add proper waits/retries instead of hardcoded sleep
- Ensure test data isolation (don't reuse data across parallel tests)
- Check for race conditions or async processing delays
- Make assertions less brittle (avoid checking full response order if not guaranteed)
- Rerun failed tests once automatically in CI, but investigate root cause rather than just retrying blindly

**Q: How do you test negative scenarios for a REST API?**
- Missing mandatory fields
- Wrong data types (string instead of int)
- Invalid/expired/missing auth token
- Unsupported HTTP method on an endpoint (e.g., DELETE on a read-only resource)
- Boundary/length violations (too long a string, negative numbers where not allowed)
- Invalid JSON structure (malformed body)

**Q: Functional vs non-functional testing of an API?**
- **Functional**: Does the API do what it's supposed to do — correct response, correct status code, correct data.
- **Non-functional**: Performance (response time under load), security (auth, injection), scalability, reliability.

**Q: How do you ensure test data doesn't clash when running tests in parallel?**
- Generate unique test data per run (timestamps, UUIDs)
- Use isolated test environments/databases per test suite
- Clean up (teardown) created data after each test
- Avoid shared mutable state between parallel test threads

---

## 4. Security & Auth

**Q: What is OAuth 2.0? Explain the flow briefly.**
OAuth 2.0 is an authorization framework that lets a client get limited access to a resource on behalf of a user without sharing credentials. Typical flow: client redirects user to authorization server → user logs in and grants consent → authorization server returns an authorization code → client exchanges the code for an access token → client uses the token to call the API.

**Q: Authentication vs Authorization?**
- **Authentication**: Verifying *who* you are (login, credentials).
- **Authorization**: Verifying *what* you're allowed to do (permissions/roles) after being authenticated.

**Q: How do you test for SQL Injection / XSS in APIs?**
Send payloads like `' OR '1'='1` or `<script>alert(1)</script>` in input fields and verify the API sanitizes/rejects them rather than executing them or reflecting them unescaped in the response.

**Q: What is JWT? How do you verify it in tests?**
JWT (JSON Web Token) is a compact, self-contained token with three parts: header, payload, signature (base64-encoded, separated by dots). In tests, you can decode the payload to verify claims (expiry, user ID, roles) and confirm the signature is valid using the secret/public key.

**Q: How do you handle API keys/secrets securely in test automation?**
Never hardcode them in the codebase. Use environment variables, a secrets manager (Vault, AWS Secrets Manager), or CI/CD pipeline secret variables, and keep them out of version control (`.gitignore`).

---

## 5. Framework / Automation Design (Experienced Roles)

**Q: How do you design an API automation framework from scratch?**
- Choose a language/tool (REST Assured + TestNG/JUnit, or Postman/Newman)
- Layered structure: config layer (base URLs, env), request/payload builders, reusable API client methods, test layer, utilities (JSON schema validation, data generation)
- Externalize test data (JSON/CSV/Excel)
- Add reporting (Extent/Allure) and logging
- Integrate with CI/CD

**Q: How do you handle environment-specific configs?**
Maintain separate config files (`dev.properties`, `qa.properties`, `staging.properties`) or environment variables, and load the right one based on a flag/parameter passed at runtime (e.g., `-Denv=qa`).

**Q: How do you integrate API tests into CI/CD?**
Add a build step (Jenkins/GitHub Actions) that runs the test suite (e.g., `mvn test` or `newman run collection.json`) after deployment to an environment, and fail the pipeline if tests fail, with reports published as build artifacts.

**Q: Schema validation / contract testing — Pact?**
JSON Schema validation ensures a response matches an expected structure/type. **Pact** is a consumer-driven contract testing tool — the consumer defines its expectations of the API (a "contract"), and the provider verifies it can satisfy that contract, catching breaking changes before deployment.

**Q: How do you generate test reports?**
Using reporting libraries like **Extent Reports** or **Allure**, which generate HTML dashboards showing pass/fail counts, execution time, logs, and screenshots/attachments per test.

**Q: How would you test an API with 100+ endpoints efficiently?**
- Prioritize by business criticality (core flows first: auth, payments, orders)
- Group endpoints by module and build reusable request builders
- Use data-driven testing to cover variations without duplicating test code
- Automate regression-prone areas first, leave rarely-changing/low-risk endpoints for smoke-level checks

---

## 6. Tricky Conceptual Questions

**Q: Status code is 200 but response body has wrong data — pass or fail?**
**Fail.** Status code alone only confirms the request was processed successfully — it says nothing about data correctness. Always validate the response body/business logic, not just the status code.

**Q: Can two different HTTP methods point to the same endpoint but do different things?**
Yes. E.g., `/users/1` — GET retrieves the user, PUT updates the user, DELETE removes the user. Same URI, different actions based on method.

**Q: API testing vs API monitoring?**
- **API testing**: Done during development/QA cycles to validate functionality before release.
- **API monitoring**: Continuous, done in production, to track uptime, response time, and error rates in real time, alerting when something breaks live.

---

*Tip: For scenario-based questions, structure your answer as: (1) what you'd check first, (2) how you'd isolate the cause, (3) how you'd validate the fix. Interviewers are testing your approach, not just the "correct" answer.*
