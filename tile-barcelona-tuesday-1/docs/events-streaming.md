# Events and Streaming

The request package implements the Node.js Stream interface, making Request instances both readable and writable streams. This enables efficient streaming of large request and response bodies, combined with a rich event system for fine-grained control over the request lifecycle.

## Capabilities

### Request Events

Request instances emit events throughout the request lifecycle.

```javascript { .api }
/**
 * Emitted when the HTTP request is created
 * @event request
 * @param req - Node.js http.ClientRequest object
 */
request.on('request', (req: http.ClientRequest) => void);

/**
 * Emitted when HTTP response is received
 * @event response
 * @param response - Node.js http.IncomingMessage object
 */
request.on('response', (response: http.IncomingMessage) => void);

/**
 * Emitted when data chunk is received
 * @event data
 * @param chunk - Buffer containing data
 */
request.on('data', (chunk: Buffer) => void);

/**
 * Emitted when response stream ends
 * @event end
 */
request.on('end', () => void);

/**
 * Emitted when request completes successfully
 * @event complete
 * @param response - HTTP response object
 * @param body - Complete response body (if buffered)
 */
request.on('complete', (response: http.IncomingMessage, body?: string | Buffer) => void);

/**
 * Emitted when an error occurs
 * @event error
 * @param error - Error object
 */
request.on('error', (error: Error) => void);

/**
 * Emitted when request is aborted
 * @event abort
 */
request.on('abort', () => void);

/**
 * Emitted when a stream is piped to the request
 * @event pipe
 * @param src - Source stream being piped
 */
request.on('pipe', (src: Stream) => void);

/**
 * Emitted when request socket is drained
 * @event drain
 */
request.on('drain', () => void);

/**
 * Emitted when socket is assigned to request
 * @event socket
 * @param socket - Network socket
 */
request.on('socket', (socket: net.Socket) => void);

/**
 * Emitted when redirect occurs
 * @event redirect
 */
request.on('redirect', () => void);

/**
 * Emitted when connection closes
 * @event close
 */
request.on('close', () => void);
```

**Usage Examples:**

```javascript
const request = require('request');

// Monitor request lifecycle
const req = request('http://example.com');

req.on('request', function(clientRequest) {
  console.log('Request created:', clientRequest.method, clientRequest.path);
});

req.on('response', function(response) {
  console.log('Response received:', response.statusCode);
  console.log('Headers:', response.headers);
});

req.on('data', function(chunk) {
  console.log('Data chunk received:', chunk.length, 'bytes');
});

req.on('end', function() {
  console.log('Response stream ended');
});

req.on('complete', function(response, body) {
  console.log('Request complete');
});

req.on('error', function(err) {
  console.error('Request error:', err.message);
});

// Track redirects
req.on('redirect', function() {
  console.log('Redirecting to:', this.uri.href);
});

// Monitor socket events
req.on('socket', function(socket) {
  console.log('Socket assigned:', socket.remoteAddress);
});
```

### Streaming Request Body

Write data to request as a writable stream.

```javascript { .api }
/**
 * Write data to request stream
 * @param chunk - Data to write
 * @param encoding - Character encoding (if chunk is string)
 * @param callback - Called when chunk is processed
 * @returns true if internal buffer is empty, false if backpressure
 */
function Request.prototype.write(
  chunk: string | Buffer,
  encoding?: string,
  callback?: () => void
): boolean;

/**
 * End the request stream
 * @param chunk - Optional final data to write
 * @param encoding - Character encoding (if chunk is string)
 * @param callback - Called when request ends
 */
function Request.prototype.end(
  chunk?: string | Buffer,
  encoding?: string,
  callback?: () => void
): void;
```

**Usage Examples:**

```javascript
// Stream data to POST request
const req = request.post('http://example.com/upload');

req.write('First chunk of data\n');
req.write('Second chunk of data\n');
req.end('Final chunk');

// Handle response
req.on('response', function(res) {
  console.log('Upload complete:', res.statusCode);
});

// Pipe readable stream to request
const fs = require('fs');
const fileStream = fs.createReadStream('large-file.json');

fileStream.pipe(request.post('http://example.com/upload'));

// Manual chunked upload with backpressure handling
const uploadData = getLargeDataSource();  // Some data source
const req = request.post('http://example.com/upload');

function writeChunk() {
  let canWrite = true;
  while (canWrite && uploadData.hasMore()) {
    const chunk = uploadData.getNextChunk();
    canWrite = req.write(chunk);
  }

  if (!canWrite) {
    // Wait for drain event before writing more
    req.once('drain', writeChunk);
  } else if (uploadData.hasMore()) {
    process.nextTick(writeChunk);
  } else {
    req.end();
  }
}

writeChunk();
```

### Streaming Response Body

Read response data as a readable stream.

```javascript { .api }
/**
 * Pipe response to destination stream
 * @param dest - Destination writable stream
 * @param opts - Pipe options
 * @returns Destination stream
 */
function Request.prototype.pipe(
  dest: WritableStream,
  opts?: { end?: boolean }
): WritableStream;

/**
 * Pause response stream
 */
function Request.prototype.pause(): void;

/**
 * Resume paused response stream
 */
function Request.prototype.resume(): void;

/**
 * Destroy the stream
 */
function Request.prototype.destroy(): void;
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Pipe response to file
request('http://example.com/file.pdf')
  .pipe(fs.createWriteStream('output.pdf'));

// Pipe with completion handling
request('http://example.com/data.json')
  .pipe(fs.createWriteStream('data.json'))
  .on('finish', function() {
    console.log('File downloaded successfully');
  });

// Pipe through transform stream
const zlib = require('zlib');

request('http://example.com/file.txt')
  .pipe(zlib.createGzip())
  .pipe(fs.createWriteStream('file.txt.gz'));

// Pause and resume streaming
const req = request('http://example.com/large-file');
const output = fs.createWriteStream('output.dat');

req.pipe(output);

// Pause after 1MB
let bytesReceived = 0;
req.on('data', function(chunk) {
  bytesReceived += chunk.length;
  if (bytesReceived > 1024 * 1024) {
    req.pause();
    console.log('Paused at 1MB');

    // Resume after 1 second
    setTimeout(function() {
      console.log('Resuming...');
      req.resume();
    }, 1000);
  }
});
```

### Bidirectional Streaming

Stream request and response simultaneously.

```javascript { .api }
/**
 * Request instances are duplex streams
 * They can be both read from and written to
 */
interface Request extends Stream.Duplex {
  readable: boolean;
  writable: boolean;
}
```

**Usage Examples:**

```javascript
const fs = require('fs');

// Upload and download simultaneously
const uploadStream = fs.createReadStream('upload.dat');
const downloadStream = fs.createWriteStream('download.dat');

uploadStream
  .pipe(request.post('http://example.com/process'))
  .pipe(downloadStream);

// Proxy request through
const http = require('http');

http.createServer(function(req, res) {
  // Proxy incoming request to backend
  req.pipe(request('http://backend.example.com' + req.url))
     .pipe(res);
}).listen(8080);

// Transform and upload stream
const transform = new Transform({
  transform(chunk, encoding, callback) {
    // Process chunk
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

fs.createReadStream('input.txt')
  .pipe(transform)
  .pipe(request.post('http://example.com/upload'));
```

### Aborting Requests

Cancel in-progress requests.

```javascript { .api }
/**
 * Abort the request
 * Emits 'abort' event and cleans up resources
 */
function Request.prototype.abort(): void;
```

**Usage Examples:**

```javascript
// Abort after timeout
const req = request('http://example.com/slow');

setTimeout(function() {
  req.abort();
}, 5000);

req.on('abort', function() {
  console.log('Request aborted');
});

req.on('error', function(err) {
  if (err.code === 'ECONNRESET') {
    console.log('Request was aborted');
  }
});

// Abort on user action
const req = request('http://example.com/large-file')
  .pipe(fs.createWriteStream('output.dat'));

process.on('SIGINT', function() {
  console.log('Aborting download...');
  req.abort();
  process.exit();
});
```

### Progress Tracking

Monitor upload and download progress.

```javascript
// Track download progress
let totalBytes = 0;
let receivedBytes = 0;

const req = request('http://example.com/large-file');

req.on('response', function(res) {
  totalBytes = parseInt(res.headers['content-length'], 10);
  console.log('Total size:', totalBytes, 'bytes');
});

req.on('data', function(chunk) {
  receivedBytes += chunk.length;
  const percent = (receivedBytes / totalBytes * 100).toFixed(2);
  console.log('Progress:', percent + '%');
});

req.pipe(fs.createWriteStream('output.dat'));

// Track upload progress
const fs = require('fs');
const fileSize = fs.statSync('upload.dat').size;
let uploadedBytes = 0;

const fileStream = fs.createReadStream('upload.dat');
const req = request.post('http://example.com/upload');

fileStream.on('data', function(chunk) {
  uploadedBytes += chunk.length;
  const percent = (uploadedBytes / fileSize * 100).toFixed(2);
  console.log('Upload progress:', percent + '%');
});

fileStream.pipe(req);

req.on('response', function(res) {
  console.log('Upload complete:', res.statusCode);
});
```

## Usage Patterns

### Download File with Progress

Download file with progress reporting:

```javascript
const fs = require('fs');

function downloadFile(url, dest, callback) {
  const file = fs.createWriteStream(dest);
  let receivedBytes = 0;
  let totalBytes = 0;

  const req = request(url);

  req.on('response', function(res) {
    if (res.statusCode !== 200) {
      callback(new Error('Bad status: ' + res.statusCode));
      return;
    }

    totalBytes = parseInt(res.headers['content-length'], 10);
    console.log('Downloading', totalBytes, 'bytes');
  });

  req.on('data', function(chunk) {
    receivedBytes += chunk.length;
    const percent = (receivedBytes / totalBytes * 100).toFixed(1);
    process.stdout.write(`\rProgress: ${percent}%`);
  });

  req.on('error', callback);

  file.on('finish', function() {
    console.log('\nDownload complete');
    file.close(callback);
  });

  req.pipe(file);
}

downloadFile('http://example.com/file.pdf', './file.pdf', function(err) {
  if (err) console.error('Download failed:', err.message);
});
```

### Stream Processing Pipeline

Process streaming data through multiple stages:

```javascript
const request = require('request');
const zlib = require('zlib');
const crypto = require('crypto');
const fs = require('fs');

// Download, decompress, hash, and save
request('http://example.com/data.json.gz')
  .pipe(zlib.createGunzip())
  .pipe(crypto.createHash('sha256').setEncoding('hex'))
  .pipe(process.stdout);

// Multi-stage transformation
const { Transform } = require('stream');

const jsonParser = new Transform({
  transform(chunk, encoding, callback) {
    // Parse and transform JSON
    try {
      const data = JSON.parse(chunk);
      this.push(JSON.stringify(data, null, 2));
      callback();
    } catch (err) {
      callback(err);
    }
  }
});

request('http://api.example.com/data')
  .pipe(jsonParser)
  .pipe(fs.createWriteStream('formatted.json'));
```

### Event-Driven Request Handling

Use events for fine-grained control:

```javascript
function smartRequest(url, options) {
  const req = request(url, options);
  const metrics = {
    startTime: Date.now(),
    requestSent: null,
    responseReceived: null,
    bytesReceived: 0,
    endTime: null
  };

  req.on('request', function() {
    metrics.requestSent = Date.now();
    console.log('Request sent after', metrics.requestSent - metrics.startTime, 'ms');
  });

  req.on('response', function(res) {
    metrics.responseReceived = Date.now();
    console.log('Response received after', metrics.responseReceived - metrics.startTime, 'ms');
    console.log('Status:', res.statusCode);
  });

  req.on('data', function(chunk) {
    metrics.bytesReceived += chunk.length;
  });

  req.on('end', function() {
    metrics.endTime = Date.now();
    console.log('Total time:', metrics.endTime - metrics.startTime, 'ms');
    console.log('Total bytes:', metrics.bytesReceived);
  });

  req.on('error', function(err) {
    console.error('Request failed:', err.message);
  });

  return req;
}

smartRequest('http://example.com/api/data');
```

### Conditional Streaming

Stream based on response headers:

```javascript
const req = request('http://example.com/data');

req.on('response', function(res) {
  const contentType = res.headers['content-type'];
  const contentLength = parseInt(res.headers['content-length'], 10);

  if (contentType.includes('application/json')) {
    // Buffer JSON responses
    let body = '';
    res.on('data', chunk => body += chunk);
    res.on('end', () => console.log(JSON.parse(body)));
  } else if (contentLength > 1024 * 1024) {
    // Stream large files to disk
    res.pipe(fs.createWriteStream('large-file.dat'));
  } else {
    // Buffer small files
    const chunks = [];
    res.on('data', chunk => chunks.push(chunk));
    res.on('end', () => console.log(Buffer.concat(chunks)));
  }
});
```

### Stream Error Handling

Robust error handling for streams:

```javascript
function safeStreamRequest(url, outputPath, callback) {
  const output = fs.createWriteStream(outputPath);
  const req = request(url);

  req.on('error', function(err) {
    output.destroy();
    fs.unlink(outputPath, () => {});
    callback(err);
  });

  output.on('error', function(err) {
    req.abort();
    fs.unlink(outputPath, () => {});
    callback(err);
  });

  output.on('finish', function() {
    callback(null);
  });

  req.pipe(output);
}

safeStreamRequest('http://example.com/file.dat', './output.dat', function(err) {
  if (err) {
    console.error('Stream failed:', err.message);
  } else {
    console.log('Stream succeeded');
  }
});
```

## Types

### Stream Interface

Request implements the Node.js Stream interface:

```javascript { .api }
interface Request extends Stream.Duplex {
  // Readable stream properties
  readable: boolean;
  pause(): void;
  resume(): void;
  pipe<T extends NodeJS.WritableStream>(destination: T, options?: { end?: boolean }): T;

  // Writable stream properties
  writable: boolean;
  write(chunk: any, encoding?: string, callback?: () => void): boolean;
  end(chunk?: any, encoding?: string, callback?: () => void): void;

  // Cleanup
  destroy(): void;
}
```

## Notes

### Streaming Best Practices

- **Always handle errors** on both request and destination streams
- **Use pipe() for automatic backpressure** handling
- **Clean up resources** (close files, abort requests) on errors
- **Consider memory usage** when buffering large responses
- **Monitor progress** for long-running streams
- **Use pause/resume** for flow control when needed
- **Abort requests** that are no longer needed to free resources

### Event Timing

- `request` event fires when underlying http.ClientRequest is created
- `response` event fires when response headers are received
- `data` events fire as response body chunks arrive
- `end` event fires when response body is complete
- `complete` event fires after `end` with full buffered body (if applicable)
- `error` event can fire at any point during request lifecycle

### Stream Behavior

- Request instances are duplex streams (both readable and writable)
- Writing to request streams the request body
- Reading from request streams the response body
- Piping to request sends data as request body
- Piping from request sends response body to destination
- Request body streaming must complete before response arrives
- Response streaming begins after headers are received

### Memory Considerations

- Piping streams avoids buffering entire contents in memory
- Callback-based requests buffer entire response in memory
- Use streaming for large files (> 10MB) to reduce memory usage
- Transform streams can process data incrementally
- Pause/resume enables manual memory management

### Error Handling

- Both request and response streams can emit errors
- Always attach error handlers to avoid uncaught exceptions
- Abort requests on error to clean up resources
- Clean up file handles and other resources on error
- Some errors (like ECONNRESET) may indicate aborted requests
