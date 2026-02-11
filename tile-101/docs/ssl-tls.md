# SSL/TLS Configuration

Comprehensive SSL/TLS options for client certificates, custom certificate authorities, protocol selection, and certificate validation.

## Capabilities

### SSL Certificate Validation

Control SSL certificate validation.

```javascript { .api }
/**
 * SSL validation options
 */
interface SSLValidationOptions {
  /** Require valid SSL certificates (default: true) */
  strictSSL?: boolean;

  /** Reject unauthorized certificates */
  rejectUnauthorized?: boolean;
}
```

**Usage Examples:**

```javascript
// Strict SSL (default - verify certificates)
request('https://example.com', callback);

// Disable SSL verification (not recommended for production)
request({
  uri: 'https://self-signed.example.com',
  strictSSL: false
}, callback);

// Equivalent using rejectUnauthorized
request({
  uri: 'https://self-signed.example.com',
  rejectUnauthorized: false
}, callback);
```

**Notes:**
- `strictSSL: true` is the default (validates certificates)
- `strictSSL: false` sets `rejectUnauthorized: false`
- Only disable in development/testing environments
- Production should always validate certificates

### Custom Certificate Authority

Specify custom CA certificates for validation.

```javascript { .api }
/**
 * Custom CA option
 */
interface CAOption {
  /** Custom Certificate Authority (string, Buffer, or array) */
  ca?: string | Buffer | Array<string | Buffer>;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Single CA certificate
request({
  url: 'https://api.some-server.com/',
  ca: fs.readFileSync('ca.cert.pem')
}, callback);

// Multiple CA certificates
request({
  url: 'https://api.some-server.com/',
  ca: [
    fs.readFileSync('ca1.cert.pem'),
    fs.readFileSync('ca2.cert.pem')
  ]
}, callback);

// CA as string
const caCert = fs.readFileSync('ca.cert.pem', 'utf8');
request({
  url: 'https://api.some-server.com/',
  ca: caCert
}, callback);
```

**Use Cases:**
- Self-signed certificates
- Internal CA certificates
- Private PKI infrastructure
- Certificate pinning

### Client Certificates

Provide client-side certificates for mutual TLS authentication.

```javascript { .api }
/**
 * Client certificate options
 */
interface ClientCertOptions {
  /** Client certificate (PEM format) */
  cert?: string | Buffer;

  /** Client private key (PEM format) */
  key?: string | Buffer;

  /** Passphrase for encrypted private key */
  passphrase?: string;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');
const path = require('path');

// Client certificate with separate cert and key
const certFile = path.resolve(__dirname, 'ssl/client.crt');
const keyFile = path.resolve(__dirname, 'ssl/client.key');

request({
  url: 'https://api.some-server.com/',
  cert: fs.readFileSync(certFile),
  key: fs.readFileSync(keyFile)
}, callback);

// With passphrase-protected key
request({
  url: 'https://api.some-server.com/',
  cert: fs.readFileSync(certFile),
  key: fs.readFileSync(keyFile),
  passphrase: 'password'
}, callback);

// With custom CA
const caFile = path.resolve(__dirname, 'ssl/ca.cert.pem');

request({
  url: 'https://api.some-server.com/',
  cert: fs.readFileSync(certFile),
  key: fs.readFileSync(keyFile),
  passphrase: 'password',
  ca: fs.readFileSync(caFile)
}, callback);
```

### PFX/PKCS12 Certificates

Use PFX/PKCS12 format certificates (combines cert, key, and CA).

```javascript { .api }
/**
 * PFX certificate option
 */
interface PFXOption {
  /** PFX/PKCS12 certificate (replaces cert and key) */
  pfx?: string | Buffer;

  /** Passphrase for PFX */
  passphrase?: string;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// PFX certificate
request({
  url: 'https://api.some-server.com/',
  pfx: fs.readFileSync('certificate.pfx'),
  passphrase: 'password'
}, callback);

// PFX with custom CA
request({
  url: 'https://api.some-server.com/',
  pfx: fs.readFileSync('certificate.pfx'),
  passphrase: 'password',
  ca: fs.readFileSync('ca.cert.pem')
}, callback);
```

**Note:** PFX format combines certificate, private key, and optionally CA certificates in a single file.

### Using agentOptions

Alternative way to specify SSL/TLS options via agentOptions.

```javascript { .api }
/**
 * Agent options for SSL/TLS
 */
interface AgentSSLOptions {
  agentOptions?: {
    cert?: string | Buffer;
    key?: string | Buffer;
    pfx?: string | Buffer;
    passphrase?: string;
    ca?: string | Buffer | Array<string | Buffer>;
    ciphers?: string;
    secureProtocol?: string;
    secureOptions?: number;
  };
}
```

**Usage Examples:**

```javascript
const fs = require('fs');
const path = require('path');

const certFile = path.resolve(__dirname, 'ssl/client.crt');
const keyFile = path.resolve(__dirname, 'ssl/client.key');

// Client certificate via agentOptions
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    cert: fs.readFileSync(certFile),
    key: fs.readFileSync(keyFile),
    passphrase: 'password'
  }
}, callback);

// Disable SSLv3
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    cert: fs.readFileSync(certFile),
    key: fs.readFileSync(keyFile),
    passphrase: 'password',
    securityOptions: 'SSL_OP_NO_SSLv3'
  }
}, callback);

// PFX via agentOptions
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    pfx: fs.readFileSync('certificate.pfx'),
    passphrase: 'password'
  }
}, callback);

// Custom CA via agentOptions
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    ca: fs.readFileSync('ca.cert.pem')
  }
}, callback);
```

**Notes:**
- Recommended to use direct options (cert, key, ca) instead of agentOptions
- agentOptions required for some advanced scenarios
- agentOptions not applied same way in proxied environments
- Use direct options for better proxy compatibility

### Protocol and Cipher Configuration

Configure SSL/TLS protocol version and cipher suites.

```javascript { .api }
/**
 * Protocol and cipher options
 */
interface ProtocolCipherOptions {
  /** SSL/TLS protocol method (e.g., 'SSLv3_method', 'TLSv1_2_method') */
  secureProtocol?: string;

  /** SSL options flags */
  secureOptions?: number;

  /** Cipher suite specification */
  ciphers?: string;
}
```

**Usage Examples:**

```javascript
// Force SSLv3 only
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    secureProtocol: 'SSLv3_method'
  }
}, callback);

// Force TLS 1.2
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    secureProtocol: 'TLSv1_2_method'
  }
}, callback);

// Disable SSLv3
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    secureOptions: require('constants').SSL_OP_NO_SSLv3
  }
}, callback);

// Custom cipher suite
request({
  url: 'https://api.some-server.com/',
  ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384'
}, callback);

// Cipher via agentOptions
request({
  url: 'https://api.some-server.com/',
  agentOptions: {
    ciphers: 'HIGH:!aNULL:!MD5'
  }
}, callback);
```

**Common secureProtocol Values:**
- `'SSLv3_method'` - SSL 3.0 (deprecated, insecure)
- `'TLSv1_method'` - TLS 1.0
- `'TLSv1_1_method'` - TLS 1.1
- `'TLSv1_2_method'` - TLS 1.2
- `'TLS_method'` - Negotiate highest available

**Common secureOptions Flags:**
- `SSL_OP_NO_SSLv3` - Disable SSL 3.0
- `SSL_OP_NO_TLSv1` - Disable TLS 1.0
- `SSL_OP_NO_TLSv1_1` - Disable TLS 1.1

### Complete SSL/TLS Configuration Example

Comprehensive example with all SSL/TLS options.

**Usage Example:**

```javascript
const fs = require('fs');
const path = require('path');

const certFile = path.resolve(__dirname, 'ssl/client.crt');
const keyFile = path.resolve(__dirname, 'ssl/client.key');
const caFile = path.resolve(__dirname, 'ssl/ca.cert.pem');

request({
  url: 'https://api.some-server.com/',

  // Validation
  strictSSL: true,
  rejectUnauthorized: true,

  // Custom CA
  ca: fs.readFileSync(caFile),

  // Client certificate
  cert: fs.readFileSync(certFile),
  key: fs.readFileSync(keyFile),
  passphrase: 'password',

  // Protocol and ciphers
  ciphers: 'HIGH:!aNULL:!MD5',
  secureProtocol: 'TLSv1_2_method',
  secureOptions: require('constants').SSL_OP_NO_SSLv3

}, function(err, response, body) {
  if (err) {
    console.error('SSL Error:', err);
    return;
  }
  console.log('Secure request successful');
});
```

## Types

### SSL/TLS Option Types

```javascript { .api }
/**
 * SSL/TLS configuration options
 */
interface SSLOptions {
  /** Require valid SSL certificates (default: true) */
  strictSSL?: boolean;

  /** Reject unauthorized certificates */
  rejectUnauthorized?: boolean;

  /** Certificate Authority */
  ca?: string | Buffer | Array<string | Buffer>;

  /** Client certificate (PEM) */
  cert?: string | Buffer;

  /** Client private key (PEM) */
  key?: string | Buffer;

  /** PFX/PKCS12 certificate */
  pfx?: string | Buffer;

  /** Passphrase for private key or PFX */
  passphrase?: string;

  /** Cipher suite */
  ciphers?: string;

  /** SSL/TLS protocol */
  secureProtocol?: string;

  /** SSL options flags */
  secureOptions?: number;
}

/**
 * Agent options for SSL/TLS
 */
interface AgentSSLOptions {
  agentOptions?: {
    cert?: string | Buffer;
    key?: string | Buffer;
    pfx?: string | Buffer;
    passphrase?: string;
    ca?: string | Buffer | Array<string | Buffer>;
    ciphers?: string;
    secureProtocol?: string;
    secureOptions?: number;
    rejectUnauthorized?: boolean;
  };
}
```

## Notes

### General
- Direct options (cert, key, ca) are recommended over agentOptions
- agentOptions not applied same way in proxied environments
- SSL validation enabled by default (strictSSL: true)

### Certificates
- Certificates should be in PEM format (Base64 encoded)
- Client certificates used for mutual TLS authentication
- PFX format combines cert, key, and optionally CA in single file
- CA certificates required for self-signed or internal certificates

### Validation
- `strictSSL: false` disables all certificate validation (not recommended)
- `rejectUnauthorized: false` allows self-signed certificates
- Custom CA required for private/internal PKI
- Production environments should always validate certificates

### Protocol Selection
- `secureProtocol` forces specific SSL/TLS version
- `secureOptions` allows disabling specific versions
- Modern applications should use TLS 1.2 or higher
- SSLv3 and older are deprecated and insecure

### Ciphers
- `ciphers` option controls cipher suite selection
- Format is OpenSSL cipher list format
- Strong ciphers recommended: `'HIGH:!aNULL:!MD5'`
- Default ciphers usually sufficient for modern servers

### Agent Options
- Required for some advanced configurations
- Supports additional options not available as direct options
- Less reliable in proxy scenarios
- Use for: securityOptions, custom agent configurations

### File Paths
- Use absolute paths for certificate files
- `fs.readFileSync()` reads files synchronously
- Can pass Buffer or string to certificate options
- PEM certificates as string must include full PEM structure
