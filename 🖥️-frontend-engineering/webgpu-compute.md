# WebGPU Compute: When Graphics Cards Do Math Homework 🧮

Welcome to the wild side of WebGPU – compute shaders! This is where we stop caring about pretty pixels and start using the GPU as a massive parallel calculator. It's like discovering your gaming rig is secretly a supercomputer!

## What Are Compute Shaders? 🤖

Imagine you have a task that needs to be done 1 million times. You could:
1. Do it sequentially on the CPU (boring, slow)
2. Hire 1 million interns (expensive, chaotic)
3. Use a compute shader (fast, elegant, makes you look smart at parties)

Compute shaders are programs that run on the GPU without caring about graphics. They're perfect for:
- Machine learning inference
- Physics simulations
- Image processing
- Cryptography (but please be nice)
- Anything embarrassingly parallel

## Your First Compute Shader: Parallel Addition 🎯

Let's start simple – adding two arrays together. It's like "Hello World" but with more math:

```javascript
// The JavaScript side
async function setupCompute(device) {
  // Create buffers for our data
  const BUFFER_SIZE = 1000000; // 1 million numbers!
  const input1 = new Float32Array(BUFFER_SIZE).map(() => Math.random());
  const input2 = new Float32Array(BUFFER_SIZE).map(() => Math.random());

  // Create GPU buffers
  const buffer1 = device.createBuffer({
    size: input1.byteLength,
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST,
  });
  const buffer2 = device.createBuffer({
    size: input2.byteLength,
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_DST,
  });
  const resultBuffer = device.createBuffer({
    size: input1.byteLength,
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_SRC,
  });

  // Upload data to GPU
  device.queue.writeBuffer(buffer1, 0, input1);
  device.queue.writeBuffer(buffer2, 0, input2);

  // Create the compute shader
  const computeShaderModule = device.createShaderModule({
    code: `
      @group(0) @binding(0) var<storage, read> input1: array<f32>;
      @group(0) @binding(1) var<storage, read> input2: array<f32>;
      @group(0) @binding(2) var<storage, read_write> output: array<f32>;

      @compute @workgroup_size(256)
      fn main(@builtin(global_invocation_id) global_id: vec3<u32>) {
        let index = global_id.x;

        // Bounds check - important!
        if (index >= arrayLength(&input1)) {
          return;
        }

        // The actual computation
        output[index] = input1[index] + input2[index];
      }
    `
  });

  // Create compute pipeline
  const computePipeline = device.createComputePipeline({
    layout: 'auto',
    compute: {
      module: computeShaderModule,
      entryPoint: 'main',
    }
  });

  // Create bind group
  const bindGroup = device.createBindGroup({
    layout: computePipeline.getBindGroupLayout(0),
    entries: [
      { binding: 0, resource: { buffer: buffer1 }},
      { binding: 1, resource: { buffer: buffer2 }},
      { binding: 2, resource: { buffer: resultBuffer }},
    ],
  });

  // Run the compute shader!
  const commandEncoder = device.createCommandEncoder();
  const passEncoder = commandEncoder.beginComputePass();
  passEncoder.setPipeline(computePipeline);
  passEncoder.setBindGroup(0, bindGroup);
  passEncoder.dispatchWorkgroups(Math.ceil(BUFFER_SIZE / 256));
  passEncoder.end();

  // Get the results back
  const readBuffer = device.createBuffer({
    size: resultBuffer.size,
    usage: GPUBufferUsage.MAP_READ | GPUBufferUsage.COPY_DST,
  });
  commandEncoder.copyBufferToBuffer(resultBuffer, 0, readBuffer, 0, resultBuffer.size);

  device.queue.submit([commandEncoder.finish()]);

  // Read the results
  await readBuffer.mapAsync(GPUMapMode.READ);
  const results = new Float32Array(readBuffer.getMappedRange());
  console.log("First 10 results:", results.slice(0, 10));
}
```

## Real-World Example: Particle System 🌟

Let's create a particle system that runs entirely on the GPU:

```wgsl
// Particle structure
struct Particle {
  position: vec3<f32>,
  velocity: vec3<f32>,
  life: f32,
  size: f32,
  color: vec4<f32>,
}

struct SimParams {
  deltaTime: f32,
  gravity: vec3<f32>,
  windForce: vec3<f32>,
  particleCount: u32,
}

@group(0) @binding(0) var<storage, read_write> particles: array<Particle>;
@group(0) @binding(1) var<uniform> params: SimParams;

// Pseudo-random number generator
fn rand(seed: vec2<f32>) -> f32 {
  return fract(sin(dot(seed, vec2<f32>(12.9898, 78.233))) * 43758.5453);
}

@compute @workgroup_size(256)
fn updateParticles(@builtin(global_invocation_id) global_id: vec3<u32>) {
  let index = global_id.x;
  if (index >= params.particleCount) {
    return;
  }

  var particle = particles[index];

  // Update physics
  particle.velocity += params.gravity * params.deltaTime;
  particle.velocity += params.windForce * params.deltaTime;

  // Add some turbulence
  let turbulence = vec3<f32>(
    rand(vec2<f32>(f32(index), particle.position.x)) - 0.5,
    rand(vec2<f32>(f32(index), particle.position.y)) - 0.5,
    rand(vec2<f32>(f32(index), particle.position.z)) - 0.5
  ) * 2.0;
  particle.velocity += turbulence * params.deltaTime;

  // Update position
  particle.position += particle.velocity * params.deltaTime;

  // Update life
  particle.life -= params.deltaTime;

  // Respawn dead particles
  if (particle.life <= 0.0) {
    particle.position = vec3<f32>(
      rand(vec2<f32>(f32(index), 0.0)) * 10.0 - 5.0,
      10.0,
      rand(vec2<f32>(f32(index), 1.0)) * 10.0 - 5.0
    );
    particle.velocity = vec3<f32>(0.0, -1.0, 0.0);
    particle.life = 5.0 + rand(vec2<f32>(f32(index), 2.0)) * 5.0;
    particle.color = vec4<f32>(
      rand(vec2<f32>(f32(index), 3.0)),
      rand(vec2<f32>(f32(index), 4.0)),
      rand(vec2<f32>(f32(index), 5.0)),
      1.0
    );
  }

  // Fade out based on life
  particle.color.a = particle.life / 10.0;

  particles[index] = particle;
}
```

## Advanced Compute: Matrix Multiplication 🔢

Here's where things get spicy – implementing matrix multiplication that would make your linear algebra professor proud:

```wgsl
// Optimized matrix multiplication using shared memory
struct Matrices {
  A: array<array<f32, 1024>, 1024>,
  B: array<array<f32, 1024>, 1024>,
  C: array<array<f32, 1024>, 1024>,
}

@group(0) @binding(0) var<storage, read> matrices: Matrices;
@group(0) @binding(1) var<storage, read_write> result: Matrices;

const TILE_SIZE = 16u;
var<workgroup> tileA: array<array<f32, TILE_SIZE>, TILE_SIZE>;
var<workgroup> tileB: array<array<f32, TILE_SIZE>, TILE_SIZE>;

@compute @workgroup_size(TILE_SIZE, TILE_SIZE)
fn matmul(
  @builtin(global_invocation_id) global_id: vec3<u32>,
  @builtin(local_invocation_id) local_id: vec3<u32>,
  @builtin(workgroup_id) workgroup_id: vec3<u32>
) {
  let row = global_id.y;
  let col = global_id.x;
  let localRow = local_id.y;
  let localCol = local_id.x;

  var sum = 0.0;

  // Tile-based computation for cache efficiency
  let numTiles = 1024u / TILE_SIZE;
  for (var t = 0u; t < numTiles; t++) {
    // Load tiles into shared memory
    let tileRow = workgroup_id.y * TILE_SIZE + localRow;
    let tileCol = t * TILE_SIZE + localCol;
    if (tileRow < 1024u && tileCol < 1024u) {
      tileA[localRow][localCol] = matrices.A[tileRow][tileCol];
    }

    let tileRow2 = t * TILE_SIZE + localRow;
    let tileCol2 = workgroup_id.x * TILE_SIZE + localCol;
    if (tileRow2 < 1024u && tileCol2 < 1024u) {
      tileB[localRow][localCol] = matrices.B[tileRow2][tileCol2];
    }

    // Synchronize to ensure all threads have loaded their data
    workgroupBarrier();

    // Compute partial sum for this tile
    for (var k = 0u; k < TILE_SIZE; k++) {
      sum += tileA[localRow][k] * tileB[k][localCol];
    }

    workgroupBarrier();
  }

  // Write result
  if (row < 1024u && col < 1024u) {
    result.C[row][col] = sum;
  }
}
```

## Image Processing with Compute 🖼️

Let's build a real-time image filter that would make Instagram jealous:

```wgsl
// Gaussian blur with compute shaders
@group(0) @binding(0) var inputTexture: texture_2d<f32>;
@group(0) @binding(1) var outputTexture: texture_storage_2d<rgba8unorm, write>;

const KERNEL_SIZE = 5;
const kernel: array<f32, 25> = array<f32, 25>(
  1.0/256.0,  4.0/256.0,  6.0/256.0,  4.0/256.0, 1.0/256.0,
  4.0/256.0, 16.0/256.0, 24.0/256.0, 16.0/256.0, 4.0/256.0,
  6.0/256.0, 24.0/256.0, 36.0/256.0, 24.0/256.0, 6.0/256.0,
  4.0/256.0, 16.0/256.0, 24.0/256.0, 16.0/256.0, 4.0/256.0,
  1.0/256.0,  4.0/256.0,  6.0/256.0,  4.0/256.0, 1.0/256.0
);

@compute @workgroup_size(8, 8)
fn gaussianBlur(@builtin(global_invocation_id) global_id: vec3<u32>) {
  let dimensions = textureDimensions(inputTexture);
  let coord = vec2<i32>(global_id.xy);

  if (coord.x >= dimensions.x || coord.y >= dimensions.y) {
    return;
  }

  var color = vec4<f32>(0.0);

  for (var i = -2; i <= 2; i++) {
    for (var j = -2; j <= 2; j++) {
      let sampleCoord = coord + vec2<i32>(i, j);
      let kernelIndex = (i + 2) * 5 + (j + 2);

      // Clamp to texture bounds
      let clampedCoord = clamp(sampleCoord, vec2<i32>(0), dimensions - 1);
      color += textureLoad(inputTexture, clampedCoord, 0) * kernel[kernelIndex];
    }
  }

  textureStore(outputTexture, coord, color);
}

// Edge detection using Sobel operator
@compute @workgroup_size(8, 8)
fn sobelEdgeDetection(@builtin(global_invocation_id) global_id: vec3<u32>) {
  let coord = vec2<i32>(global_id.xy);
  let dimensions = textureDimensions(inputTexture);

  if (coord.x >= dimensions.x || coord.y >= dimensions.y) {
    return;
  }

  // Sobel X kernel: [-1, 0, 1; -2, 0, 2; -1, 0, 1]
  // Sobel Y kernel: [-1, -2, -1; 0, 0, 0; 1, 2, 1]
  var gx = 0.0;
  var gy = 0.0;

  for (var i = -1; i <= 1; i++) {
    for (var j = -1; j <= 1; j++) {
      let sampleCoord = clamp(coord + vec2<i32>(i, j), vec2<i32>(0), dimensions - 1);
      let pixel = textureLoad(inputTexture, sampleCoord, 0);
      let gray = dot(pixel.rgb, vec3<f32>(0.299, 0.587, 0.114));

      // Sobel X
      if (i != 0) {
        gx += gray * f32(i) * (2.0 - abs(f32(j)));
      }

      // Sobel Y
      if (j != 0) {
        gy += gray * f32(j) * (2.0 - abs(f32(i)));
      }
    }
  }

  let magnitude = sqrt(gx * gx + gy * gy);
  let edge = vec4<f32>(magnitude, magnitude, magnitude, 1.0);
  textureStore(outputTexture, coord, edge);
}
```

## Machine Learning on the GPU 🧠

Yes, you can do neural network inference in WebGPU! Here's a simple example:

```wgsl
// Simple neural network layer
struct NeuralLayer {
  weights: array<array<f32, 128>, 128>,
  bias: array<f32, 128>,
}

@group(0) @binding(0) var<storage, read> input: array<f32, 128>;
@group(0) @binding(1) var<storage, read> layer: NeuralLayer;
@group(0) @binding(2) var<storage, read_write> output: array<f32, 128>;

fn relu(x: f32) -> f32 {
  return max(0.0, x);
}

fn sigmoid(x: f32) -> f32 {
  return 1.0 / (1.0 + exp(-x));
}

@compute @workgroup_size(128)
fn forward_pass(@builtin(global_invocation_id) global_id: vec3<u32>) {
  let neuron_idx = global_id.x;

  if (neuron_idx >= 128u) {
    return;
  }

  var sum = layer.bias[neuron_idx];

  // Compute weighted sum
  for (var i = 0u; i < 128u; i++) {
    sum += input[i] * layer.weights[neuron_idx][i];
  }

  // Apply activation function
  output[neuron_idx] = relu(sum);
}

// Batch processing for efficiency
struct BatchData {
  samples: array<array<f32, 128>, 64>, // 64 samples, 128 features each
}

@group(0) @binding(0) var<storage, read> batch: BatchData;
@group(0) @binding(1) var<storage, read> model: NeuralLayer;
@group(0) @binding(2) var<storage, read_write> predictions: array<array<f32, 128>, 64>;

@compute @workgroup_size(8, 16) // Process 8 samples × 16 neurons at once
fn batch_inference(
  @builtin(global_invocation_id) global_id: vec3<u32>
) {
  let sample_idx = global_id.x;
  let neuron_idx = global_id.y;

  if (sample_idx >= 64u || neuron_idx >= 128u) {
    return;
  }

  var sum = model.bias[neuron_idx];
  for (var i = 0u; i < 128u; i++) {
    sum += batch.samples[sample_idx][i] * model.weights[neuron_idx][i];
  }

  predictions[sample_idx][neuron_idx] = sigmoid(sum);
}
```

## Performance Optimization Tips 🏎️

### 1. Workgroup Size Matters

```javascript
// Finding optimal workgroup size
async function findOptimalWorkgroupSize(device, dataSize) {
  const sizes = [64, 128, 256, 512];
  const timings = {};

  for (let size of sizes) {
    const start = performance.now();

    // Run compute with this workgroup size
    await runComputeWithSize(device, dataSize, size);

    timings[size] = performance.now() - start;
  }

  console.log("Workgroup timings:", timings);
  return Object.entries(timings).sort((a, b) => a[1] - b[1])[0][0];
}
```

### 2. Memory Access Patterns

```wgsl
// Bad: Random memory access
@compute @workgroup_size(256)
fn bad_pattern(@builtin(global_invocation_id) id: vec3<u32>) {
  let random_index = u32(rand(vec2<f32>(f32(id.x), 0.0)) * 1000000.0);
  output[id.x] = input[random_index]; // Cache misses galore!
}

// Good: Coalesced memory access
@compute @workgroup_size(256)
fn good_pattern(@builtin(global_invocation_id) id: vec3<u32>) {
  output[id.x] = input[id.x]; // Sequential access, cache-friendly!
}
```

### 3. Use Shared Memory Wisely

```wgsl
// Reduction algorithm using shared memory
var<workgroup> shared_data: array<f32, 256>;

@compute @workgroup_size(256)
fn parallel_reduction(
  @builtin(global_invocation_id) global_id: vec3<u32>,
  @builtin(local_invocation_id) local_id: vec3<u32>
) {
  // Load data into shared memory
  shared_data[local_id.x] = input[global_id.x];
  workgroupBarrier();

  // Parallel reduction
  for (var stride = 128u; stride > 0u; stride >>= 1u) {
    if (local_id.x < stride) {
      shared_data[local_id.x] += shared_data[local_id.x + stride];
    }
    workgroupBarrier();
  }

  // Thread 0 writes the result
  if (local_id.x == 0u) {
    atomicAdd(&global_sum, shared_data[0]);
  }
}
```

## Wild Applications of Compute 🎪

### 1. Ray Tracing in a Compute Shader

```wgsl
// Because why not render entire scenes without the graphics pipeline?
struct Ray {
  origin: vec3<f32>,
  direction: vec3<f32>,
}

struct Sphere {
  center: vec3<f32>,
  radius: f32,
  color: vec3<f32>,
}

fn intersectSphere(ray: Ray, sphere: Sphere) -> f32 {
  let oc = ray.origin - sphere.center;
  let b = dot(oc, ray.direction);
  let c = dot(oc, oc) - sphere.radius * sphere.radius;
  let discriminant = b * b - c;

  if (discriminant < 0.0) {
    return -1.0;
  }

  return -b - sqrt(discriminant);
}

@compute @workgroup_size(8, 8)
fn rayTrace(@builtin(global_invocation_id) pixel: vec3<u32>) {
  let uv = vec2<f32>(pixel.xy) / vec2<f32>(dimensions);
  let ray = Ray(
    vec3<f32>(0.0, 0.0, -5.0),
    normalize(vec3<f32>(uv * 2.0 - 1.0, 1.0))
  );

  // Trace through scene...
  var color = vec3<f32>(0.0);
  // ... ray tracing magic happens here ...

  textureStore(outputImage, pixel.xy, vec4<f32>(color, 1.0));
}
```

### 2. Cryptocurrency Mining (Don't Actually Do This)

```wgsl
// For educational purposes only! 😇
@compute @workgroup_size(256)
fn definitely_not_mining(@builtin(global_invocation_id) id: vec3<u32>) {
  // This would be where you'd implement SHA-256
  // But we're responsible developers who respect users' hardware
  // So here's a joke instead:
  output[id.x] = 42.0; // The answer to everything
}
```

## The Zen of Compute 🧘

Compute shaders teach us that:
1. **Parallelism is power** - Think in thousands of threads, not one
2. **Memory is precious** - Cache misses hurt more than breakups
3. **Synchronization is expensive** - Barriers are like traffic lights in your code
4. **The GPU doesn't care about your feelings** - It just wants data to crunch

## Debugging Compute Shaders 🔍

The tricky part about compute shaders? When they go wrong, they go *really* wrong. Here's how to debug:

```javascript
// Debug buffer pattern
async function debugCompute(device, pipeline, data) {
  // Create a debug buffer
  const debugBuffer = device.createBuffer({
    size: 4096, // Space for debug values
    usage: GPUBufferUsage.STORAGE | GPUBufferUsage.COPY_SRC,
  });

  // Add it to your compute shader
  // @group(0) @binding(99) var<storage, read_write> debug: array<f32>;

  // In shader: debug[global_id.x] = some_value_to_inspect;

  // Read it back and analyze
  const staging = device.createBuffer({
    size: debugBuffer.size,
    usage: GPUBufferUsage.MAP_READ | GPUBufferUsage.COPY_DST,
  });

  // ... run compute, copy buffer, read results ...
}
```

---

*Remember: With great compute power comes great electricity bills. Use WebGPU compute responsibly!* ⚡