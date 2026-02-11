# Build a File Downloader

Create a Node.js utility that downloads files from URLs and saves them to disk efficiently using streaming.

## Requirements

Your downloader should:

1. Download files from HTTP/HTTPS URLs
2. Stream the response directly to a file (no buffering entire content in memory)
3. Support downloading large files efficiently
4. Handle download errors and file write errors
5. Provide progress feedback or completion notification
6. Support multiple concurrent downloads

## Use Cases

- Download images, videos, or documents from URLs
- Save API responses directly to files
- Efficiently handle large file downloads without memory issues

## Example Usage

```javascript
const downloader = require('./file-downloader');

// Single file download
downloader.download('https://httpbin.org/image/png', './output/image.png', (err) => {
  if (err) {
    console.error('Download failed:', err);
  } else {
    console.log('Download complete!');
  }
});

// Multiple files
downloader.downloadMultiple([
  { url: 'https://httpbin.org/image/jpeg', dest: './output/image1.jpg' },
  { url: 'https://httpbin.org/image/png', dest: './output/image2.png' }
], (err, results) => {
  console.log('All downloads complete');
});
```

## Expected Behavior

- Response data should be streamed directly to files
- Large files should download without consuming excessive memory
- File writing should happen incrementally as data arrives
- Errors in either the HTTP request or file writing should be caught

## Dependencies { .dependencies }

### request 2.74.1 { .dependency }

A simplified HTTP request client with built-in streaming support for efficient data transfer.

## Test Cases

### Test 1: Basic Download @test

Input: URL to downloadable file and destination path

Expected behavior: Should stream response to file and save successfully

### Test 2: Large File Handling @test

Input: URL to large file (simulated with /drip or /bytes endpoint)

Expected behavior: Should handle large downloads without buffering entire content

### Test 3: Error Handling @test

Input: Invalid URL or write-protected destination

Expected behavior: Should emit error event and handle gracefully
