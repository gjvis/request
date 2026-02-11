# Build a Redirect Analyzer Tool

Create a Node.js tool that analyzes HTTP redirects and provides control over redirect behavior.

## Requirements

Your tool should provide functions to:

1. Follow redirects automatically and return the final destination
2. Disable redirect following to get the initial redirect response
3. Limit the maximum number of redirects to prevent infinite loops
4. Track the redirect chain (all intermediate URLs visited)
5. Implement custom redirect logic based on response data

## Use Cases

- **Auto-follow mode**: Follow all redirects and return final content
- **No-follow mode**: Get the redirect response without following
- **Limited follows**: Set maximum redirect count (e.g., 5)
- **Custom validation**: Only follow redirects to specific domains

## Example Usage

```javascript
const analyzer = require('./redirect-analyzer');

// Auto-follow redirects
analyzer.followRedirects('https://httpbin.org/redirect/3', (err, data) => {
  console.log('Final URL:', data.finalUrl);
  console.log('Redirect count:', data.redirectCount);
});

// Don't follow redirects
analyzer.noFollow('https://httpbin.org/redirect/1', (err, response) => {
  console.log('Location header:', response.headers.location);
  console.log('Status code:', response.statusCode);
});

// Limit redirects
analyzer.limitedFollow('https://httpbin.org/redirect/10', 5, (err, data) => {
  console.log('Result:', data); // Should error after 5 redirects
});
```

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with configurable redirect handling and tracking capabilities.

## Test Cases

### Test 1: Auto-follow Redirects @test

Input: URL with 3 redirects

Expected behavior: Should follow all redirects and return the final destination content

### Test 2: Disable Following @test

Input: URL with redirects, following disabled

Expected behavior: Should return 3xx status code and Location header without following

### Test 3: Max Redirects @test

Input: URL with 10 redirects, max set to 5

Expected behavior: Should stop after 5 redirects and return an error or partial result
