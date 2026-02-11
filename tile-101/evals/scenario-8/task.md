# Build a Secure HTTPS Client with Custom Certificates

Create a Node.js client that makes HTTPS requests with custom SSL/TLS certificate configuration.

## Requirements

Your secure client should support:

1. Connecting to HTTPS endpoints with self-signed certificates
2. Providing custom Certificate Authority (CA) certificates
3. Using client certificates for mutual TLS authentication
4. Controlling SSL certificate validation behavior
5. Configuring secure protocol versions and cipher suites

## Use Cases

- Connect to internal APIs with self-signed certificates
- Implement mutual TLS (mTLS) with client certificates
- Accept specific CA certificates for private infrastructure
- Disable strict SSL validation for development/testing

## Example Usage

```javascript
const secureClient = require('./secure-client');

// Connect with self-signed cert (disable strict SSL)
secureClient.connectInsecure('https://self-signed.badssl.com/', (err, response) => {
  console.log('Connected to self-signed endpoint');
});

// Use custom CA certificate
secureClient.connectWithCA('https://internal-api.company.com', '/path/to/ca-cert.pem', (err, data) => {
  console.log('Connected with custom CA');
});

// Mutual TLS with client certificate
secureClient.connectMutualTLS('https://secure-api.example.com', {
  cert: '/path/to/client-cert.pem',
  key: '/path/to/client-key.pem',
  ca: '/path/to/ca-cert.pem'
}, (err, data) => {
  console.log('Connected with client certificate');
});
```

## Security Considerations

- Disabling SSL validation should only be used in development
- Production code should always validate certificates
- Client certificates should be kept secure
- Use appropriate cipher suites and protocol versions

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with comprehensive SSL/TLS certificate configuration options.

## Test Cases

### Test 1: Disable Strict SSL @test

Input: HTTPS URL with self-signed certificate

Expected behavior: Should connect successfully with SSL validation disabled

### Test 2: Custom CA Certificate @test

Input: HTTPS URL and path to CA certificate file

Expected behavior: Should trust the custom CA and validate certificates against it

### Test 3: Client Certificate Auth @test

Input: HTTPS URL, client certificate, and private key

Expected behavior: Should present client certificate for mutual TLS authentication
