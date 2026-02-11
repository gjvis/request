# Build a Simple HTTP Status Checker

Create a Node.js script that checks the HTTP status of multiple URLs and reports their availability.

## Requirements

Your script should:

1. Accept an array of URLs to check
2. Make HTTP GET requests to each URL
3. Report the status code and response time for each URL
4. Handle errors gracefully (network errors, timeouts, etc.)
5. Display results in a readable format showing:
   - URL
   - Status code (or "ERROR" if request failed)
   - Whether the site is up (2xx status codes)
   - Any error messages

## Example Usage

```javascript
const checker = require('./status-checker');

const urls = [
  'https://www.google.com',
  'https://httpbin.org/status/404',
  'https://httpbin.org/delay/2'
];

checker.checkStatus(urls, (results) => {
  console.log(results);
});
```

## Expected Output Format

```
Status Check Results:
--------------------
URL: https://www.google.com
Status: 200
Up: Yes

URL: https://httpbin.org/status/404
Status: 404
Up: No

URL: https://httpbin.org/delay/2
Status: 200
Up: Yes
```

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client for Node.js that makes it easy to make HTTP calls.

## Test Cases

### Test 1: Successful Request @test

Input: `['https://httpbin.org/status/200']`

Expected behavior: Should return status 200 and mark as "Up: Yes"

### Test 2: Not Found Error @test

Input: `['https://httpbin.org/status/404']`

Expected behavior: Should return status 404 and mark as "Up: No"

### Test 3: Multiple URLs @test

Input: `['https://httpbin.org/status/200', 'https://httpbin.org/status/500']`

Expected behavior: Should check both URLs and return their respective statuses (200 as up, 500 as down)
