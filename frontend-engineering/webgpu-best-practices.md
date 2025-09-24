# WebGPU Best Practices & Resources: The Wisdom of the GPU Elders 📜

After spending countless hours debugging shader compilation errors and wondering why your frame rate is 3 FPS, you learn a thing or two. Here's the accumulated wisdom of WebGPU developers who've suffered so you don't have to!

## Performance Best Practices 🚀

### 1. Resource Management: Don't Be a Hoarder

```javascript
// Bad: Creating resources every frame
function renderBadly() {
  const buffer = device.createBuffer({ /* ... */ }); // 🚫 Memory leak incoming!
  const pipeline = device.createRenderPipeline({ /* ... */ }); // 🚫 Expensive!
  // render...
}

// Good: Create once, reuse forever
class ResourceManager {
  constructor(device) {
    this.device = device;
    this.buffers = new Map();
    this.pipelines = new Map();
    this.textures = new Map();
  }

  getOrCreateBuffer(key, descriptor) {
    if (!this.buffers.has(key)) {
      this.buffers.set(key, this.device.createBuffer(descriptor));
    }
    return this.buffers.get(key);
  }

  // Pool pattern for temporary resources
  borrowBuffer(size) {
    // Get from pool or create new
    const buffer = this.bufferPool.pop() ||
                   this.device.createBuffer({ size, usage: /* ... */ });
    return buffer;
  }

  returnBuffer(buffer) {
    this.bufferPool.push(buffer);
  }
}
```

### 2. Command Recording: Batch Like Your Life Depends On It

```javascript
// Bad: One command buffer per draw call
for (let mesh of meshes) {
  const encoder = device.createCommandEncoder();
  const pass = encoder.beginRenderPass(/* ... */);
  pass.setPipeline(pipeline);
  pass.draw(mesh.vertexCount);
  pass.end();
  device.queue.submit([encoder.finish()]); // 🚫 GPU go brrr... slowly
}

// Good: Batch everything
const encoder = device.createCommandEncoder();
const pass = encoder.beginRenderPass(/* ... */);
pass.setPipeline(pipeline);

for (let mesh of meshes) {
  pass.setVertexBuffer(0, mesh.vertexBuffer);
  pass.setBindGroup(0, mesh.bindGroup);
  pass.draw(mesh.vertexCount);
}

pass.end();
device.queue.submit([encoder.finish()]); // ✅ One submission to rule them all
```

### 3. Buffer Updates: The Art of Not Stalling

```javascript
// Bad: Updating buffers that are in use
device.queue.writeBuffer(uniformBuffer, 0, data); // GPU might be using this!
// Immediate draw... GPU sad 😢

// Good: Double/Triple buffering
class UniformBufferRing {
  constructor(device, size, count = 3) {
    this.buffers = Array(count).fill(null).map(() =>
      device.createBuffer({
        size,
        usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST,
      })
    );
    this.index = 0;
  }

  getNext() {
    const buffer = this.buffers[this.index];
    this.index = (this.index + 1) % this.buffers.length;
    return buffer;
  }
}
```

## Memory Management Patterns 🧠

### The Dynamic Buffer Strategy

```javascript
class DynamicVertexBuffer {
  constructor(device, initialSize = 1024 * 1024) {
    this.device = device;
    this.size = initialSize;
    this.used = 0;
    this.buffer = this.createBuffer(initialSize);
  }

  createBuffer(size) {
    return this.device.createBuffer({
      size,
      usage: GPUBufferUsage.VERTEX | GPUBufferUsage.COPY_DST,
    });
  }

  write(data) {
    // Grow if needed
    if (this.used + data.byteLength > this.size) {
      const newSize = Math.max(this.size * 2, this.used + data.byteLength);
      const newBuffer = this.createBuffer(newSize);

      // Copy old data (if needed)
      const encoder = this.device.createCommandEncoder();
      encoder.copyBufferToBuffer(this.buffer, 0, newBuffer, 0, this.used);
      this.device.queue.submit([encoder.finish()]);

      this.buffer.destroy(); // Clean up old buffer
      this.buffer = newBuffer;
      this.size = newSize;
    }

    this.device.queue.writeBuffer(this.buffer, this.used, data);
    const offset = this.used;
    this.used += data.byteLength;
    return offset;
  }

  reset() {
    this.used = 0;
  }
}
```

### The Texture Atlas Pattern

```javascript
class TextureAtlas {
  constructor(device, size = 4096) {
    this.device = device;
    this.size = size;
    this.regions = new Map();
    this.packer = new BinPacker(size, size); // Imaginary bin packing library

    this.texture = device.createTexture({
      size: [size, size],
      format: 'rgba8unorm',
      usage: GPUTextureUsage.TEXTURE_BINDING |
             GPUTextureUsage.COPY_DST |
             GPUTextureUsage.RENDER_ATTACHMENT,
    });
  }

  addTexture(key, imageData) {
    const rect = this.packer.pack(imageData.width, imageData.height);
    if (!rect) {
      throw new Error('Atlas is full! Time for texture tetris.');
    }

    // Update atlas texture
    this.device.queue.writeTexture(
      { texture: this.texture, origin: [rect.x, rect.y] },
      imageData.data,
      { bytesPerRow: imageData.width * 4 },
      { width: imageData.width, height: imageData.height }
    );

    // Store UV coordinates
    this.regions.set(key, {
      u0: rect.x / this.size,
      v0: rect.y / this.size,
      u1: (rect.x + rect.width) / this.size,
      v1: (rect.y + rect.height) / this.size,
    });
  }

  getUVs(key) {
    return this.regions.get(key);
  }
}
```

## Shader Optimization Tips 🎯

### 1. Minimize Texture Samples

```wgsl
// Bad: Sampling in a loop
@fragment
fn fs_bad(@location(0) uv: vec2<f32>) -> @location(0) vec4<f32> {
  var color = vec4<f32>(0.0);
  for (var i = 0; i < 16; i++) {
    color += textureSample(myTexture, mySampler, uv + vec2<f32>(f32(i) * 0.01, 0.0));
  }
  return color / 16.0;
}

// Good: Use mipmaps or pre-filter
@fragment
fn fs_good(@location(0) uv: vec2<f32>) -> @location(0) vec4<f32> {
  // Use LOD bias for blur effect
  return textureSampleBias(myTexture, mySampler, uv, 3.0);
}
```

### 2. Branch Optimization

```wgsl
// Bad: Divergent branching
@fragment
fn fs_branchy(@location(0) uv: vec2<f32>) -> @location(0) vec4<f32> {
  if (uv.x > 0.5) {
    // Complex calculation A
    return expensiveFunction1();
  } else {
    // Complex calculation B
    return expensiveFunction2();
  }
}

// Good: Use math instead of branches
@fragment
fn fs_mathy(@location(0) uv: vec2<f32>) -> @location(0) vec4<f32> {
  let factor = step(0.5, uv.x);
  return mix(calculation2(), calculation1(), factor);
}
```

### 3. Precision Awareness

```wgsl
// Use the right precision for the job
struct VertexData {
  @location(0) position: vec3<f32>,     // Full precision for positions
  @location(1) normal: vec3<f16>,       // Half precision often fine for normals
  @location(2) texcoord: vec2<f16>,     // Half precision for UVs
  @location(3) color: vec4<u8>,         // 8-bit for colors
}
```

## Common Pitfalls and How to Avoid Them 🕳️

### 1. The Synchronization Trap

```javascript
// Problem: Reading buffer immediately after write
device.queue.writeBuffer(buffer, 0, data);
await buffer.mapAsync(GPUMapMode.READ); // 🚫 Might read stale data!

// Solution: Use onSubmittedWorkDone
device.queue.writeBuffer(buffer, 0, data);
await device.queue.onSubmittedWorkDone();
await buffer.mapAsync(GPUMapMode.READ); // ✅ Now it's safe
```

### 2. The Alignment Apocalypse

```wgsl
// Problem: Misaligned struct
struct BadUniforms {
  time: f32,      // Offset: 0
  resolution: vec2<f32>, // Offset: 4 🚫 vec2 needs 8-byte alignment!
}

// Solution: Proper alignment
struct GoodUniforms {
  time: f32,      // Offset: 0
  _padding: f32,  // Offset: 4
  resolution: vec2<f32>, // Offset: 8 ✅
}

// Or use the @align attribute
struct BetterUniforms {
  time: f32,
  @align(8) resolution: vec2<f32>,
}
```

### 3. The Validation Layer Nightmare

```javascript
// Enable validation in development
const device = await adapter.requestDevice({
  requiredFeatures: ['timestamp-query'],
  requiredLimits: {
    maxBufferSize: 268435456,
  },
});

// Add error handling
device.addEventListener('uncapturederror', (event) => {
  console.error('GPU Error:', event.error);

  // Send to error tracking service
  if (production) {
    errorTracker.log('WebGPU Error', {
      message: event.error.message,
      // Include relevant context
    });
  }
});
```

## Testing Strategies 🧪

### 1. GPU Feature Detection

```javascript
class FeatureDetector {
  static async checkSupport() {
    const report = {
      webgpu: false,
      features: [],
      limits: {},
      textureFormats: [],
    };

    if (!navigator.gpu) {
      return report;
    }

    try {
      const adapter = await navigator.gpu.requestAdapter();
      if (!adapter) return report;

      report.webgpu = true;
      report.features = [...adapter.features];

      // Check texture format support
      const formats = ['rgba8unorm', 'rgba16float', 'r32float', 'bc1-rgba-unorm'];
      for (const format of formats) {
        try {
          const device = await adapter.requestDevice();
          device.createTexture({
            size: [1, 1],
            format,
            usage: GPUTextureUsage.TEXTURE_BINDING,
          });
          report.textureFormats.push(format);
        } catch {}
      }

      // Check limits
      const device = await adapter.requestDevice();
      report.limits = {
        maxTextureDimension2D: device.limits.maxTextureDimension2D,
        maxBufferSize: device.limits.maxBufferSize,
        maxVertexBuffers: device.limits.maxVertexBuffers,
        maxBindGroups: device.limits.maxBindGroups,
      };
    } catch (e) {
      console.error('Feature detection failed:', e);
    }

    return report;
  }
}
```

### 2. Performance Profiling

```javascript
class GPUProfiler {
  constructor(device) {
    this.device = device;
    this.querySet = device.createQuerySet({
      type: 'timestamp',
      count: 32,
    });
    this.resolveBuffer = device.createBuffer({
      size: 32 * 8,
      usage: GPUBufferUsage.QUERY_RESOLVE | GPUBufferUsage.COPY_SRC,
    });
  }

  beginPass(encoder, index) {
    encoder.writeTimestamp(this.querySet, index * 2);
  }

  endPass(encoder, index) {
    encoder.writeTimestamp(this.querySet, index * 2 + 1);
  }

  async readTimings() {
    const staging = this.device.createBuffer({
      size: this.resolveBuffer.size,
      usage: GPUBufferUsage.MAP_READ | GPUBufferUsage.COPY_DST,
    });

    const encoder = this.device.createCommandEncoder();
    encoder.resolveQuerySet(this.querySet, 0, 32, this.resolveBuffer, 0);
    encoder.copyBufferToBuffer(this.resolveBuffer, 0, staging, 0, staging.size);
    this.device.queue.submit([encoder.finish()]);

    await staging.mapAsync(GPUMapMode.READ);
    const timings = new BigInt64Array(staging.getMappedRange());

    const results = [];
    for (let i = 0; i < 16; i++) {
      const start = timings[i * 2];
      const end = timings[i * 2 + 1];
      if (end > start) {
        results.push({
          pass: i,
          duration: Number(end - start) / 1000000, // Convert to milliseconds
        });
      }
    }

    staging.unmap();
    return results;
  }
}
```

## Essential Resources 📚

### Official Documentation
- **[WebGPU Spec](https://www.w3.org/TR/webgpu/)** - The bible of WebGPU (warning: very dry reading)
- **[WGSL Spec](https://www.w3.org/TR/WGSL/)** - Everything about the shading language
- **[MDN WebGPU](https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API)** - More digestible documentation

### Learning Resources
- **[WebGPU Fundamentals](https://webgpufundamentals.org/)** - Like WebGL Fundamentals but cooler
- **[Raw WebGPU](https://alain.xyz/blog/raw-webgpu)** - Excellent deep dive into WebGPU
- **[Austin Eng's WebGPU Samples](https://austin-eng.com/webgpu-samples/)** - Canonical examples

### Tools & Libraries
- **[Babylon.js](https://www.babylonjs.com/)** - Full-featured 3D engine with WebGPU support
- **[Three.js WebGPU](https://threejs.org/examples/#webgpu)** - Three.js experimental WebGPU renderer
- **[wgpu](https://wgpu.rs/)** - Rust implementation (great for native + web)
- **[Dawn](https://dawn.googlesource.com/dawn)** - Google's WebGPU implementation
- **[Tint](https://dawn.googlesource.com/tint)** - WGSL compiler and validator

### Community Haunts
- **[WebGPU Discord](https://discord.gg/webgpu)** - Where the GPU wizards hang out
- **[Graphics Programming Discord](https://discord.gg/graphicsprogramming)** - Broader graphics community
- **[r/WebGPU](https://www.reddit.com/r/webgpu/)** - Reddit community (surprisingly active!)
- **[WebGPU GitHub Discussions](https://github.com/gpuweb/gpuweb/discussions)** - Official spec discussions

## Browser Compatibility Magic 🌐

```javascript
// The ultimate compatibility checker
class WebGPUCompat {
  static async init() {
    // Check for WebGPU support
    if (!navigator.gpu) {
      return this.fallbackToWebGL();
    }

    const adapter = await navigator.gpu.requestAdapter();
    if (!adapter) {
      console.warn('No WebGPU adapter found, falling back to WebGL');
      return this.fallbackToWebGL();
    }

    // Check for required features
    const requiredFeatures = [];
    const optionalFeatures = ['timestamp-query', 'depth-clip-control'];

    const availableFeatures = [...adapter.features];
    const hasRequired = requiredFeatures.every(f => availableFeatures.includes(f));

    if (!hasRequired) {
      return this.fallbackToWebGL();
    }

    // Request device with optional features
    const deviceDescriptor = {
      requiredFeatures,
      requiredLimits: {
        maxBindGroups: 4,
        maxUniformBufferBindingSize: 65536,
      },
    };

    // Add optional features that are available
    for (const feature of optionalFeatures) {
      if (availableFeatures.includes(feature)) {
        deviceDescriptor.requiredFeatures.push(feature);
      }
    }

    try {
      const device = await adapter.requestDevice(deviceDescriptor);
      return { type: 'webgpu', device, adapter };
    } catch (e) {
      console.error('Failed to create WebGPU device:', e);
      return this.fallbackToWebGL();
    }
  }

  static fallbackToWebGL() {
    const canvas = document.createElement('canvas');
    const gl = canvas.getContext('webgl2') || canvas.getContext('webgl');

    if (!gl) {
      return { type: 'none', error: 'No graphics API available' };
    }

    return { type: gl.canvas.getContext('webgl2') ? 'webgl2' : 'webgl', context: gl };
  }
}
```

## Future-Proofing Your Code 🔮

```javascript
// Prepare for upcoming features
class FutureWebGPU {
  static async checkUpcomingFeatures(adapter) {
    const features = {
      // Current features
      timestampQuery: adapter.features.has('timestamp-query'),
      depthClipControl: adapter.features.has('depth-clip-control'),

      // Upcoming features (check periodically)
      rayTracing: adapter.features.has('ray-tracing'), // One day...
      meshShaders: adapter.features.has('mesh-shader'), // Coming soon™
      bindless: adapter.features.has('bindless'), // The dream
    };

    return features;
  }

  // Abstract API for future changes
  static createCompatiblePipeline(device, descriptor) {
    // Add version checks and polyfills
    const version = this.getWebGPUVersion();

    if (version >= 2.0) {
      // Use new pipeline features
      descriptor.advanced = true;
    }

    return device.createRenderPipeline(descriptor);
  }
}
```

## The Philosophical Conclusion 🧘

WebGPU is not just an API; it's a journey. A journey from the comfortable lands of WebGL to the powerful but sometimes treacherous territories of modern GPU programming.

Remember:
- **Start simple** - Don't try to build Cyberpunk 2077 on your first day
- **Profile everything** - Assumptions are the enemy of performance
- **Read the specs** - They're boring but will save you hours of debugging
- **Join the community** - We're all figuring this out together
- **Have fun** - You're literally commanding thousands of cores to do your bidding!

### The WebGPU Developer's Mantra

> "I shall not create resources in the render loop.
> I shall batch my commands wisely.
> I shall respect the alignment requirements.
> I shall profile before I optimize.
> And I shall always check if WebGPU is actually supported."

---

*May your shaders compile on the first try and your frame times be consistent! Happy GPU programming!* 🚀✨