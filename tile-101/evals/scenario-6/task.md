# Build a Custom API Client with Defaults

Create a reusable API client that wraps HTTP functionality with preset configurations for a specific API service.

## Requirements

Your API client should:

1. Create a configured client with default settings (base URL, headers, timeout)
2. Allow individual requests to override defaults
3. Support all HTTP methods (GET, POST, PUT, DELETE) with the defaults
4. Share common configuration like authentication headers across all requests
5. Create multiple client instances with different configurations

## Use Cases

- Create an API client for a specific service with base URL and auth headers
- Make requests without repeating common configuration
- Override defaults on a per-request basis when needed
- Maintain separate clients for different API environments (dev, staging, prod)

## Example Usage

```javascript
const ApiClient = require('./api-client');

// Create client with defaults
const client = ApiClient.create({
  baseUrl: 'https://jsonplaceholder.typicode.com',
  headers: {
    'User-Agent': 'MyApp/1.0',
    'X-API-Key': 'secret-key'
  },
  timeout: 5000,
  json: true
});

// Use the client - defaults are automatically applied
client.get('/users/1', (err, user) => {
  console.log('User:', user);
});

client.post('/users', {
  body: { name: 'New User', email: 'user@example.com' }
}, (err, created) => {
  console.log('Created:', created);
});

// Override defaults for specific request
client.get('/posts/1', {
  timeout: 10000  // Override default timeout
}, (err, post) => {
  console.log('Post:', post);
});
```

## Expected Behavior

- All requests should automatically include the default configuration
- The base URL should be prepended to all relative paths
- Default headers should be sent with every request
- Individual requests can override any default setting

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with support for creating configured instances with default options.

## Test Cases

### Test 1: Defaults Applied @test

Input: Client with base URL and headers, make GET request to relative path

Expected behavior: Should prepend base URL and include default headers automatically

### Test 2: Multiple Methods @test

Input: Use GET, POST, PUT methods on the same client instance

Expected behavior: All methods should inherit the default configuration

### Test 3: Override Defaults @test

Input: Request with custom timeout that differs from client default

Expected behavior: Should use the custom timeout while keeping other defaults
