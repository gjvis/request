# Streaming

Request provides a full Node.js Stream interface, allowing efficient data transfer without buffering entire requests or responses in memory.

## Capabilities

### Stream Interface

The Request class extends Node.js Stream and provides both readable and writable stream interfaces.

```javascript { .api }
/**
 * Request class extends Node.js Stream
 */
class Request extends Stream {
  /** Indicates the request is readable */
  readable: boolean;

  /** Indicates the request is writable */
  writable: boolean;
}
```

### Pipe Method

Pipes the response to a destination stream.

```javascript { .api }
/**
 * Pipes the response to a destination stream
 * @param {Stream} dest - Destination stream
 * @param {object} [opts] - Pipe options
 * @returns {Stream} Destination stream (for chaining)
 */
Request.prototype.pipe(dest, opts);
```

**Usage Examples:**

```javascript
const request = require('request');
const fs = require('fs');

// Pipe response to file
request('http://example.com/image.png')
  .pipe(fs.createWriteStream('image.png'));

// Pipe with error handling
request('http://example.com/doodle.png')
  .on('error', function(err) {
    console.error('Download failed:', err);
  })
  .pipe(fs.createWriteStream('doodle.png'));

// Pipe request to request
request.get('http://example.com/img.png')
  .pipe(request.put('http://mysite.com/img.png'));

// Chain pipes
request('http://example.com/data.json')
  .pipe(processStream)
  .pipe(fs.createWriteStream('output.json'));
```

### Piping FROM Streams

You can pipe streams into request to upload data.

**Usage Examples:**

```javascript
const fs = require('fs');

// Upload file via PUT
fs.createReadStream('file.json')
  .pipe(request.put('http://mysite.com/obj.json'));

// Upload file via POST with multipart
const formData = {
  file: fs.createReadStream('document.pdf'),
  name: 'My Document'
};
request.post({
  uri: 'http://service.com/upload',
  formData: formData
}, function(err, response, body) {
  console.log('Upload complete');
});

// Pipe from HTTP request to another
http.createServer(function (req, resp) {
  if (req.url === '/proxy') {
    req.pipe(request('http://backend.com/api')).pipe(resp);
  }
});
```

### Write Method

Writes data to the request body stream.

```javascript { .api }
/**
 * Writes data to the request body
 * @param {...*} args - Arguments passed to underlying stream.write()
 */
Request.prototype.write(...args);
```

**Usage Examples:**

```javascript
// Manually write request body
const req = request.post('http://api.example.com/data');

req.write('chunk1');
req.write('chunk2');
req.end('final chunk');

// Writing JSON data
const req = request.post({
  uri: 'http://api.example.com/stream',
  headers: {
    'Content-Type': 'application/json'
  }
});

req.write('{"items":[');
req.write('{"id":1},');
req.write('{"id":2}');
req.write(']}');
req.end();
```

### End Method

Ends the request, optionally writing a final chunk of data.

```javascript { .api }
/**
 * Ends the request, optionally writing final chunk
 * @param {string|Buffer} [chunk] - Final data to write
 */
Request.prototype.end(chunk);
```

**Usage Examples:**

```javascript
// End without data
const req = request.post('http://api.example.com/data');
req.write('data');
req.end();

// End with final chunk
const req = request.post('http://api.example.com/data');
req.end('final data');

// End after stream operations
const req = request.get('http://example.com/data');
req.on('data', function(chunk) {
  console.log('Received:', chunk);
});
req.on('end', function() {
  console.log('Stream ended');
});
```

### Pause Method

Pauses reading from the response stream.

```javascript { .api }
/**
 * Pauses reading from the response stream
 */
Request.prototype.pause();
```

**Usage Examples:**

```javascript
// Control flow with pause
const req = request('http://example.com/large-file');

req.on('data', function(chunk) {
  console.log('Received chunk:', chunk.length);

  // Pause to process data
  req.pause();

  processChunk(chunk, function() {
    // Resume when ready
    req.resume();
  });
});
```

### Resume Method

Resumes reading from the response stream after pause.

```javascript { .api }
/**
 * Resumes reading from the response stream
 */
Request.prototype.resume();
```

**Usage Examples:**

```javascript
// Resume after pause
const req = request('http://example.com/data');

req.pause(); // Pause immediately

setTimeout(function() {
  console.log('Resuming...');
  req.resume(); // Resume after delay
}, 1000);

req.on('data', function(chunk) {
  console.log('Data:', chunk);
});
```

### Abort Method

Aborts the ongoing request.

```javascript { .api }
/**
 * Aborts the ongoing request
 */
Request.prototype.abort();
```

**Usage Examples:**

```javascript
// Abort on timeout
const req = request('http://example.com/slow-endpoint');

setTimeout(function() {
  req.abort();
  console.log('Request aborted');
}, 5000);

req.on('error', function(err) {
  console.error('Error:', err.message);
});

// Abort based on condition
const req = request('http://example.com/data');
let totalBytes = 0;

req.on('data', function(chunk) {
  totalBytes += chunk.length;

  // Abort if response is too large
  if (totalBytes > 1024 * 1024) { // 1MB
    req.abort();
    console.log('Response too large, aborted');
  }
});
```

### Destroy Method

Destroys the request stream, cleaning up resources.

```javascript { .api }
/**
 * Destroys the request stream
 */
Request.prototype.destroy();
```

**Usage Examples:**

```javascript
// Clean up request
const req = request('http://example.com/data');

// Later...
req.destroy();

// Destroy on error
const req = request('http://example.com/data');

req.on('error', function(err) {
  console.error('Error occurred:', err);
  req.destroy();
});
```

## Stream Events

The Request instance emits standard Node.js stream events plus custom events:

### Standard Stream Events

```javascript { .api }
/**
 * Emitted when response is received
 * @event response
 * @param {http.IncomingMessage} response - HTTP response object
 */
request.on('response', function(response) {});

/**
 * Emitted when response data chunk is available
 * @event data
 * @param {Buffer|string} chunk - Data chunk
 */
request.on('data', function(chunk) {});

/**
 * Emitted when response ends
 * @event end
 */
request.on('end', function() {});

/**
 * Emitted on errors
 * @event error
 * @param {Error} error - Error object
 */
request.on('error', function(error) {});

/**
 * Emitted when connection closes
 * @event close
 */
request.on('close', function() {});

/**
 * Emitted when piped to another stream
 * @event pipe
 * @param {Stream} src - Source stream
 */
request.on('pipe', function(src) {});

/**
 * Emitted when write buffer is empty
 * @event drain
 */
request.on('drain', function() {});
```

### Custom Request Events

```javascript { .api }
/**
 * Emitted when request completes successfully (request-specific)
 * Only emitted when no callback is provided
 * @event complete
 * @param {http.IncomingMessage} response - HTTP response object
 * @param {string|Buffer|object} body - Response body
 */
request.on('complete', function(response, body) {});
```

**Usage Examples:**

```javascript
const request = require('request');

// Full event-driven example
const req = request('http://example.com/data');

req.on('response', function(response) {
  console.log('Status:', response.statusCode);
  console.log('Headers:', response.headers);
});

req.on('data', function(chunk) {
  console.log('Received:', chunk.length, 'bytes');
});

req.on('end', function() {
  console.log('Transfer complete');
});

req.on('error', function(err) {
  console.error('Error:', err.message);
});

// Complete event (no callback)
request('http://example.com/api')
  .on('complete', function(response, body) {
    console.log('Complete:', body);
  });
```

## Streaming Patterns

### File Download

```javascript
const request = require('request');
const fs = require('fs');

request('http://example.com/large-file.zip')
  .on('error', function(err) {
    console.error('Download failed:', err);
  })
  .on('response', function(response) {
    console.log('Status:', response.statusCode);
    console.log('Size:', response.headers['content-length']);
  })
  .pipe(fs.createWriteStream('large-file.zip'))
  .on('finish', function() {
    console.log('Download complete');
  });
```

### File Upload

```javascript
const request = require('request');
const fs = require('fs');

fs.createReadStream('upload.pdf')
  .pipe(request.put({
    uri: 'http://api.example.com/documents',
    headers: {
      'Content-Type': 'application/pdf'
    }
  }))
  .on('response', function(response) {
    console.log('Upload status:', response.statusCode);
  })
  .on('error', function(err) {
    console.error('Upload failed:', err);
  });
```

### Proxy Pattern

```javascript
const http = require('http');
const request = require('request');

http.createServer(function (req, resp) {
  // Simple proxy
  req.pipe(request('http://backend.com' + req.url)).pipe(resp);
}).listen(3000);
```

### Transform Stream

```javascript
const request = require('request');
const { Transform } = require('stream');

// Create transform stream
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

// Pipe through transform
request('http://example.com/data.txt')
  .pipe(upperCaseTransform)
  .pipe(process.stdout);
```

## Encoding

By default, response bodies are decoded as UTF-8 strings. Control encoding with the `encoding` option:

```javascript
// UTF-8 encoding (default)
request('http://example.com/text')
  .on('data', function(chunk) {
    console.log(typeof chunk); // 'string'
  });

// Binary data (no encoding)
request({ uri: 'http://example.com/image.png', encoding: null })
  .on('data', function(chunk) {
    console.log(Buffer.isBuffer(chunk)); // true
  });

// Specific encoding
request({ uri: 'http://example.com/data', encoding: 'latin1' })
  .on('data', function(chunk) {
    console.log(chunk); // latin1 encoded string
  });
```

## GZIP Compression

Enable automatic gzip decompression:

```javascript
request({ uri: 'http://example.com/data', gzip: true })
  .pipe(fs.createWriteStream('decompressed.txt'));

// Without gzip option, you'll get compressed data
request('http://example.com/data')
  .pipe(zlib.createGunzip())
  .pipe(fs.createWriteStream('decompressed.txt'));
```
