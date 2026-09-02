# Mobile Gestures: The Touch Event Hellscape

## What Seems Simple
"Just detect when the user swipes, pinches, or taps."

## The Actual Complexity

### 1. The Touch Point Identity Crisis
```javascript
// User puts two fingers down
touchstart: touches.length = 2
// touches[0].identifier = 24601
// touches[1].identifier = 24602

// User lifts first finger, puts it back down
touchend: touches.length = 1
touchstart: touches.length = 2
// touches[0].identifier = 24602 (was touches[1]!)
// touches[1].identifier = 24603 (new!)

// Your pinch gesture tracker just exploded
// Which finger is which? Arrays reshuffled!
```

### 2. The 300ms Click Delay Disaster
```javascript
// Mobile Safari's original sin:
element.addEventListener('touchend', (e) => {
  // User tapped!
  // Browser: "But what if they're about to double-tap to zoom?"
  // Waits 300ms...
  // Then fires 'click'
});

// Solutions that all break something:
// 1. touch-action: manipulation (breaks pinch zoom)
// 2. FastClick.js (breaks form inputs)
// 3. pointer-events (breaks iOS 12)
// 4. viewport width=device-width (breaks desktop mode)
```

### 3. The Scroll vs Swipe Paradox
```javascript
let startX, startY;
let isScrolling = null;

touchstart = (e) => {
  startX = e.touches[0].clientX;
  startY = e.touches[0].clientY;
};

touchmove = (e) => {
  if (isScrolling === null) {
    const deltaX = Math.abs(e.touches[0].clientX - startX);
    const deltaY = Math.abs(e.touches[0].clientY - startY);

    // Is user trying to scroll or swipe?
    isScrolling = deltaY > deltaX;

    if (!isScrolling) {
      e.preventDefault(); // Stop scroll
      // But now you've broken momentum scrolling!
      // And pull-to-refresh!
      // And address bar hiding!
    }
  }
  // Too late! Browser already started scrolling
  // Visual jank as gesture fights scroll
};
```

### 4. The Gesture Velocity Calculation
```javascript
// Calculate swipe velocity for momentum
const samples = [];

touchmove = (e) => {
  samples.push({
    x: e.touches[0].clientX,
    y: e.touches[0].clientY,
    time: Date.now()
  });

  if (samples.length > 10) samples.shift();
};

touchend = () => {
  // Calculate velocity from samples
  // But wait:
  // - Finger decelerated before lifting (natural motion)
  // - Last few samples show near-zero velocity
  // - But user expects momentum!

  // Take samples from 100ms ago?
  // But that's outdated if user changed direction
  // Weighted average? Now you need calculus
};
```

### 5. The Pinch-Zoom Center Point Math
```javascript
// Two fingers pinching
function getPinchCenter(touches) {
  const x = (touches[0].clientX + touches[1].clientX) / 2;
  const y = (touches[0].clientY + touches[1].clientY) / 2;
  return { x, y };
  // Simple, right?
}

// But the center point MOVES during pinch!
// User pinches with unequal finger movement
// Center drifts
// Image zooms toward wrong point
// User frustrated

// Need to:
// 1. Track initial center
// 2. Track current center
// 3. Apply transform-origin adjustments
// 4. Compensate with translate()
// 5. Account for existing transforms
// Matrix math required
```

### 6. The Android vs iOS Chaos
```javascript
// iOS: touchstart → touchmove → touchend → click
// Android: touchstart → touchmove → touchend → (no click if moved)

// iOS: preventDefault() in touchstart stops scrolling
// Android: Must be in first touchmove

// iOS: touches[0].force for 3D Touch
// Android: touches[0].pressure but different scale

// iOS: gesturestart/gesturechange for pinch
// Android: Doesn't exist, calculate manually

// iOS: Triple-tap selects paragraph
// Android: Triple-tap does nothing
// Your gesture: Conflicts differently per OS
```

### 7. The Passive Listener Performance Trap
```javascript
// Old way - blocks scrolling
element.addEventListener('touchstart', (e) => {
  e.preventDefault(); // Blocks browser scroll
}, false);

// New way - doesn't block
element.addEventListener('touchstart', handler, { passive: true });
// Can't call preventDefault()!

// But you need to conditionally prevent:
element.addEventListener('touchstart', (e) => {
  if (isSwipeGesture(e)) {
    e.preventDefault(); // ERROR! Passive listener can't prevent
  }
}, { passive: false }); // Now you're slow again
```

### 8. The Rotation Gesture Insanity
```javascript
// Calculate rotation between two fingers
function getRotation(touches) {
  const dx = touches[1].clientX - touches[0].clientX;
  const dy = touches[1].clientY - touches[0].clientY;
  return Math.atan2(dy, dx) * 180 / Math.PI;
}

// But wait:
// - Fingers crossed? Angle flips 180°
// - One finger stationary? Rotation center wrong
// - Combined with pinch? Math explodes
// - Crosses 360°? Suddenly -359°

// Need to track:
// - Previous angle
// - Accumulated rotation
// - Handle angle wrapping
// - Detect deliberate vs accidental rotation
```

### 9. The Multi-Touch Gesture Conflict
```javascript
// User puts down two fingers
// Are they:
// 1. Starting a pinch?
// 2. Starting a two-finger scroll?
// 3. Starting a rotation?
// 4. About to put down a third finger?
// 5. Accidentally touching while reaching?

// You must decide NOW
// Wrong guess = broken UX

// Common solution: Wait 100ms
// Result: All gestures feel laggy
```

### 10. The Edge Swipe System Gesture
```javascript
// iOS: Swipe from left edge = Back navigation
// Android: Swipe from left edge = Drawer menu
// Your carousel: Swipe from left edge = Previous slide

// All three trigger!

// Try to prevent system gesture:
e.preventDefault(); // Doesn't work
e.stopPropagation(); // Doesn't work
touch-action: none; // Breaks everything else

// Real solution: Don't use edge gestures
// UX solution: Your app feels non-native
```

### 11. The Touch Target Size Reality
```javascript
// Finger touch area: ~8-10mm
// In pixels at different densities:
// - iPhone 6: ~44px
// - iPhone 14 Pro: ~61px
// - Cheap Android: ~30px
// - Tablet: ~40px

element.addEventListener('touchstart', (e) => {
  // e.touches[0].radiusX ~= 20px
  // But click point is center
  // User thinks they tapped button A
  // Actually hit button B's edge
});

// Solution: Invisible expanded hit areas
// Problem: Now they overlap
// Bigger problem: Which wins?
```

### 12. The Momentum Physics Simulation
```javascript
class MomentumScroller {
  handleRelease(velocity) {
    // iOS momentum: Cubic-bezier curve
    // Android: Different curve
    // Custom: Must feel "natural"

    const deceleration = 0.997; // Magic number
    const threshold = 0.5; // When to stop

    function animate() {
      velocity *= deceleration;
      position += velocity;

      // But also need:
      // - Rubber band at edges
      // - Snap points
      // - Bounce back
      // - Match native scroll exactly
      // - 60fps or feels janky

      if (Math.abs(velocity) > threshold) {
        requestAnimationFrame(animate);
      }
    }
  }
}
```

### 13. The Gesture Recognition State Machine
```javascript
class GestureRecognizer {
  constructor() {
    this.states = {
      IDLE: 'idle',
      POSSIBLE_TAP: 'possible_tap',
      POSSIBLE_SWIPE: 'possible_swipe',
      POSSIBLE_PINCH: 'possible_pinch',
      SCROLLING: 'scrolling',
      SWIPING: 'swiping',
      PINCHING: 'pinching',
      FAILED: 'failed'
    };
  }

  // State transitions based on:
  // - Time (tap vs long press)
  // - Distance (tap vs swipe)
  // - Direction (scroll vs swipe)
  // - Finger count (changes everything)
  // - Velocity (swipe vs drag)

  // 50+ state transitions
  // Each can fail and need recovery
}
```

### 14. The Palm Rejection Nightmare
```javascript
// User holding phone with palm touching screen edge
touchstart: 5 touches detected!
// touches[0-3]: Palm base
// touches[4]: Actual finger

// How to detect palm vs finger?
// - Touch area (but varies by hand size)
// - Touch pressure (but varies by device)
// - Touch shape (but not all devices report)
// - Location (but depends on grip)

// Most apps: Just break with accidental touches
```

### 15. The Accessibility Gesture Override
```javascript
// User has accessibility enabled:
// - Three-finger tap: Speak
// - Three-finger swipe: Navigate
// - Two-finger rotate: Rotor
// - Split tap: Select

// Your gestures:
element.addEventListener('touchstart', (e) => {
  if (e.touches.length === 3) {
    // Your custom three-finger gesture
    // CONFLICTS with accessibility!
    // User can't use your app
  }
});

// No way to detect if accessibility is on
// No way to override safely
// Must redesign entire gesture system
```

## The Real Implementation

```javascript
class ProductionGestureHandler {
  constructor() {
    this.recognizers = [];
    this.activeGestures = new Map();
    this.touchHistory = new CircularBuffer(100);
    this.vendorWorkarounds = new Map([
      ['iOS-Safari', this.fixIOSSafari],
      ['Android-Chrome', this.fixAndroidChrome],
      ['Samsung-Internet', this.fixSamsungBrowser],
      // 20 more...
    ]);
  }

  handleTouch(event) {
    // 500+ lines of edge cases
    // Still breaks on new devices
    // Still conflicts with native gestures
    // Still feels slightly off
  }
}
```

## Why Every App Feels Different

- **Instagram**: Custom everything, conflicts with system gestures
- **Google Maps**: Two-finger rotate works 70% of the time
- **Twitter**: Pull-to-refresh triggers while scrolling up
- **TikTok**: Swipe sometimes scrolls, sometimes navigates
- **Banking apps**: Disable all gestures for "security"

The web wasn't designed for touch. We're retrofitting mouse events onto finger physics. Every gesture library (Hammer.js, ZingTouch, etc.) is thousands of lines of workarounds for problems that shouldn't exist.