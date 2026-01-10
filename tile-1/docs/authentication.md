# Authentication

Request provides comprehensive authentication support including HTTP Basic/Digest/Bearer, OAuth 1.0, AWS signatures, Hawk, and HTTP Signature.

## Capabilities

### HTTP Authentication

Set HTTP authentication credentials (Basic, Digest, or Bearer).

```javascript { .api }
/**
 * Sets HTTP authentication
 * @param {string|object} user - Username or options object
 * @param {string} [pass] - Password
 * @param {boolean} [sendImmediately] - Send credentials without waiting for 401 (default: true for Basic, false for Digest)
 * @param {string} [bearer] - Bearer token string
 */
Request.prototype.auth(user, pass, sendImmediately, bearer);
```

**Usage with auth Option:**

```javascript
const request = require('request');

// Basic authentication
request.get({
  uri: 'http://api.example.com/protected',
  auth: {
    user: 'username',
    pass: 'password',
    sendImmediately: true
  }
}, function(err, response, body) {
  console.log('Response:', body);
});

// Alternative property names
request.get({
  uri: 'http://api.example.com/protected',
  auth: {
    username: 'john',
    password: 'secret'
  }
}, function(err, response, body) {
  console.log(body);
});

// Bearer token authentication
request.get({
  uri: 'http://api.example.com/protected',
  auth: {
    bearer: 'your-token-here'
  }
}, function(err, response, body) {
  console.log(body);
});

// Digest authentication
request.get({
  uri: 'http://api.example.com/protected',
  auth: {
    user: 'username',
    pass: 'password',
    sendImmediately: false // Wait for 401 challenge for Digest
  }
}, function(err, response, body) {
  console.log(body);
});
```

**Usage with auth Method:**

```javascript
// Basic auth with method
request.get('http://api.example.com/protected')
  .auth('username', 'password', true)
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
  });

// Bearer token with method
request.get('http://api.example.com/protected')
  .auth(null, null, true, 'bearer-token-here')
  .on('response', function(response) {
    console.log('Authenticated');
  });
```

**In URL (Basic Auth Only):**

```javascript
// Basic auth in URL (not recommended for security)
request.get('http://username:password@api.example.com/protected',
  function(err, response, body) {
    console.log(body);
  }
);
```

### OAuth 1.0 Signing

Sign requests using OAuth 1.0 protocol.

```javascript { .api }
/**
 * Signs request with OAuth 1.0
 * @param {object} _oauth - OAuth configuration object
 */
Request.prototype.oauth(_oauth);
```

**OAuth Options:**

```javascript { .api }
interface OAuthOptions {
  /** OAuth consumer key */
  consumer_key: string;

  /** OAuth consumer secret */
  consumer_secret: string;

  /** OAuth token (optional for request token flow) */
  token?: string;

  /** OAuth token secret (optional for request token flow) */
  token_secret?: string;

  /** OAuth verifier (for access token exchange) */
  verifier?: string;

  /** OAuth callback URL */
  callback?: string;

  /** Request URL (usually auto-detected) */
  url?: string;

  /** HTTP method (usually auto-detected) */
  method?: string;

  /** Additional parameters to sign */
  data?: object;

  /** Transport method: 'header' (default), 'query', or 'body' */
  transport_method?: string;

  /** Signature method: 'HMAC-SHA1' (default), 'HMAC-SHA256', 'PLAINTEXT', 'RSA-SHA1' */
  signature_method?: string;

  /** Body hash extension */
  body_hash?: boolean;
}
```

**Usage Examples:**

```javascript
const request = require('request');

// OAuth 1.0 request
request.get({
  uri: 'http://api.example.com/protected',
  oauth: {
    consumer_key: 'your-consumer-key',
    consumer_secret: 'your-consumer-secret',
    token: 'user-access-token',
    token_secret: 'user-token-secret'
  }
}, function(err, response, body) {
  console.log('OAuth response:', body);
});

// OAuth request token (3-legged OAuth)
request.post({
  uri: 'http://api.example.com/oauth/request_token',
  oauth: {
    callback: 'http://mysite.com/callback',
    consumer_key: 'key',
    consumer_secret: 'secret'
  }
}, function(err, response, body) {
  console.log('Request token:', body);
});

// OAuth access token exchange
request.post({
  uri: 'http://api.example.com/oauth/access_token',
  oauth: {
    consumer_key: 'key',
    consumer_secret: 'secret',
    token: 'request-token',
    token_secret: 'request-token-secret',
    verifier: 'oauth-verifier'
  }
}, function(err, response, body) {
  console.log('Access token:', body);
});

// OAuth with additional signed parameters
request.get({
  uri: 'http://api.example.com/data',
  oauth: {
    consumer_key: 'key',
    consumer_secret: 'secret',
    token: 'token',
    token_secret: 'token-secret',
    data: {
      user_id: '123',
      scope: 'read'
    }
  }
}, function(err, response, body) {
  console.log(body);
});
```

### AWS Signature Signing

Sign requests using AWS Signature Version 2 or Version 4.

```javascript { .api }
/**
 * Signs request with AWS signature (v2 or v4)
 * @param {object} opts - AWS signing options
 * @param {Date} [now] - Optional timestamp
 */
Request.prototype.aws(opts, now);
```

**AWS Options:**

```javascript { .api }
interface AWSOptions {
  /** AWS access key ID */
  key: string;

  /** AWS secret access key */
  secret: string;

  /** AWS session token (for temporary credentials) */
  session?: string;

  /** AWS service name (for v4 signature) */
  service?: string;

  /** AWS region (for v4 signature) */
  region?: string;

  /** S3 bucket name */
  bucket?: string;

  /** Use signature version 4 (default: auto-detect) */
  sign_version?: number;
}
```

**Usage Examples:**

```javascript
const request = require('request');

// AWS Signature v4 (recommended)
request.get({
  uri: 'http://s3.amazonaws.com/bucket/file.txt',
  aws: {
    key: 'AWS_ACCESS_KEY_ID',
    secret: 'AWS_SECRET_ACCESS_KEY',
    service: 's3',
    region: 'us-east-1'
  }
}, function(err, response, body) {
  console.log('S3 response:', body);
});

// AWS Signature v2 (legacy)
request.get({
  uri: 'http://s3.amazonaws.com/bucket/file.txt',
  aws: {
    key: 'AWS_ACCESS_KEY_ID',
    secret: 'AWS_SECRET_ACCESS_KEY',
    bucket: 'bucket',
    sign_version: 2
  }
}, function(err, response, body) {
  console.log(body);
});

// With temporary credentials (STS)
request.get({
  uri: 'http://s3.amazonaws.com/bucket/file.txt',
  aws: {
    key: 'temporary-access-key',
    secret: 'temporary-secret-key',
    session: 'session-token',
    service: 's3',
    region: 'us-west-2'
  }
}, function(err, response, body) {
  console.log(body);
});

// Custom service (API Gateway, etc.)
request.post({
  uri: 'http://api-id.execute-api.us-east-1.amazonaws.com/prod/resource',
  aws: {
    key: 'AWS_ACCESS_KEY_ID',
    secret: 'AWS_SECRET_ACCESS_KEY',
    service: 'execute-api',
    region: 'us-east-1'
  },
  json: { data: 'value' }
}, function(err, response, body) {
  console.log(body);
});
```

### Hawk Authentication

Sign requests using Hawk authentication protocol.

```javascript { .api }
/**
 * Signs request with Hawk authentication
 * @param {object} opts - Hawk options
 */
Request.prototype.hawk(opts);
```

**Hawk Options:**

```javascript { .api }
interface HawkOptions {
  /** Hawk credentials object */
  credentials: {
    /** Key ID */
    id: string;

    /** Shared secret key */
    key: string;

    /** Algorithm: 'sha1' or 'sha256' */
    algorithm: string;
  };

  /** Application specific data */
  ext?: string;

  /** Timestamp (defaults to now) */
  timestamp?: number;

  /** Nonce (defaults to auto-generated) */
  nonce?: string;

  /** Local time offset in milliseconds */
  localtimeOffsetMsec?: number;

  /** Payload for payload validation */
  payload?: string;

  /** Content-Type for payload validation */
  contentType?: string;

  /** Application ID (Oz extension) */
  app?: string;

  /** Delegated-by (Oz extension) */
  dlg?: string;
}
```

**Usage Examples:**

```javascript
const request = require('request');

// Hawk authentication
request.get({
  uri: 'http://api.example.com/resource',
  hawk: {
    credentials: {
      id: 'client-id',
      key: 'shared-secret-key',
      algorithm: 'sha256'
    }
  }
}, function(err, response, body) {
  console.log('Hawk response:', body);
});

// Hawk with payload validation
request.post({
  uri: 'http://api.example.com/resource',
  hawk: {
    credentials: {
      id: 'client-id',
      key: 'shared-secret-key',
      algorithm: 'sha256'
    },
    ext: 'app-specific-data',
    payload: JSON.stringify({ data: 'value' }),
    contentType: 'application/json'
  },
  json: { data: 'value' }
}, function(err, response, body) {
  console.log(body);
});

// Hawk with time sync
request.get({
  uri: 'http://api.example.com/resource',
  hawk: {
    credentials: {
      id: 'client-id',
      key: 'shared-secret-key',
      algorithm: 'sha256'
    },
    localtimeOffsetMsec: 60000 // 1 minute offset
  }
}, function(err, response, body) {
  console.log(body);
});
```

### HTTP Signature Authentication

Sign requests using HTTP Signature (Joyent's HTTP Signature Scheme).

```javascript { .api }
/**
 * Signs request using HTTP Signature
 * @param {object} opts - HTTP signature options
 */
Request.prototype.httpSignature(opts);
```

**HTTP Signature Options:**

```javascript { .api }
interface HTTPSignatureOptions {
  /** Key ID */
  keyId: string;

  /** Private key (PEM format string or Buffer) */
  key: string | Buffer;

  /** Headers to sign (array of header names) */
  headers?: string[];

  /** Signature algorithm (default: 'rsa-sha256') */
  algorithm?: string;

  /** Passphrase for encrypted private key */
  passphrase?: string;
}
```

**Usage Examples:**

```javascript
const request = require('request');
const fs = require('fs');

// Read private key
const privateKey = fs.readFileSync('private-key.pem', 'utf8');

// HTTP Signature authentication
request.get({
  uri: 'http://api.example.com/resource',
  httpSignature: {
    keyId: 'my-key-id',
    key: privateKey,
    headers: ['(request-target)', 'host', 'date']
  }
}, function(err, response, body) {
  console.log('Signed response:', body);
});

// With specific algorithm
request.post({
  uri: 'http://api.example.com/resource',
  httpSignature: {
    keyId: 'my-key-id',
    key: privateKey,
    algorithm: 'rsa-sha256',
    headers: ['(request-target)', 'host', 'date', 'digest']
  },
  json: { data: 'value' }
}, function(err, response, body) {
  console.log(body);
});

// With encrypted private key
request.get({
  uri: 'http://api.example.com/resource',
  httpSignature: {
    keyId: 'my-key-id',
    key: fs.readFileSync('encrypted-key.pem'),
    passphrase: 'key-password',
    headers: ['date', 'host']
  }
}, function(err, response, body) {
  console.log(body);
});
```

## Authentication Patterns

### Bearer Token in Header (Manual)

```javascript
// Set Authorization header manually
request.get({
  uri: 'http://api.example.com/protected',
  headers: {
    'Authorization': 'Bearer your-token-here'
  }
}, function(err, response, body) {
  console.log(body);
});
```

### API Key Authentication

```javascript
// API key in header
request.get({
  uri: 'http://api.example.com/data',
  headers: {
    'X-API-Key': 'your-api-key'
  }
}, function(err, response, body) {
  console.log(body);
});

// API key in query string
request.get({
  uri: 'http://api.example.com/data',
  qs: {
    api_key: 'your-api-key'
  }
}, function(err, response, body) {
  console.log(body);
});
```

### Custom Authentication Header

```javascript
// Custom authentication scheme
request.get({
  uri: 'http://api.example.com/resource',
  headers: {
    'Authorization': 'Custom-Scheme token=xyz123,signature=abc456'
  }
}, function(err, response, body) {
  console.log(body);
});
```

### Multiple Authentication Methods

```javascript
// Combine methods (e.g., API key + OAuth)
request.get({
  uri: 'http://api.example.com/resource',
  headers: {
    'X-API-Key': 'api-key-value'
  },
  oauth: {
    consumer_key: 'oauth-key',
    consumer_secret: 'oauth-secret',
    token: 'user-token',
    token_secret: 'token-secret'
  }
}, function(err, response, body) {
  console.log(body);
});
```

## Authentication with Defaults

Create authenticated request instances with default credentials:

```javascript
// Create authenticated request instance
const authenticatedRequest = request.defaults({
  auth: {
    bearer: 'your-default-token'
  }
});

// All requests use the token
authenticatedRequest.get('http://api.example.com/users', function(err, res, body) {
  console.log(body);
});

authenticatedRequest.post({
  uri: 'http://api.example.com/users',
  json: { name: 'John' }
}, function(err, res, body) {
  console.log(body);
});
```

## Important Notes

1. **sendImmediately**:
   - `true` (default for Basic): Sends credentials preemptively
   - `false` (default for Digest): Waits for 401 challenge

2. **Security**: Never hardcode credentials in source code. Use environment variables or secure configuration:

```javascript
request.get({
  uri: 'http://api.example.com/protected',
  auth: {
    user: process.env.API_USERNAME,
    pass: process.env.API_PASSWORD
  }
}, callback);
```

3. **Bearer Token**: Can be set via `auth.bearer` or manually in Authorization header

4. **OAuth**: Request handles signature generation and parameter encoding automatically

5. **AWS**: Automatically detects and uses appropriate signature version (v2 or v4) based on service

6. **Hawk/HTTP Signature**: More secure than Basic auth as credentials are never sent over the network
