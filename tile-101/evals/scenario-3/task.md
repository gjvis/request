# Build a Session-Aware HTTP Client

Create a Node.js client that maintains cookies across multiple HTTP requests to simulate a logged-in user session.

## Requirements

Your client should:

1. Create and maintain a cookie storage container
2. Make an initial request to get session cookies (e.g., login)
3. Automatically send stored cookies with subsequent requests
4. Support multiple sequential requests using the same session
5. Provide a way to inspect stored cookies for a given URL

## Scenario

Simulate a user workflow:
1. Visit a login page and receive session cookies
2. Make authenticated requests using those cookies
3. Access protected resources that require the session

## Example Usage

```javascript
const sessionClient = require('./session-client');

const client = sessionClient.create();

// Step 1: Login and get cookies
client.login('https://httpbin.org/cookies/set?session=abc123', (err, result) => {
  console.log('Logged in, cookies stored');

  // Step 2: Make authenticated request
  client.get('https://httpbin.org/cookies', (err, cookies) => {
    console.log('Current cookies:', cookies);
  });

  // Step 3: Get stored cookies
  const stored = client.getCookies('https://httpbin.org');
  console.log('Stored cookies:', stored);
});
```

## Expected Behavior

- Cookies received from the first request should automatically be sent with subsequent requests
- The cookie container should persist across multiple requests
- Cookies should be domain-specific and follow HTTP cookie rules

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with built-in cookie jar management for maintaining session state.

## Test Cases

### Test 1: Cookie Storage @test

Input: Request to endpoint that sets cookies

Expected behavior: Should store cookies automatically for subsequent requests

### Test 2: Cookie Transmission @test

Input: Second request to same domain

Expected behavior: Should automatically include previously stored cookies

### Test 3: Cookie Retrieval @test

Input: Call to get stored cookies for a URL

Expected behavior: Should return array of cookie objects or formatted cookie string
