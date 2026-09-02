# Auto-Resizing Textarea: Death by a Thousand Edge Cases

## What Seems Simple
"Make the textarea grow as the user types."

## The Actual Complexity

### 1. The Clone Wars
```javascript
// The "standard" approach - create invisible clone
function autoResize(textarea) {
  const clone = textarea.cloneNode();
  clone.style.visibility = 'hidden';
  clone.style.position = 'absolute';
  clone.value = textarea.value;
  document.body.appendChild(clone);

  textarea.style.height = clone.scrollHeight + 'px';
  clone.remove();
}

// Problems:
// 1. Clone needs EXACT same styles (all 200+ CSS properties)
// 2. Different box-sizing breaks everything
// 3. Costs a full layout recalculation
// 4. Flickers on slow devices
```

### 2. The ScrollHeight Lies
```javascript
// scrollHeight includes padding... sometimes
const textarea = document.querySelector('textarea');
textarea.style.padding = '20px';
textarea.style.boxSizing = 'border-box';

console.log(textarea.scrollHeight);
// Chrome: includes padding
// Firefox: might not include bottom padding
// Safari: depends on overflow-y value
// IE11: just makes something up
```

### 3. The Placeholder Paradox
```html
<textarea placeholder="Enter your message here..."></textarea>
```

```javascript
// Empty textarea with long placeholder
// Should it size to fit the placeholder?
// What if placeholder is multi-line?
// What if placeholder is longer than maxHeight?

// Each browser has different opinions!
```

### 4. Line Height Calculation Hell
```css
textarea {
  line-height: 1.5;      /* Relative to font-size */
  line-height: 24px;      /* Absolute */
  line-height: 150%;      /* Percentage */
  line-height: normal;    /* Browser decides (different per font!) */
}
```

```javascript
// How many lines of text fit?
const lineHeight = getComputedStyle(textarea).lineHeight;
// Returns: "normal" 🤦

// Now you need to:
// 1. Create a test element
// 2. Measure single line vs two lines
// 3. Calculate difference
// 4. Hope user doesn't have minimum font size set
```

### 5. The Async Font Loading Nightmare
```javascript
// Page loads, textarea auto-sizes with fallback font
document.fonts.ready.then(() => {
  // Custom font loads, all measurements are now wrong!
  // Text that fit in 3 lines now needs 4
  // Textarea is now too short
  // Need to recalculate everything
});

// But wait! Fonts can load multiple times:
// - Initial load
// - Bold variant loads when user types **bold**
// - Italic loads when user types *italic*
// Each changes metrics!
```

### 6. The Zoom Catastrophe
```javascript
// User has browser zoomed to 110%
// All pixel calculations are now fractional
textarea.scrollHeight; // 102.3 (not 102!)
textarea.style.height = '102.3px'; // Browser rounds to 102
// Result: Scrollbar flickers in and out
```

### 7. Mobile Keyboard Viewport Chaos
```javascript
// iOS Safari: Viewport shrinks when keyboard appears
// Chrome Android: Viewport height stays same, content scrolls
// Samsung Internet: Does whatever it wants

visualViewport.addEventListener('resize', () => {
  // Keyboard appeared/disappeared
  // But your textarea is now:
  // - Partially hidden
  // - Scrolled out of view
  // - Different height than calculated
  // - Fighting with viewport resize
});
```

### 8. Copy-Paste Destruction
```javascript
textarea.addEventListener('paste', (e) => {
  // User pastes 10MB of text
  // Auto-resize tries to calculate height
  // Browser freezes calculating scrollHeight
  // OR textarea becomes 50,000px tall
  // OR you cap it but now need scrollbar
  // When do you show "Read more" vs scroll?
});
```

### 9. The White-Space CSS Maze
```css
textarea {
  white-space: pre;        /* Default - preserves spaces and breaks */
  white-space: pre-wrap;   /* Wraps long lines */
  white-space: pre-line;   /* Collapses spaces */
  white-space: nowrap;     /* Horizontal scrolling hell */
}
```

Each changes how height is calculated. Mix with `word-break`, `overflow-wrap`, and `hyphens` for exponential complexity.

### 10. IME Composition Insanity
```javascript
// User typing in Japanese
textarea.addEventListener('compositionstart', () => {
  // "hello" appears underlined while typing
  // This is NOT in textarea.value yet
  // scrollHeight doesn't account for it
  // Height jumps when composition ends
});

// During composition:
// textarea.value = "previous text"
// Visual display = "previous text こんにちは" (composing)
// scrollHeight = height of "previous text" only
```

### 11. The Performance Death Spiral
```javascript
function autoResize() {
  textarea.style.height = 'auto'; // Force recalculation
  textarea.style.height = textarea.scrollHeight + 'px';
}

// This causes:
// 1. Layout (invalidates everything)
// 2. Reflow (recalculates positions)
// 3. Paint (redraws textarea)
// 4. Composite (GPU update)

// On EVERY keystroke!
// Type 100 WPM = 500 characters/min = 8 reflows/second
// Page becomes unusable
```

### 12. Transform and Scale Breakdown
```css
.scaled-container {
  transform: scale(1.5);
}
```

```javascript
// scrollHeight returns unscaled value
// But visual height needs scaled value
// offsetHeight is scaled... sometimes
// getBoundingClientRect() is scaled
// getComputedStyle() is not scaled

// Math required: Matrix multiplication with transform origin
```

## The Real Implementation Attempts

### Attempt 1: The GitHub Approach
```javascript
// GitHub's actual code (simplified)
class AutoResizeTextarea {
  constructor() {
    this.clone = this.createClone();
    this.rafId = null;
    this.resizeObserver = new ResizeObserver(/*...*/);
    this.mutationObserver = new MutationObserver(/*...*/);
    this.fontLoadTracker = new FontFaceObserver(/*...*/);
  }

  // 300+ lines of edge cases
}
```

### Attempt 2: The Facebook Approach
Just use `contenteditable` div and deal with different problems:
- No form submission
- No maxlength
- No validation
- Accessibility nightmare
- Different selection API
- XSS vulnerabilities

### Attempt 3: The "Good Enough" Approach
```javascript
// What most production apps actually do:
textarea.style.height = (textarea.scrollHeight + 2) + 'px';
// The +2 is magic, don't remove it
// It works 95% of the time
// The other 5% gets bug reports
```

## Why This Is Still Unsolved
CSS Working Group has been discussing `field-sizing: content` since 2019. Still not implemented. The problem is that fundamental that even browser vendors can't agree on how it should work.

Every major web app has their own implementation:
- Slack: Custom
- Discord: Custom
- Twitter: Custom
- WhatsApp Web: Custom
- Notion: Entire custom text engine

None are perfect. All have edge cases.