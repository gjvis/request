# Build an OAuth 1.0 API Client

Create a Node.js client that authenticates with an API using OAuth 1.0 signature-based authentication.

## Requirements

Your OAuth client should:

1. Sign requests using OAuth 1.0 protocol
2. Support consumer key/secret and token/token_secret credentials
3. Use HMAC-SHA1 signature method
4. Include OAuth parameters in the Authorization header
5. Handle both GET and POST requests with OAuth signatures
6. Generate proper timestamps and nonces automatically

## OAuth Flow

For this task, focus on signing requests (not the full OAuth handshake):
- Assume you already have consumer credentials and access tokens
- Sign each request with the OAuth signature
- Include proper OAuth parameters in headers

## Example Usage

```javascript
const oauthClient = require('./oauth-client');

const credentials = {
  consumer_key: 'your-consumer-key',
  consumer_secret: 'your-consumer-secret',
  token: 'user-access-token',
  token_secret: 'user-token-secret'
};

// Make OAuth-signed GET request
oauthClient.get('https://api.example.com/account/verify', credentials, (err, response) => {
  console.log('Account:', response);
});

// Make OAuth-signed POST request
oauthClient.post('https://api.example.com/status/update', credentials, {
  status: 'Hello World'
}, (err, response) => {
  console.log('Posted:', response);
});
```

## OAuth Signature Requirements

- Signature method: HMAC-SHA1
- OAuth parameters should be in Authorization header (not query string)
- Automatic timestamp and nonce generation
- Proper percent-encoding of parameters

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with built-in OAuth 1.0 signing capabilities.

## Test Cases

### Test 1: OAuth GET Request @test

Input: GET request with OAuth credentials

Expected behavior: Should include properly signed Authorization header with OAuth parameters

### Test 2: OAuth POST Request @test

Input: POST request with OAuth credentials and body data

Expected behavior: Should sign the request including body parameters in signature base string

### Test 3: Automatic Parameters @test

Input: OAuth credentials without timestamp or nonce

Expected behavior: Should automatically generate oauth_timestamp and oauth_nonce
