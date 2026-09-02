# WebGPU Shaders & Pipelines: The Art of GPU Programming 🎨

Welcome to the magical world of shaders and pipelines! If WebGPU is a symphony orchestra, then shaders are the musicians and pipelines are the conductor. Let's dive into this beautiful chaos!

## WGSL: The Language of the GPU 🗣️

WGSL (WebGPU Shading Language) is how we talk to the GPU. It's like learning a new language, except this one lets you manipulate millions of pixels per second. Pretty cool party trick!

### The Anatomy of a Shader 🔬

Think of shaders as tiny programs that run in parallel thousands (or millions!) of times. Each one is like a worker bee in a massive hive, all working together to create your final image.

```wgsl
// A complete WGSL shader with all the bells and whistles
struct VertexInput {
  @location(0) position: vec3<f32>,
  @location(1) normal: vec3<f32>,
  @location(2) uv: vec2<f32>,
}

struct VertexOutput {
  @builtin(position) clip_position: vec4<f32>,
  @location(0) world_position: vec3<f32>,
  @location(1) normal: vec3<f32>,
  @location(2) uv: vec2<f32>,
}

struct Uniforms {
  model: mat4x4<f32>,
  view: mat4x4<f32>,
  projection: mat4x4<f32>,
  time: f32,
}

@binding(0) @group(0) var<uniform> uniforms: Uniforms;
@binding(1) @group(0) var mySampler: sampler;
@binding(2) @group(0) var myTexture: texture_2d<f32>;

@vertex
fn vs_main(input: VertexInput) -> VertexOutput {
  var output: VertexOutput;

  // Apply transformations
  let worldPos = uniforms.model * vec4<f32>(input.position, 1.0);
  output.world_position = worldPos.xyz;
  output.clip_position = uniforms.projection * uniforms.view * worldPos;

  // Transform normal to world space
  output.normal = normalize((uniforms.model * vec4<f32>(input.normal, 0.0)).xyz);

  // Pass through UV coordinates
  output.uv = input.uv;

  return output;
}

@fragment
fn fs_main(input: VertexOutput) -> @location(0) vec4<f32> {
  // Sample the texture
  let textureColor = textureSample(myTexture, mySampler, input.uv);

  // Simple directional lighting
  let lightDir = normalize(vec3<f32>(0.5, 1.0, 0.3));
  let diffuse = max(dot(input.normal, lightDir), 0.0);

  // Add some time-based color shifting for fun
  let timeShift = sin(uniforms.time * 2.0) * 0.5 + 0.5;
  let finalColor = textureColor.rgb * diffuse * vec3<f32>(1.0, timeShift, 1.0 - timeShift);

  return vec4<f32>(finalColor, textureColor.a);
}
```

## The Pipeline: Your GPU Assembly Line 🏭

A pipeline is like a recipe for the GPU. It tells it exactly how to process your data from start to finish. Here's how to build one that would make Gordon Ramsay proud:

```javascript
// Creating a sophisticated render pipeline
async function createAdvancedPipeline(device, format) {
  // Step 1: Create the shader module
  const shaderModule = device.createShaderModule({
    label: "Advanced shaders",
    code: await fetch('/shaders/advanced.wgsl').then(r => r.text()),
  });

  // Step 2: Define the pipeline layout
  const pipelineLayout = device.createPipelineLayout({
    label: "Advanced Pipeline Layout",
    bindGroupLayouts: [
      // Layout for uniforms and textures
      device.createBindGroupLayout({
        entries: [
          {
            binding: 0,
            visibility: GPUShaderStage.VERTEX | GPUShaderStage.FRAGMENT,
            buffer: { type: 'uniform' }
          },
          {
            binding: 1,
            visibility: GPUShaderStage.FRAGMENT,
            sampler: { type: 'filtering' }
          },
          {
            binding: 2,
            visibility: GPUShaderStage.FRAGMENT,
            texture: { sampleType: 'float' }
          }
        ]
      })
    ]
  });

  // Step 3: Create the pipeline with all the fixings
  const pipeline = device.createRenderPipeline({
    label: "Advanced Render Pipeline",
    layout: pipelineLayout,

    // Vertex stage configuration
    vertex: {
      module: shaderModule,
      entryPoint: "vs_main",
      buffers: [
        {
          arrayStride: 32, // 3 floats position + 3 floats normal + 2 floats UV
          stepMode: 'vertex',
          attributes: [
            { shaderLocation: 0, offset: 0, format: 'float32x3' },  // position
            { shaderLocation: 1, offset: 12, format: 'float32x3' }, // normal
            { shaderLocation: 2, offset: 24, format: 'float32x2' }, // uv
          ]
        }
      ]
    },

    // Fragment stage configuration
    fragment: {
      module: shaderModule,
      entryPoint: "fs_main",
      targets: [{
        format: format,
        blend: {
          color: {
            srcFactor: 'src-alpha',
            dstFactor: 'one-minus-src-alpha',
            operation: 'add',
          },
          alpha: {
            srcFactor: 'one',
            dstFactor: 'one-minus-src-alpha',
            operation: 'add',
          }
        }
      }]
    },

    // Primitive configuration
    primitive: {
      topology: 'triangle-list',
      cullMode: 'back',
      frontFace: 'ccw'
    },

    // Depth/stencil configuration
    depthStencil: {
      depthWriteEnabled: true,
      depthCompare: 'less',
      format: 'depth24plus',
    }
  });

  return pipeline;
}
```

## Advanced Shader Techniques 🧙‍♂️

### 1. Instanced Rendering: One Mesh, Many Copies

Want to render a forest of 10,000 trees without your GPU breaking a sweat? Instanced rendering is your friend!

```wgsl
struct InstanceData {
  @location(5) transform: mat4x4<f32>,
  @location(9) color: vec4<f32>,
}

@vertex
fn vs_instanced(
  vertex: VertexInput,
  instance: InstanceData,
  @builtin(instance_index) instanceIdx: u32
) -> VertexOutput {
  var output: VertexOutput;

  // Use the instance transform
  let worldPos = instance.transform * vec4<f32>(vertex.position, 1.0);
  output.clip_position = uniforms.projection * uniforms.view * worldPos;

  // Vary color based on instance
  output.color = instance.color * vec4<f32>(1.0, 1.0, 1.0, 1.0);

  // Add some variation based on instance index
  let variation = f32(instanceIdx) * 0.1;
  output.world_position = worldPos.xyz + vec3<f32>(sin(variation), 0.0, cos(variation));

  return output;
}
```

### 2. Post-Processing Effects: Making Things Pretty

Post-processing is like Instagram filters for your 3D scenes:

```wgsl
// A fancy bloom effect shader
@fragment
fn fs_bloom(
  @location(0) uv: vec2<f32>
) -> @location(0) vec4<f32> {
  let original = textureSample(sceneTexture, sceneSampler, uv);

  // Extract bright areas
  let brightness = dot(original.rgb, vec3<f32>(0.2126, 0.7152, 0.0722));
  var bloom = vec3<f32>(0.0);

  if (brightness > 0.7) {
    bloom = original.rgb;

    // Blur the bright areas
    for (var i = -4; i <= 4; i++) {
      for (var j = -4; j <= 4; j++) {
        let offset = vec2<f32>(f32(i), f32(j)) * 0.003;
        bloom += textureSample(sceneTexture, sceneSampler, uv + offset).rgb * 0.05;
      }
    }
  }

  // Combine original with bloom
  return vec4<f32>(original.rgb + bloom * 0.3, original.a);
}
```

### 3. Procedural Generation: Creating Worlds from Math

Who needs texture artists when you have math? 🤓

```wgsl
// Procedural noise function (simplified Perlin noise)
fn noise(p: vec2<f32>) -> f32 {
  let i = floor(p);
  let f = fract(p);

  // Smooth interpolation
  let u = f * f * (3.0 - 2.0 * f);

  // Random gradients
  let a = random(i);
  let b = random(i + vec2<f32>(1.0, 0.0));
  let c = random(i + vec2<f32>(0.0, 1.0));
  let d = random(i + vec2<f32>(1.0, 1.0));

  return mix(mix(a, b, u.x), mix(c, d, u.x), u.y);
}

// Fractal Brownian Motion for terrain
fn fbm(p: vec2<f32>) -> f32 {
  var value = 0.0;
  var amplitude = 0.5;
  var frequency = 1.0;

  for (var i = 0; i < 6; i++) {
    value += amplitude * noise(p * frequency);
    frequency *= 2.0;
    amplitude *= 0.5;
  }

  return value;
}

@fragment
fn fs_terrain(@location(0) worldPos: vec3<f32>) -> @location(0) vec4<f32> {
  let height = fbm(worldPos.xz * 0.05);

  // Color based on height
  var color = vec3<f32>(0.0);
  if (height < 0.3) {
    color = vec3<f32>(0.2, 0.3, 0.8); // Water
  } else if (height < 0.5) {
    color = vec3<f32>(0.8, 0.7, 0.5); // Sand
  } else if (height < 0.7) {
    color = vec3<f32>(0.2, 0.6, 0.2); // Grass
  } else {
    color = vec3<f32>(0.9, 0.9, 0.95); // Snow
  }

  return vec4<f32>(color, 1.0);
}
```

## Pipeline Optimization Tips 🚀

### 1. Batch Your Draw Calls

```javascript
// Bad: Many individual draw calls
for (let object of objects) {
  pass.setPipeline(object.pipeline);
  pass.setBindGroup(0, object.bindGroup);
  pass.draw(object.vertexCount);
}

// Good: Batched rendering
const batchedObjects = batchByPipeline(objects);
for (let batch of batchedObjects) {
  pass.setPipeline(batch.pipeline);
  for (let object of batch.objects) {
    pass.setBindGroup(0, object.bindGroup);
    pass.draw(object.vertexCount);
  }
}
```

### 2. Use Indexed Drawing

```javascript
// Create an index buffer for a cube
const indices = new Uint16Array([
  0, 1, 2,  2, 3, 0,  // Front
  4, 5, 6,  6, 7, 4,  // Back
  // ... more faces
]);

const indexBuffer = device.createBuffer({
  size: indices.byteLength,
  usage: GPUBufferUsage.INDEX | GPUBufferUsage.COPY_DST,
});

device.queue.writeBuffer(indexBuffer, 0, indices);

// Draw with indices
pass.setIndexBuffer(indexBuffer, 'uint16');
pass.drawIndexed(indices.length);
```

### 3. Pipeline Caching Strategy

```javascript
class PipelineCache {
  constructor(device) {
    this.device = device;
    this.cache = new Map();
  }

  getPipeline(key, createFn) {
    if (!this.cache.has(key)) {
      this.cache.set(key, createFn(this.device));
    }
    return this.cache.get(key);
  }

  // Use it like this:
  warmup(configurations) {
    for (let config of configurations) {
      this.getPipeline(config.key, () => createPipeline(config));
    }
  }
}
```

## Fun Pipeline Patterns 🎭

### The State Machine Pipeline

```javascript
class RenderStateMachine {
  constructor(device) {
    this.states = {
      opaque: this.createOpaquePipeline(device),
      transparent: this.createTransparentPipeline(device),
      wireframe: this.createWireframePipeline(device),
      shadow: this.createShadowPipeline(device),
    };
    this.currentState = null;
  }

  setState(pass, state) {
    if (this.currentState !== state) {
      pass.setPipeline(this.states[state]);
      this.currentState = state;
    }
  }
}
```

### The Multi-Pass Renderer

```javascript
async function multiPassRender(device, scene) {
  const encoder = device.createCommandEncoder();

  // Pass 1: Shadow mapping
  const shadowPass = encoder.beginRenderPass({
    colorAttachments: [],
    depthStencilAttachment: {
      view: shadowDepthTexture.createView(),
      depthClearValue: 1.0,
      depthLoadOp: 'clear',
      depthStoreOp: 'store',
    }
  });
  // Render from light's perspective
  shadowPass.end();

  // Pass 2: Main render
  const mainPass = encoder.beginRenderPass({
    colorAttachments: [{
      view: context.getCurrentTexture().createView(),
      loadOp: 'clear',
      clearValue: { r: 0.1, g: 0.1, b: 0.1, a: 1 },
      storeOp: 'store',
    }]
  });
  // Render with shadows
  mainPass.end();

  // Pass 3: Post-processing
  const postPass = encoder.beginRenderPass({
    colorAttachments: [{
      view: finalTexture.createView(),
      loadOp: 'clear',
      storeOp: 'store',
    }]
  });
  // Apply effects
  postPass.end();

  device.queue.submit([encoder.finish()]);
}
```

## Secret Shader Tricks 🪄

1. **The GPU Timer Trick**: Use `uniforms.time` to create animations without JavaScript
2. **The Discard Pattern**: Use `discard` in fragment shaders for cheap transparency
3. **The Vertex Pulling Pattern**: Store vertex data in textures for massive meshes
4. **The Compute Pre-Pass**: Use compute shaders to cull before rendering
5. **The Bindless Texture Array**: Pack many textures into arrays for fewer bind calls

## Philosophical Musings on Shaders 🤔

Shaders are poetry for computers. Each line is executed millions of times, yet must be perfect. They're deterministic yet create beauty. They're limited yet limitless.

Writing shaders teaches you to think in parallel, to see the world not as objects but as streams of data flowing through silicon rivers. It's meditation for programmers – pure logic creating pure beauty.

---

*"In the beginning was the Vertex, and the Vertex was with GPU, and the Vertex was GPU." - Ancient shader wisdom* ✨