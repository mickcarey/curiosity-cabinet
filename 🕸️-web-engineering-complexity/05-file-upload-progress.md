# File Upload Progress: The Progress Bar That Lies

## What Seems Simple
"Show a progress bar while the file uploads."

## The Actual Complexity

### 1. The Progress Event Deception
```javascript
xhr.upload.addEventListener('progress', (e) => {
  const percent = (e.loaded / e.total) * 100;
  // This is measuring bytes SENT from your computer
  // NOT bytes RECEIVED by the server
  // NOT bytes PROCESSED by the server
  // NOT bytes SAVED to storage

  // Your progress: 100%
  // Server: Still receiving data
  // Actual file: Not saved yet
});
```

### 2. The Chunking Calculation Nightmare
```javascript
// Upload 100MB file in 1MB chunks
const CHUNK_SIZE = 1024 * 1024;
let uploaded = 0;

async function uploadChunk(chunk, index) {
  // Progress for THIS chunk
  xhr.upload.onprogress = (e) => {
    const chunkProgress = e.loaded;
    const totalProgress = (index * CHUNK_SIZE) + chunkProgress;
    const percent = (totalProgress / fileSize) * 100;
    // But wait! What if chunks upload in parallel?
    // What if a chunk fails and retries?
    // What if chunks arrive out of order?
  };
}
```

### 3. The Mobile Network Reality
```javascript
// User on 4G, strong signal
// Upload starts: 5MB/s
progressBar.textContent = "2 minutes remaining";

// User enters elevator
// Speed drops to: 10KB/s
// Estimated time: 3 hours

// User exits elevator
// Speed spikes to: 10MB/s
// Estimated time: -5 minutes (goes backward!)
```

### 4. The Browser Cache Lies
```javascript
// User uploads same file twice
fetch('/upload', {
  method: 'POST',
  body: file
});

// Browser: "I'll cache this!"
// Progress instantly jumps to 100%
// Server never received anything
// Upload actually failed

// But progress bar shows success!
```

### 5. Proxy and CDN Buffering
```javascript
// Your progress events:
// [=========>] 100% - Data left your browser

// Reality:
// Browser → [100%] → Cloudflare → [43%] → Load Balancer → [12%] → Server → [0%]

// Cloudflare buffers entire request
// User sees 100%
// Waits 30 seconds
// "Upload failed" - server timeout

// User: "But it said 100%!"
```

### 6. The Multipart Form Boundary Problem
```javascript
const formData = new FormData();
formData.append('file', file);          // 100MB
formData.append('metadata', JSON.stringify(meta)); // 1KB

// Total size includes:
// - File data (100MB)
// - Metadata (1KB)
// - Boundary strings (--WebKitFormBoundaryXXX)
// - Content-Disposition headers
// - Content-Type headers
// - Extra newlines

// Actual upload size: 100.5MB
// Progress bar: Goes to 99.5% and stops
// Why? Didn't account for multipart overhead
```

### 7. The Compression Confusion
```javascript
// Server expects gzipped uploads
fetch('/upload', {
  headers: { 'Content-Encoding': 'gzip' },
  body: file
});

// Browser might compress on-the-fly
// Original: 100MB
// Compressed: 30MB
// Progress shows: 30% when actually done
// OR shows 333% if calculating wrong!
```

### 8. The Service Worker Interference
```javascript
// Service worker intercepts upload
self.addEventListener('fetch', event => {
  if (event.request.url.includes('/upload')) {
    // Clone request (doubles memory!)
    const clone = event.request.clone();

    // Cache for offline retry
    cacheUpload(clone);

    // Progress fires TWICE
    // Once for SW → Cache
    // Once for SW → Server
  }
});
```

### 9. The WebSocket Upload Chaos
```javascript
// Upload via WebSocket for real-time features
const ws = new WebSocket('wss://api.example.com');
const CHUNK_SIZE = 16384; // WebSocket frame limit

let sent = 0;
function sendChunk() {
  if (ws.bufferedAmount > 0) {
    // WebSocket is buffering!
    // sent !== actually sent
    // Progress bar freezes or jumps
    setTimeout(sendChunk, 10);
    return;
  }

  ws.send(chunk);
  sent += chunk.size;
  // But when did server ACTUALLY receive it?
}
```

### 10. The HTTP/2 Multiplexing Madness
```javascript
// HTTP/2 multiplexes streams
// Upload three files simultaneously

// File A: 10MB (priority: high)
// File B: 50MB (priority: low)
// File C: 5MB (priority: high)

// Progress bars show:
// A: 40% [=====>     ]
// B: 10% [=>         ]
// C: 80% [========>  ]

// Reality: Server reassembling frames
// Actual server progress:
// A: 10%
// B: 60% (low priority but more frames arrived)
// C: 20%

// Nothing matches!
```

### 11. The Resume Upload Catastrophe
```javascript
// Upload fails at 60%
// User clicks "Resume"

async function resumeUpload() {
  // Ask server what it has
  const response = await fetch('/upload/status');
  const { bytesReceived } = await response.json();

  // Server says: 40MB received
  // But client sent: 60MB
  // Where's the missing 20MB?
  // - Proxy buffering
  // - Server processing lag
  // - Partial write to disk

  // Resume from where?
  file.slice(bytesReceived); // Might re-upload data
  file.slice(60MB); // Might skip data
}
```

### 12. The Progress Animation Uncanny Valley
```javascript
// Real network progress: Jerky
// [=>    ] 10%
// [==>   ] 15%
// [=====>] 50% (sudden jump)
// [=====>] 50% (stalls)
// [======>] 95% (jump)
// [======>] 95% (stalls forever)

// "Smooth" fake progress:
let display = 0;
const actual = 50;

function animate() {
  if (display < actual) {
    display += 0.5;
    progressBar.style.width = display + '%';
  }
  // Looks smooth but it's lying about actual progress
  requestAnimationFrame(animate);
}
```

### 13. The Time Remaining Paradox
```javascript
class TimeEstimator {
  calculate(loaded, total, elapsed) {
    const rate = loaded / elapsed;
    const remaining = (total - loaded) / rate;

    // Problems:
    // 1. Network speed varies wildly
    // 2. First few seconds are slow (TCP slow start)
    // 3. Last few seconds are slow (server processing)
    // 4. Rate calculation needs smoothing

    // Attempt 2: Exponential moving average
    this.averageRate = 0.9 * this.averageRate + 0.1 * rate;
    // Still jumps around

    // Attempt 3: Multiple samples
    this.samples.push(rate);
    if (this.samples.length > 10) this.samples.shift();
    // Now it's always wrong, just smoothly wrong
  }
}
```

### 14. The Backend Processing Void
```javascript
// File upload: 100% ✓
// Progress bar disappears
// User waits...

// Meanwhile on server:
// - Virus scan: 30 seconds
// - Generate thumbnails: 10 seconds
// - Move to S3: 20 seconds
// - Update database: 5 seconds
// - Send webhooks: 10 seconds

// User: "Is it done? Did it fail? Should I reload?"
// UI: 🤷
```

## Real Implementation Horror

```javascript
class RealWorldUploadProgress {
  constructor() {
    this.networkProgress = 0;
    this.serverAckProgress = 0;
    this.processingProgress = 0;
    this.retryCount = 0;
    this.fakeProgress = 0;
    this.stalled = false;
    this.timeEstimates = [];
  }

  async upload(file) {
    // Start fake progress so user doesn't think it's broken
    this.startFakeProgress();

    try {
      // Try upload
      await this.attemptUpload(file);
    } catch (e) {
      // Pretend we're still making progress
      this.fakeProgress = 70;

      // Actually retry
      await this.exponentialBackoffRetry(file);
    }

    // Jump to 95% and wait for server
    this.showProcessingState();

    // Poll for actual completion
    await this.pollUntilComplete();
  }

  updateProgressBar() {
    // Take maximum of all progress sources
    // So bar never goes backward
    const shown = Math.max(
      this.networkProgress,
      this.fakeProgress,
      this.minProgressForTime(Date.now() - this.startTime),
      this.previousProgress // Never go backward!
    );

    // Never show 100% unless truly done
    const capped = Math.min(shown, 99.9);

    // Smooth it out
    this.smoothedProgress += (capped - this.smoothedProgress) * 0.1;
  }
}
```

## Why Every Site's Upload Is Different

- **YouTube**: Shows multiple progress bars (uploading, processing, SD ready, HD processing)
- **Google Drive**: Lies until the last second, then shows real progress
- **Dropbox**: Shows speed graph instead of percentage
- **WeTransfer**: Just shows an animation, no real progress
- **S3 Direct Upload**: Shows individual part uploads for multipart
- **Instagram**: No progress at all, just spinner

Nobody has solved this. The progress bar is a lie we all agree to tell.