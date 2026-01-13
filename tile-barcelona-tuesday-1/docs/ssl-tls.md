# SSL/TLS Configuration

The request package provides comprehensive SSL/TLS configuration for secure HTTPS connections, including certificate management, validation control, cipher selection, and protocol configuration. Full support for custom certificates, client authentication, and corporate SSL environments.

## Capabilities

### SSL/TLS Options

Configure SSL/TLS settings for HTTPS requests.

```javascript { .api }
interface SSLOptions {
  /**
   * Certificate authority certificates (for validating server certificates)
   * Can be single cert or array of certs
   */
  ca?: string | Buffer | string[] | Buffer[];

  /**
   * Client certificate (for client authentication)
   */
  cert?: string | Buffer;

  /**
   * Client certificate private key
   */
  key?: string | Buffer;

  /**
   * PFX or PKCS12 encoded certificate and private key
   */
  pfx?: string | Buffer;

  /**
   * Passphrase for private key or pfx
   */
  passphrase?: string;

  /**
   * Validate SSL certificates (default: true)
   */
  strictSSL?: boolean;

  /**
   * Reject unauthorized or invalid SSL certificates (default: true)
   */
  rejectUnauthorized?: boolean;

  /**
   * SSL ciphers to use or exclude
   * Format: OpenSSL cipher list string
   */
  ciphers?: string;

  /**
   * SSL protocol to use
   * Examples: 'TLSv1_2_method', 'TLSv1_3_method'
   */
  secureProtocol?: string;

  /**
   * SSL options as numeric constants
   * Use Node.js crypto.constants values
   */
  secureOptions?: number;
}
```

**Usage Examples:**

```javascript
const request = require('request');
const fs = require('fs');

// Basic HTTPS request (validates certificates by default)
request('https://example.com', function(err, res, body) {
  if (err) {
    console.error('SSL Error:', err.message);
  }
});

// Disable certificate validation (NOT RECOMMENDED for production)
request({
  url: 'https://self-signed.example.com',
  strictSSL: false
});

// Use custom CA certificate
request({
  url: 'https://internal.company.com',
  ca: fs.readFileSync('./custom-ca.pem')
});

// Use multiple CA certificates
request({
  url: 'https://example.com',
  ca: [
    fs.readFileSync('./ca1.pem'),
    fs.readFileSync('./ca2.pem')
  ]
});
```

### Client Certificate Authentication

Provide client certificates for mutual TLS authentication.

```javascript { .api }
interface ClientCertOptions {
  /**
   * Client certificate in PEM format
   */
  cert: string | Buffer;

  /**
   * Private key for client certificate in PEM format
   */
  key: string | Buffer;

  /**
   * Passphrase for encrypted private key
   */
  passphrase?: string;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Client certificate with separate cert and key
request({
  url: 'https://secure-api.example.com',
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem')
});

// Client certificate with encrypted key
request({
  url: 'https://secure-api.example.com',
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./encrypted-key.pem'),
  passphrase: 'secret-password'
});

// Client certificate from PFX/PKCS12 file
request({
  url: 'https://secure-api.example.com',
  pfx: fs.readFileSync('./client-cert.pfx'),
  passphrase: 'secret-password'
});

// Read certificates as strings
const cert = fs.readFileSync('./client-cert.pem', 'utf8');
const key = fs.readFileSync('./client-key.pem', 'utf8');

request({
  url: 'https://secure-api.example.com',
  cert: cert,
  key: key
});
```

### Custom CA Certificates

Configure custom certificate authorities for validating server certificates.

```javascript { .api }
interface CAOptions {
  /**
   * Custom CA certificate(s) to trust
   * Replaces default system CA certificates
   */
  ca: string | Buffer | string[] | Buffer[];

  /**
   * Validate server certificates (default: true)
   */
  strictSSL?: boolean;
}
```

**Usage Examples:**

```javascript
// Trust custom CA for internal servers
const customCA = fs.readFileSync('./company-ca.pem');

request({
  url: 'https://internal.company.com',
  ca: customCA
});

// Trust multiple CAs (e.g., intermediate certificates)
request({
  url: 'https://example.com',
  ca: [
    fs.readFileSync('./root-ca.pem'),
    fs.readFileSync('./intermediate-ca.pem')
  ]
});

// Load CA from certificate chain file
const caChain = fs.readFileSync('./ca-chain.pem', 'utf8')
  .split('-----END CERTIFICATE-----\n')
  .filter(cert => cert.trim())
  .map(cert => cert + '-----END CERTIFICATE-----\n');

request({
  url: 'https://example.com',
  ca: caChain
});
```

### Cipher Configuration

Control SSL/TLS cipher suites for enhanced security.

```javascript { .api }
interface CipherOptions {
  /**
   * OpenSSL cipher list string
   * Format: Cipher names separated by colons
   * Use '!' prefix to exclude ciphers
   */
  ciphers?: string;

  /**
   * Minimum TLS version to accept
   */
  secureProtocol?: string;
}
```

**Usage Examples:**

```javascript
// Use only strong ciphers
request({
  url: 'https://example.com',
  ciphers: 'HIGH:!aNULL:!MD5'
});

// Specify exact cipher suite
request({
  url: 'https://example.com',
  ciphers: 'ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256'
});

// Exclude weak ciphers
request({
  url: 'https://example.com',
  ciphers: 'ALL:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK'
});

// Modern cipher configuration
request({
  url: 'https://example.com',
  ciphers: 'ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!SHA1'
});
```

### Protocol Configuration

Specify SSL/TLS protocol versions.

```javascript { .api }
interface ProtocolOptions {
  /**
   * SSL method to use
   * Values: 'TLSv1_method', 'TLSv1_1_method', 'TLSv1_2_method', 'TLSv1_3_method'
   */
  secureProtocol?: string;

  /**
   * OpenSSL options flags
   * Use constants from crypto module
   */
  secureOptions?: number;
}
```

**Usage Examples:**

```javascript
const crypto = require('crypto');

// Force TLS 1.2
request({
  url: 'https://example.com',
  secureProtocol: 'TLSv1_2_method'
});

// Force TLS 1.3 (Node.js 12+)
request({
  url: 'https://example.com',
  secureProtocol: 'TLSv1_3_method'
});

// Disable older TLS versions using secureOptions
request({
  url: 'https://example.com',
  secureOptions: crypto.constants.SSL_OP_NO_TLSv1 | crypto.constants.SSL_OP_NO_TLSv1_1
});

// Use modern TLS settings
request({
  url: 'https://example.com',
  secureProtocol: 'TLSv1_2_method',
  secureOptions: crypto.constants.SSL_OP_NO_SSLv2 |
                 crypto.constants.SSL_OP_NO_SSLv3 |
                 crypto.constants.SSL_OP_NO_TLSv1
});
```

### Certificate Validation Control

Control SSL certificate validation behavior.

```javascript { .api }
interface ValidationOptions {
  /**
   * Perform SSL certificate validation (default: true)
   * When false, accepts self-signed and expired certificates
   */
  strictSSL?: boolean;

  /**
   * Reject connections to servers with invalid certificates (default: true)
   * Alternative to strictSSL for finer control
   */
  rejectUnauthorized?: boolean;
}
```

**Usage Examples:**

```javascript
// Accept self-signed certificates (development only)
request({
  url: 'https://localhost:8443',
  strictSSL: false
});

// Alternative: use rejectUnauthorized
request({
  url: 'https://localhost:8443',
  rejectUnauthorized: false
});

// Validate certificates (default behavior)
request({
  url: 'https://example.com',
  strictSSL: true  // explicit, but true by default
});

// Handle SSL errors explicitly
request({
  url: 'https://example.com',
  strictSSL: true
}, function(err, res, body) {
  if (err && err.message.includes('SSL Error')) {
    console.error('Certificate validation failed:', err.message);
    // Handle SSL error
  }
});
```

## Usage Patterns

### Development Environment

Configure for local development with self-signed certificates:

```javascript
// Create development request instance
const devRequest = require('request').defaults({
  strictSSL: false,
  rejectUnauthorized: false
});

// Use for local HTTPS servers
devRequest('https://localhost:8443/api', function(err, res, body) {
  console.log('Development API response:', body);
});
```

### Corporate Environment

Configure for corporate proxy with custom CA:

```javascript
const fs = require('fs');

// Load corporate CA certificate
const corporateCA = fs.readFileSync('./corporate-ca.pem');

// Create corporate request instance
const corporateRequest = require('request').defaults({
  ca: corporateCA,
  proxy: 'http://proxy.company.com:8080',
  strictSSL: true
});

// Use for internal APIs
corporateRequest('https://api.company.com/data', function(err, res, body) {
  if (err) {
    console.error('Corporate API error:', err.message);
  }
});
```

### Mutual TLS Authentication

Set up mutual TLS for API authentication:

```javascript
const fs = require('fs');

const mtlsOptions = {
  cert: fs.readFileSync('./client-cert.pem'),
  key: fs.readFileSync('./client-key.pem'),
  ca: fs.readFileSync('./server-ca.pem'),
  passphrase: process.env.CERT_PASSPHRASE
};

// Create mTLS request instance
const mtlsRequest = require('request').defaults(mtlsOptions);

// Authenticate with client certificate
mtlsRequest('https://secure-api.example.com', function(err, res, body) {
  if (err) {
    console.error('mTLS authentication failed:', err.message);
  } else {
    console.log('Authenticated successfully');
  }
});
```

### High Security Configuration

Configure for maximum security:

```javascript
const crypto = require('crypto');

const secureRequest = require('request').defaults({
  strictSSL: true,
  ciphers: 'ECDHE+AESGCM:ECDHE+CHACHA20:!aNULL:!MD5:!DSS',
  secureProtocol: 'TLSv1_2_method',
  secureOptions: crypto.constants.SSL_OP_NO_SSLv2 |
                 crypto.constants.SSL_OP_NO_SSLv3 |
                 crypto.constants.SSL_OP_NO_TLSv1 |
                 crypto.constants.SSL_OP_NO_TLSv1_1
});

// Use for high-security APIs
secureRequest('https://bank-api.example.com', function(err, res, body) {
  if (err) {
    console.error('Secure connection failed:', err.message);
  }
});
```

### Loading Certificates from Files

Helper for loading certificate files:

```javascript
const fs = require('fs');
const path = require('path');

function loadCerts(certDir) {
  return {
    ca: fs.readFileSync(path.join(certDir, 'ca.pem')),
    cert: fs.readFileSync(path.join(certDir, 'client-cert.pem')),
    key: fs.readFileSync(path.join(certDir, 'client-key.pem')),
    passphrase: process.env.KEY_PASSPHRASE
  };
}

// Use loaded certificates
request({
  url: 'https://secure-api.example.com',
  ...loadCerts('./certs')
});
```

### Environment-Based Configuration

Configure SSL based on environment:

```javascript
const fs = require('fs');

function getSSLOptions() {
  if (process.env.NODE_ENV === 'production') {
    return {
      strictSSL: true,
      ca: fs.readFileSync('./prod-ca.pem'),
      cert: fs.readFileSync('./prod-cert.pem'),
      key: fs.readFileSync('./prod-key.pem')
    };
  } else {
    return {
      strictSSL: false,
      rejectUnauthorized: false
    };
  }
}

// Create environment-appropriate request instance
const apiRequest = require('request').defaults(getSSLOptions());
```

### Handling SSL Errors

Graceful SSL error handling:

```javascript
function makeSecureRequest(url, callback) {
  request({
    url: url,
    strictSSL: true
  }, function(err, res, body) {
    if (err) {
      if (err.message.includes('SSL Error') ||
          err.message.includes('certificate') ||
          err.code === 'CERT_HAS_EXPIRED' ||
          err.code === 'UNABLE_TO_VERIFY_LEAF_SIGNATURE') {

        console.error('SSL/TLS Error:', err.message);
        callback(new Error('Certificate validation failed'), null);
        return;
      }
    }
    callback(err, res, body);
  });
}

// Use with proper error handling
makeSecureRequest('https://example.com', function(err, res, body) {
  if (err) {
    console.error('Request failed:', err.message);
  } else {
    console.log('Request succeeded');
  }
});
```

## Types

### Certificate Formats

Certificates can be provided in multiple formats:

```javascript { .api }
/**
 * Certificate data can be:
 * - PEM format string (with -----BEGIN CERTIFICATE----- markers)
 * - Buffer containing PEM or DER data
 * - Array of PEM strings or Buffers (for certificate chains)
 */
type CertificateData = string | Buffer | string[] | Buffer[];

/**
 * Private key data can be:
 * - PEM format string (with -----BEGIN PRIVATE KEY----- or -----BEGIN RSA PRIVATE KEY----- markers)
 * - Buffer containing PEM or DER data
 */
type PrivateKeyData = string | Buffer;

/**
 * PFX data (PKCS#12) can be:
 * - Buffer containing PFX/PKCS12 data
 * - String path to PFX file (when used with fs.readFileSync)
 */
type PFXData = Buffer | string;
```

## Notes

### Security Best Practices

- **Always validate certificates in production** (`strictSSL: true`)
- **Never disable validation** for public APIs or production environments
- **Use strong ciphers** and disable weak algorithms
- **Keep certificates updated** and monitor expiration dates
- **Protect private keys** with proper file permissions and passphrases
- **Use environment variables** for sensitive data like passphrases
- **Prefer TLS 1.2 or higher** for modern security

### Certificate Validation

- By default, Node.js uses system CA certificates for validation
- Custom `ca` option replaces (not supplements) default CAs
- To trust both custom and system CAs, combine them in the ca array
- Certificate validation includes hostname verification
- Self-signed certificates require either custom CA or strictSSL: false

### Client Certificates

- Used for mutual TLS (mTLS) authentication
- Server must be configured to request and validate client certificates
- Private keys should be stored securely with proper permissions
- PFX format combines certificate and key in single file
- Passphrase protection recommended for private keys

### Protocol Support

- TLS 1.2 is widely supported and recommended minimum
- TLS 1.3 available in Node.js 12+ for improved security and performance
- Older protocols (SSLv2, SSLv3, TLS 1.0, TLS 1.1) should be disabled
- Protocol negotiation occurs automatically if not specified

### Common Errors

- **CERT_HAS_EXPIRED**: Server certificate has expired
- **UNABLE_TO_VERIFY_LEAF_SIGNATURE**: Cannot verify certificate chain
- **SELF_SIGNED_CERT_IN_CHAIN**: Self-signed certificate in chain
- **CERT_UNTRUSTED**: Certificate not trusted (CA not recognized)
- **DEPTH_ZERO_SELF_SIGNED_CERT**: Self-signed certificate without CA
