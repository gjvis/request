# Authentication Methods

Support for multiple authentication methods including Basic, Bearer, Digest, OAuth 1.0, AWS Signature, Hawk, and HTTP Signature.

## Capabilities

### Basic and Bearer Authentication

HTTP Basic and Bearer token authentication.

```javascript { .api }
/**
 * Basic/Bearer authentication options
 */
interface AuthOption {
  /** Username (or use 'username') */
  user?: string;
  /** Username (or use 'user') */
  username?: string;
  /** Password (or use 'password') */
  pass?: string;
  /** Password (or use 'pass') */
  password?: string;
  /** Send auth header immediately or wait for 401 (default: true) */
  sendImmediately?: boolean;
  /** Bearer token (string or function returning string) */
  bearer?: string | (() => string);
}
```

**Usage Examples:**

```javascript
// Basic authentication with options object
request.get('http://some.server.com/', {
  auth: {
    user: 'username',
    pass: 'password',
    sendImmediately: false
  }
}, callback);

// Basic auth with username/password properties
request.get('http://some.server.com/', {
  auth: {
    username: 'username',
    password: 'password'
  }
}, callback);

// Basic auth in URL
const username = 'username';
const password = 'password';
const url = 'http://' + username + ':' + password + '@some.server.com';
request({url: url}, callback);

// Bearer token
request.get('http://some.server.com/', {
  auth: {
    bearer: 'bearerToken'
  }
}, callback);

// Bearer token as function
let token = 'initialToken';
const getToken = () => token;

request.get('http://some.server.com/', {
  auth: {
    bearer: getToken
  }
}, callback);

// Using .auth() method
request.get('http://some.server.com/')
  .auth('username', 'password', false)
  .on('response', function(response) {
    console.log('Authenticated!');
  });

// Bearer with .auth() method
request.get('http://some.server.com/')
  .auth(null, null, true, 'bearerToken');
```

**Notes:**
- `sendImmediately: true` (default) sends credentials immediately
- `sendImmediately: false` waits for 401 response before sending credentials
- Digest authentication requires `sendImmediately: false`
- Bearer token can be a string or function returning a string

### OAuth 1.0 Signing

OAuth 1.0 signature support with multiple signing algorithms.

```javascript { .api }
/**
 * OAuth 1.0 authentication options
 */
interface OAuthOption {
  /** OAuth consumer key */
  consumer_key: string;
  /** OAuth consumer secret */
  consumer_secret: string;
  /** OAuth token (for authenticated requests) */
  token?: string;
  /** OAuth token secret */
  token_secret?: string;
  /** OAuth verifier (for token exchange) */
  verifier?: string;
  /** Signature method: 'HMAC-SHA1', 'RSA-SHA1', 'PLAINTEXT' (default: 'HMAC-SHA1') */
  signature_method?: string;
  /** Transport method: 'header', 'query', 'body' (default: 'header') */
  transport_method?: string;
  /** OAuth version (default: '1.0') */
  version?: string;
  /** Enable body hash or provide custom hash */
  body_hash?: boolean | string;
  /** Private key in PEM format (for RSA-SHA1) */
  private_key?: string;
}
```

**Usage Examples:**

```javascript
const qs = require('querystring');

// OAuth 1.0 three-legged flow (Twitter example)

// Step 1: Get request token
const oauth = {
  callback: 'http://mysite.com/callback/',
  consumer_key: CONSUMER_KEY,
  consumer_secret: CONSUMER_SECRET
};

request.post({
  url: 'https://api.twitter.com/oauth/request_token',
  oauth: oauth
}, function (e, r, body) {
  // Step 2: User authorization
  const req_data = qs.parse(body);
  const uri = 'https://api.twitter.com/oauth/authenticate' +
    '?' + qs.stringify({oauth_token: req_data.oauth_token});
  // Redirect user to uri...

  // Step 3: Exchange for access token (after user authorizes)
  const auth_data = qs.parse(body);  // from callback
  const oauth = {
    consumer_key: CONSUMER_KEY,
    consumer_secret: CONSUMER_SECRET,
    token: auth_data.oauth_token,
    token_secret: req_data.oauth_token_secret,
    verifier: auth_data.oauth_verifier
  };

  request.post({
    url: 'https://api.twitter.com/oauth/access_token',
    oauth: oauth
  }, function (e, r, body) {
    // Make authenticated requests
    const perm_data = qs.parse(body);
    const oauth = {
      consumer_key: CONSUMER_KEY,
      consumer_secret: CONSUMER_SECRET,
      token: perm_data.oauth_token,
      token_secret: perm_data.oauth_token_secret
    };

    request.get({
      url: 'https://api.twitter.com/1.1/users/show.json',
      oauth: oauth,
      qs: {
        screen_name: perm_data.screen_name,
        user_id: perm_data.user_id
      },
      json: true
    }, function (e, r, user) {
      console.log(user);
    });
  });
});

// RSA-SHA1 signing
request.get({
  url: 'https://api.example.com/resource',
  oauth: {
    consumer_key: CONSUMER_KEY,
    private_key: fs.readFileSync('private-key.pem', 'utf8'),
    signature_method: 'RSA-SHA1'
  }
}, callback);

// PLAINTEXT signing
request.get({
  url: 'https://api.example.com/resource',
  oauth: {
    consumer_key: CONSUMER_KEY,
    consumer_secret: CONSUMER_SECRET,
    signature_method: 'PLAINTEXT'
  }
}, callback);

// OAuth parameters in query string
request.get({
  url: 'https://api.example.com/resource',
  oauth: {
    consumer_key: CONSUMER_KEY,
    consumer_secret: CONSUMER_SECRET,
    transport_method: 'query'
  }
}, callback);

// Request body hash
request.post({
  url: 'https://api.example.com/resource',
  oauth: {
    consumer_key: CONSUMER_KEY,
    consumer_secret: CONSUMER_SECRET,
    body_hash: true  // Auto-generate body hash
  },
  body: 'request body content'
}, callback);

// Using .oauth() method
request.post('https://api.example.com/resource')
  .oauth({
    consumer_key: CONSUMER_KEY,
    consumer_secret: CONSUMER_SECRET
  })
  .on('response', function(response) {
    console.log('OAuth request sent');
  });
```

### AWS Signature Authentication

AWS signature authentication for S3 and other AWS services.

```javascript { .api }
/**
 * AWS signature authentication options
 */
interface AWSOption {
  /** AWS access key ID */
  key: string;
  /** AWS secret access key */
  secret: string;
  /** S3 bucket name (optional) */
  bucket?: string;
  /** Signature version: 2 or 4 (default: 2) */
  sign_version?: number;
}
```

**Usage Examples:**

```javascript
// AWS Signature Version 2 (default)
request.get({
  url: 'https://s3.amazonaws.com/bucket/file',
  aws: {
    key: AWS_ACCESS_KEY,
    secret: AWS_SECRET_KEY,
    bucket: 'my-bucket'
  }
}, callback);

// AWS Signature Version 4
// Note: Requires installing aws4 package separately
request.get({
  url: 'https://s3.amazonaws.com/bucket/file',
  aws: {
    key: AWS_ACCESS_KEY,
    secret: AWS_SECRET_KEY,
    sign_version: 4
  }
}, callback);

// Using .aws() method
request.get('https://s3.amazonaws.com/bucket/file')
  .aws({
    key: AWS_ACCESS_KEY,
    secret: AWS_SECRET_KEY
  }, true)  // true = sign immediately
  .on('response', function(response) {
    console.log('AWS request sent');
  });
```

**Note:** AWS Signature Version 4 requires the `aws4` package to be installed separately.

### Hawk Authentication

Hawk HTTP authentication scheme.

```javascript { .api }
/**
 * Hawk authentication options
 */
interface HawkOption {
  /** Hawk credentials object */
  credentials: {
    id: string;
    key: string;
    algorithm: string;
  };
  /** Additional Hawk options */
  [key: string]: any;
}
```

**Usage Example:**

```javascript
// Hawk authentication
request.get({
  url: 'https://api.example.com/resource',
  hawk: {
    credentials: {
      id: 'dh37fgj492je',
      key: 'werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn',
      algorithm: 'sha256'
    }
  }
}, callback);

// Using .hawk() method
request.get('https://api.example.com/resource')
  .hawk({
    credentials: {
      id: 'dh37fgj492je',
      key: 'werxhqb98rpaxn39848xrunpaw3489ruxnpa98w4rxn',
      algorithm: 'sha256'
    }
  });
```

### HTTP Signature Authentication

HTTP Signature authentication scheme.

```javascript { .api }
/**
 * HTTP Signature authentication options
 */
interface HTTPSignatureOption {
  /** Key identifier */
  keyId: string;
  /** Private key in PEM format */
  key: string;
  /** Headers to sign (optional) */
  headers?: Array<string>;
  /** Signature algorithm (optional) */
  algorithm?: string;
}
```

**Usage Example:**

```javascript
const fs = require('fs');

// HTTP Signature authentication
request.get({
  url: 'https://api.example.com/resource',
  httpSignature: {
    keyId: 'my-key-id',
    key: fs.readFileSync('private-key.pem', 'utf8')
  }
}, callback);

// With custom headers and algorithm
request.get({
  url: 'https://api.example.com/resource',
  httpSignature: {
    keyId: 'my-key-id',
    key: fs.readFileSync('private-key.pem', 'utf8'),
    headers: ['(request-target)', 'host', 'date'],
    algorithm: 'rsa-sha256'
  }
}, callback);

// Using .httpSignature() method
request.get('https://api.example.com/resource')
  .httpSignature({
    keyId: 'my-key-id',
    key: fs.readFileSync('private-key.pem', 'utf8')
  });
```

### Authentication Methods on Request Instance

Chainable authentication methods on Request instances.

```javascript { .api }
/**
 * Set Basic or Bearer authentication
 * @param user - Username (null for bearer only)
 * @param pass - Password (null for bearer only)
 * @param sendImmediately - Send immediately or wait for 401 (default: true)
 * @param bearer - Bearer token
 * @returns this
 */
interface Request {
  auth(user: string | null, pass: string | null, sendImmediately?: boolean, bearer?: string): Request;
}

/**
 * Set OAuth 1.0 authentication
 * @param oauth - OAuth configuration
 * @returns this
 */
interface Request {
  oauth(oauth: OAuthOption): Request;
}

/**
 * Set Hawk authentication
 * @param opts - Hawk options
 * @returns this
 */
interface Request {
  hawk(opts: HawkOption): Request;
}

/**
 * Set AWS signature authentication
 * @param opts - AWS options
 * @param now - Apply signature immediately (default: false)
 * @returns this
 */
interface Request {
  aws(opts: AWSOption, now?: boolean): Request;
}

/**
 * Set HTTP Signature authentication
 * @param opts - HTTP Signature options
 * @returns this
 */
interface Request {
  httpSignature(opts: HTTPSignatureOption): Request;
}
```

**Usage Examples:**

```javascript
// Chainable auth methods
request.get('http://some.server.com/')
  .auth('username', 'password', false)
  .on('response', function(response) {
    console.log('Response received');
  });

// Chaining with other methods
request.post('https://api.example.com/resource')
  .auth('username', 'password')
  .json({data: 'value'})
  .on('error', function(err) {
    console.error(err);
  });
```

## Types

### Authentication Option Types

```javascript { .api }
/**
 * Basic/Bearer authentication
 */
interface AuthOption {
  user?: string;
  username?: string;
  pass?: string;
  password?: string;
  sendImmediately?: boolean;
  bearer?: string | (() => string);
}

/**
 * OAuth 1.0 authentication
 */
interface OAuthOption {
  consumer_key: string;
  consumer_secret: string;
  token?: string;
  token_secret?: string;
  verifier?: string;
  signature_method?: 'HMAC-SHA1' | 'RSA-SHA1' | 'PLAINTEXT';
  transport_method?: 'header' | 'query' | 'body';
  version?: string;
  body_hash?: boolean | string;
  private_key?: string;
}

/**
 * AWS signature authentication
 */
interface AWSOption {
  key: string;
  secret: string;
  bucket?: string;
  sign_version?: 2 | 4;
}

/**
 * Hawk authentication
 */
interface HawkOption {
  credentials: {
    id: string;
    key: string;
    algorithm: string;
  };
  [key: string]: any;
}

/**
 * HTTP Signature authentication
 */
interface HTTPSignatureOption {
  keyId: string;
  key: string;
  headers?: Array<string>;
  algorithm?: string;
}
```
