# Build a Performance Monitoring HTTP Client

Create a Node.js client that measures request performance and handles timeout scenarios.

## Requirements

Your monitoring client should:

1. Measure and report the elapsed time for each request
2. Set timeout limits for requests
3. Handle timeout errors appropriately
4. Track timing metrics for multiple requests
5. Provide performance statistics (min, max, average response times)

## Use Cases

- Monitor API response times
- Set SLA-based timeout limits
- Detect slow endpoints
- Generate performance reports

## Example Usage

```javascript
const perfClient = require('./perf-client');

// Single timed request
perfClient.timedRequest('https://httpbin.org/delay/2', (err, result) => {
  console.log('Response time:', result.elapsedTime, 'ms');
  console.log('Status:', result.statusCode);
});

// Request with timeout
perfClient.requestWithTimeout('https://httpbin.org/delay/10', 5000, (err, result) => {
  if (err && err.code === 'ETIMEDOUT') {
    console.log('Request timed out after 5 seconds');
  }
});

// Monitor multiple endpoints
perfClient.monitorEndpoints([
  'https://httpbin.org/delay/1',
  'https://httpbin.org/delay/2',
  'https://httpbin.org/delay/3'
], (err, stats) => {
  console.log('Average response time:', stats.average);
  console.log('Slowest endpoint:', stats.slowest);
});
```

## Timing and Timeout Behavior

- Timing should measure full request-response cycle
- Timeout should abort the request if it exceeds the limit
- Timeout errors should be distinguishable from other errors
- Elapsed time should be accessible in the response

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with built-in timing measurement and configurable timeout handling.

## Test Cases

### Test 1: Measure Elapsed Time @test

Input: Request to endpoint with known delay

Expected behavior: Should return elapsed time in milliseconds, approximately matching the delay

### Test 2: Timeout Handling @test

Input: Request to slow endpoint with short timeout

Expected behavior: Should abort request and return ETIMEDOUT or ESOCKETTIMEDOUT error

### Test 3: Performance Stats @test

Input: Multiple requests to different endpoints

Expected behavior: Should collect timing data for all requests and calculate statistics
