# Authentication

The request package supports multiple authentication methods including HTTP Basic and Digest authentication, Bearer tokens, OAuth 1.0, AWS signatures (v2 and v4), Hawk authentication, and HTTP Signature Scheme.

## Capabilities

### HTTP Basic and Digest Authentication

Standard HTTP authentication using username and password.

```javascript { .api }
/**
 * Configure HTTP authentication
 * @param user - Username
 * @param pass - Password
 * @param sendImmediately - Send credentials with initial request (true) or wait for 401 challenge (false)
 * @param bearer - Bearer token (alternative to user/pass)
 * @returns Request instance for chaining
 */
function Request.prototype.auth(
  user: string,
  pass: string,
  sendImmediately?: boolean,
  bearer?: string
): Request;
```

**Auth Options Object:**

```javascript { .api }
interface AuthOptions {
  user?: string;
  username?: string;
  pass?: string;
  password?: string;
  sendImmediately?: boolean;
  bearer?: string;
}
```

**Usage Examples:**

```javascript
// Basic auth via method
request
  .get('http://api.example.com/protected')
  .auth('username', 'password');

// Basic auth via options
request({
  uri: 'http://api.example.com/protected',
  auth: {
    user: 'username',
    pass: 'password',
    sendImmediately: true
  }
}, callback);

// Digest auth (wait for 401 challenge)
request({
  uri: 'http://api.example.com/digest-protected',
  auth: {
    user: 'username',
    pass: 'password',
    sendImmediately: false
  }
}, callback);

// Auth in URL
request('http://username:password@api.example.com/protected', callback);
```

### Bearer Token Authentication

Bearer token authentication for OAuth 2.0 and similar protocols.

```javascript { .api }
/**
 * Configure Bearer token authentication
 * bearer - Bearer token string
 */
bearer: string
```

**Usage Examples:**

```javascript
// Via auth method
request
  .get('http://api.example.com/protected')
  .auth(null, null, true, 'myBearerToken123');

// Via auth options
request({
  uri: 'http://api.example.com/protected',
  auth: {
    bearer: 'myBearerToken123'
  }
}, callback);

// Via Authorization header (alternative)
request({
  uri: 'http://api.example.com/protected',
  headers: {
    'Authorization': 'Bearer myBearerToken123'
  }
}, callback);
```

### OAuth 1.0 Signing

OAuth 1.0a request signing with HMAC-SHA1, RSA-SHA1, or PLAINTEXT signature methods.

```javascript { .api }
/**
 * Configure OAuth 1.0 signing
 * @param params - OAuth parameters
 * @returns Request instance for chaining
 */
function Request.prototype.oauth(params: OAuthParams): Request;

interface OAuthParams {
  consumer_key: string;
  consumer_secret: string;
  token?: string;
  token_secret?: string;
  signature_method?: string;
  version?: string;
  timestamp?: number;
  nonce?: string;
  transport_method?: string;
  realm?: string;
  body_hash?: boolean;
}
```

**OAuth Parameters:**

- **consumer_key** (string, required): OAuth consumer key
- **consumer_secret** (string, required): OAuth consumer secret
- **token** (string): OAuth access token
- **token_secret** (string): OAuth access token secret
- **signature_method** (string): Signature method - 'HMAC-SHA1' (default), 'RSA-SHA1', or 'PLAINTEXT'
- **version** (string): OAuth version, defaults to '1.0'
- **timestamp** (number): Request timestamp (auto-generated if not provided)
- **nonce** (string): Unique random value (auto-generated if not provided)
- **transport_method** (string): Where to send OAuth params - 'header' (default), 'query', or 'body'
- **realm** (string): Authorization realm
- **body_hash** (boolean): Include body hash for POST/PUT requests

**Usage Examples:**

```javascript
// OAuth 1.0 with consumer credentials only
request({
  uri: 'http://api.twitter.com/oauth/request_token',
  oauth: {
    consumer_key: 'consumerKey',
    consumer_secret: 'consumerSecret'
  }
}, callback);

// OAuth 1.0 with access token
request({
  uri: 'http://api.twitter.com/1.1/statuses/update.json',
  method: 'POST',
  form: { status: 'Hello Twitter!' },
  oauth: {
    consumer_key: 'consumerKey',
    consumer_secret: 'consumerSecret',
    token: 'accessToken',
    token_secret: 'accessTokenSecret'
  }
}, callback);

// Using method syntax
request
  .get('http://api.example.com/protected')
  .oauth({
    consumer_key: 'key',
    consumer_secret: 'secret',
    token: 'token',
    token_secret: 'token_secret'
  });

// RSA-SHA1 signature
request({
  uri: 'http://api.example.com/resource',
  oauth: {
    consumer_key: 'key',
    consumer_secret: rsaPrivateKey,
    signature_method: 'RSA-SHA1'
  }
}, callback);

// Include body hash
request({
  uri: 'http://api.example.com/resource',
  method: 'POST',
  body: 'request body',
  oauth: {
    consumer_key: 'key',
    consumer_secret: 'secret',
    body_hash: true
  }
}, callback);
```

### AWS Signature Signing

Sign requests for Amazon Web Services using AWS Signature Version 2 or Version 4.

```javascript { .api }
/**
 * Configure AWS signature signing
 * @param options - AWS signing options
 * @param now - Current timestamp (optional, for testing)
 * @returns Request instance for chaining
 */
function Request.prototype.aws(options: AWSOptions, now?: boolean): Request;

interface AWSOptions {
  key: string;
  secret: string;
  bucket?: string;
  sign_version?: number;
  service?: string;
  region?: string;
}
```

**AWS Options:**

- **key** (string, required): AWS access key ID
- **secret** (string, required): AWS secret access key
- **bucket** (string): S3 bucket name
- **sign_version** (number): Signature version - 2 or 4 (default: 4)
- **service** (string): AWS service name (required for v4, e.g., 's3', 'ec2')
- **region** (string): AWS region (required for v4, e.g., 'us-east-1')

**Usage Examples:**

```javascript
// AWS Signature Version 4
request({
  uri: 'https://s3.amazonaws.com/bucket/file.txt',
  aws: {
    key: 'AWS_ACCESS_KEY',
    secret: 'AWS_SECRET_KEY',
    sign_version: 4,
    service: 's3',
    region: 'us-east-1'
  }
}, callback);

// AWS Signature Version 2
request({
  uri: 'https://s3.amazonaws.com/bucket/file.txt',
  aws: {
    key: 'AWS_ACCESS_KEY',
    secret: 'AWS_SECRET_KEY',
    bucket: 'bucket',
    sign_version: 2
  }
}, callback);

// Using method syntax
request
  .get('https://s3.amazonaws.com/mybucket/myfile.txt')
  .aws({
    key: process.env.AWS_ACCESS_KEY,
    secret: process.env.AWS_SECRET_KEY,
    service: 's3',
    region: 'us-west-2'
  });
```

### Hawk Authentication

Hawk HTTP authentication scheme providing cryptographic verification of requests.

```javascript { .api }
/**
 * Configure Hawk authentication
 * @param options - Hawk credentials and options
 * @returns Request instance for chaining
 */
function Request.prototype.hawk(options: HawkOptions): Request;

interface HawkOptions {
  credentials: {
    id: string;
    key: string;
    algorithm: string;
  };
  ext?: string;
  timestamp?: number;
  nonce?: string;
  app?: string;
  dlg?: string;
}
```

**Hawk Options:**

- **credentials** (object, required):
  - **id** (string): Credential identifier
  - **key** (string): Shared secret key
  - **algorithm** (string): HMAC algorithm - 'sha256' or 'sha1'
- **ext** (string): Application-specific data
- **timestamp** (number): Request timestamp
- **nonce** (string): Unique random value
- **app** (string): Application identifier
- **dlg** (string): Delegated-by identifier

**Usage Example:**

```javascript
request({
  uri: 'http://api.example.com/protected',
  method: 'GET',
  hawk: {
    credentials: {
      id: 'dh37fgj492je',
      key: 'werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn',
      algorithm: 'sha256'
    }
  }
}, callback);

// Using method syntax
request
  .get('http://api.example.com/protected')
  .hawk({
    credentials: {
      id: 'myId',
      key: 'myKey',
      algorithm: 'sha256'
    },
    ext: 'app-specific-data'
  });
```

### HTTP Signature Authentication

HTTP Signature Scheme for signing and verifying HTTP messages.

```javascript { .api }
/**
 * Configure HTTP Signature authentication
 * @param options - HTTP Signature options
 * @returns Request instance for chaining
 */
function Request.prototype.httpSignature(options: HttpSignatureOptions): Request;

interface HttpSignatureOptions {
  keyId: string;
  key: string;
  headers?: string[];
  algorithm?: string;
  authorizationHeaderName?: string;
}
```

**HTTP Signature Options:**

- **keyId** (string, required): Key identifier
- **key** (string, required): Private key (PEM format)
- **headers** (string[]): List of headers to include in signature (default: ['date'])
- **algorithm** (string): Signature algorithm - 'rsa-sha1', 'rsa-sha256', 'hmac-sha1', 'hmac-sha256', etc.
- **authorizationHeaderName** (string): Header name for authorization (default: 'authorization')

**Usage Examples:**

```javascript
const fs = require('fs');
const privateKey = fs.readFileSync('private-key.pem', 'utf8');

request({
  uri: 'http://api.example.com/protected',
  httpSignature: {
    keyId: 'my-key-id',
    key: privateKey,
    headers: ['(request-target)', 'date', 'host']
  }
}, callback);

// Using method syntax
request
  .get('http://api.example.com/protected')
  .httpSignature({
    keyId: 'my-key-id',
    key: privateKey,
    algorithm: 'rsa-sha256'
  });

// HMAC signature
request({
  uri: 'http://api.example.com/protected',
  httpSignature: {
    keyId: 'hmac-key-1',
    key: 'shared-secret',
    algorithm: 'hmac-sha256',
    headers: ['date', 'host', 'digest']
  }
}, callback);
```

## Combining Authentication with Other Options

All authentication methods can be combined with other request options:

```javascript
// OAuth with custom headers and timeout
request({
  uri: 'http://api.twitter.com/1.1/statuses/home_timeline.json',
  oauth: {
    consumer_key: 'key',
    consumer_secret: 'secret',
    token: 'token',
    token_secret: 'token_secret'
  },
  headers: {
    'User-Agent': 'my-twitter-client/1.0'
  },
  timeout: 5000,
  json: true
}, callback);

// AWS with gzip compression
request({
  uri: 'https://s3.amazonaws.com/bucket/large-file.json',
  aws: {
    key: process.env.AWS_KEY,
    secret: process.env.AWS_SECRET,
    service: 's3',
    region: 'us-east-1'
  },
  gzip: true,
  json: true
}, callback);
```

## Authentication Error Handling

Authentication failures typically result in 401 (Unauthorized) or 403 (Forbidden) status codes:

```javascript
request({
  uri: 'http://api.example.com/protected',
  auth: {
    user: 'username',
    pass: 'password'
  }
}, function(error, response, body) {
  if (!error && response.statusCode === 401) {
    console.error('Authentication failed');
  } else if (!error && response.statusCode === 403) {
    console.error('Access forbidden');
  } else if (!error && response.statusCode === 200) {
    console.log('Authentication successful');
  }
});
```
