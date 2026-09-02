# Smooth Scrolling: 60fps of Lies

## What Seems Simple
"Just animate the scroll position smoothly from A to B."

## The Actual Complexity

### 1. The Frame Rate Paradox
```javascript
// You want 60fps = 16.67ms per frame
// But JavaScript doesn't run in perfect 16.67ms intervals

requestAnimationFrame(timestamp => {
  // timestamp might be: 16.7, 33.3, 50.1, 66.6...
  // Not uniform! Could be 16.7ms or 16.8ms apart
  // Accumulated error = jerky scrolling
});
```

### 2. Multiple Coordinate Systems Fighting
```javascript
// User initiates smooth scroll while:
// 1. CSS animation is moving an element
// 2. Position: sticky element is transitioning
// 3. IntersectionObserver is firing
// 4. Another smooth scroll is in progress
// 5. User is also manually scrolling with wheel

// Which wins? What's the final position?
```

### 3. The Rubber Band Effect (Mobile)
```javascript
// iOS momentum scrolling with bounce
window.scrollTo({
  top: 1000,
  behavior: 'smooth'
});

// But iOS is already doing elastic bounce at top
// Result: Scroll fights against rubber band physics
// Scroll position can go NEGATIVE (beyond bounds)
// Your animation math explodes
```

### 4. Paint vs Composite vs Layout Thrashing
```javascript
// Innocent looking code:
function smoothScroll(target) {
  const start = window.scrollY;
  const distance = target - start;

  function step(progress) {
    window.scrollTo(0, start + distance * progress);
    // ☠️ This triggers:
    // 1. Layout recalculation (all position:fixed elements)
    // 2. Paint (if any backgrounds are fixed)
    // 3. Composite (GPU needs new layers)
    // = 3ms + 5ms + 2ms = 10ms per frame
    // 60fps budget blown!
  }
}
```

### 5. Scroll Anchoring Chaos
Modern browsers try to prevent "scroll jumping" when content loads above:

```javascript
// User smooth scrolls to section#pricing
// During scroll, an image lazy-loads above
// Browser "helpfully" adjusts scroll position to compensate
// Your animation target is now wrong!
// Scroll ends up 200px off target
```

### 6. Fractional Pixels & Subpixel Rendering
```javascript
// Your easing function calculates:
// scrollTo(0, 1234.56789);

// Browser rounds differently per platform:
// Chrome: 1234.5 (nearest 0.5)
// Safari: 1235 (ceils in some cases)
// Firefox: 1234.567 (keeps more precision)

// Accumulates to visible jitter on high-DPI screens
```

### 7. The Touch vs Wheel vs Keyboard Nightmare
```javascript
// User starts smooth scroll via button click
// IMMEDIATELY uses scroll wheel

// Options:
// 1. Cancel smooth scroll (jarring)
// 2. Ignore user input (frustrating)
// 3. Blend both (math hell)
// 4. Queue them (feels laggy)

// Each browser picks differently!
```

### 8. Transform vs Scroll Performance
```javascript
// Method 1: Actually scrolling
element.scrollTop += delta; // Triggers reflow

// Method 2: Fake it with transform
element.style.transform = `translateY(${-delta}px)`; // GPU accelerated

// But Method 2 breaks:
// - position: fixed elements
// - IntersectionObserver
// - :target CSS selectors
// - Accessibility scroll position
// - Browser find-in-page
// - URL hash navigation
```

### 9. Request Animation Frame Scheduling
```javascript
let scrolling = false;

function smoothScroll() {
  scrolling = true;
  requestAnimationFrame(animate);
}

function animate() {
  // Plot twist: RAF doesn't fire when tab is backgrounded
  // User switches tabs during scroll
  // Comes back: scroll frozen mid-animation
  // Or worse: all frames fire at once = instant teleport

  if (scrolling) requestAnimationFrame(animate);
}
```

### 10. The Easing Function Uncanny Valley
```javascript
// Linear looks robotic
// Ease-in-out looks "floaty"
// Custom cubic-bezier can overshoot
// Spring physics can oscillate forever

// Apple's magic formula (reverse-engineered):
function appleEasing(t) {
  // Involves derivatives and differential equations
  // Different for trackpad vs mouse vs touch
  // Changes based on scroll distance
  // Has "magnetic" snapping near endpoints
  return /* 50 lines of math */;
}
```

## The Real Implementation

```javascript
class SmoothScroller {
  constructor() {
    this.activeScrolls = new Map();
    this.wheelDetector = new WheelInterruptDetector();
    this.touchTracker = new TouchMomentumTracker();
    this.frameScheduler = new RAFScheduler();
    this.physicsEngine = new ScrollPhysics();
    this.anchorCompensator = new ScrollAnchorObserver();
    this.subpixelAccumulator = new Float64Array(2);
    // ... 47 more subsystems
  }

  scrollTo(target) {
    // 500+ lines of edge case handling
  }
}
```

## Why Nobody Gets It Right
Even native `scrollTo({ behavior: 'smooth' })` is inconsistent across browsers. Google's Material Design, Apple's iOS, and Microsoft's Fluent all have different "smooth" implementations.

The problem is so complex that most sites just use a library (smooth-scroll-polyfill, locomotive-scroll, etc.) which itself is thousands of lines of battle-tested edge cases.