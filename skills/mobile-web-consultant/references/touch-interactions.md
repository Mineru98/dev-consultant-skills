# Touch Interactions Guide

Complete reference for implementing touch interactions in mobile web applications.

## Touch Events

### Basic Events

```javascript
element.addEventListener('touchstart', handleTouchStart);
element.addEventListener('touchmove', handleTouchMove);
element.addEventListener('touchend', handleTouchEnd);
element.addEventListener('touchcancel', handleTouchCancel);

function handleTouchStart(e) {
  const touch = e.touches[0];
  console.log('Start:', touch.clientX, touch.clientY);
}

function handleTouchMove(e) {
  const touch = e.touches[0];
  console.log('Move:', touch.clientX, touch.clientY);
}

function handleTouchEnd(e) {
  const touch = e.changedTouches[0];
  console.log('End:', touch.clientX, touch.clientY);
}
```

### Preventing Default Behaviors

```css
/* Disable tap highlight */
* {
  -webkit-tap-highlight-color: transparent;
}

/* Disable text selection */
.no-select {
  user-select: none;
  -webkit-user-select: none;
}

/* Disable pull-to-refresh (custom implementation) */
body {
  overscroll-behavior-y: contain;
}
```

```javascript
// Prevent default for custom gestures
element.addEventListener('touchmove', (e) => {
  e.preventDefault();
}, { passive: false });
```

## Gesture Recognition

### Tap Detection

```javascript
class TapDetector {
  constructor(element, options = {}) {
    this.element = element;
    this.tapThreshold = options.tapThreshold || 10;
    this.tapTimeout = options.tapTimeout || 300;

    this.startX = 0;
    this.startY = 0;
    this.startTime = 0;

    element.addEventListener('touchstart', this.handleStart.bind(this));
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }

  handleStart(e) {
    const touch = e.touches[0];
    this.startX = touch.clientX;
    this.startY = touch.clientY;
    this.startTime = Date.now();
  }

  handleEnd(e) {
    const touch = e.changedTouches[0];
    const dx = touch.clientX - this.startX;
    const dy = touch.clientY - this.startY;
    const distance = Math.sqrt(dx * dx + dy * dy);
    const duration = Date.now() - this.startTime;

    if (distance < this.tapThreshold && duration < this.tapTimeout) {
      this.element.dispatchEvent(new CustomEvent('tap', {
        detail: { x: touch.clientX, y: touch.clientY }
      }));
    }
  }
}

// Usage
const detector = new TapDetector(element);
element.addEventListener('tap', (e) => {
  console.log('Tapped at:', e.detail.x, e.detail.y);
});
```

### Swipe Detection

```javascript
class SwipeDetector {
  constructor(element, options = {}) {
    this.element = element;
    this.threshold = options.threshold || 50;
    this.restraint = options.restraint || 100;
    this.allowedTime = options.allowedTime || 300;

    this.startX = 0;
    this.startY = 0;
    this.startTime = 0;

    element.addEventListener('touchstart', this.handleStart.bind(this));
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }

  handleStart(e) {
    const touch = e.touches[0];
    this.startX = touch.clientX;
    this.startY = touch.clientY;
    this.startTime = Date.now();
  }

  handleEnd(e) {
    const touch = e.changedTouches[0];
    const dx = touch.clientX - this.startX;
    const dy = touch.clientY - this.startY;
    const elapsed = Date.now() - this.startTime;

    if (elapsed <= this.allowedTime) {
      if (Math.abs(dx) >= this.threshold && Math.abs(dy) <= this.restraint) {
        const direction = dx > 0 ? 'right' : 'left';
        this.element.dispatchEvent(new CustomEvent('swipe', {
          detail: { direction, distance: Math.abs(dx) }
        }));
      } else if (Math.abs(dy) >= this.threshold && Math.abs(dx) <= this.restraint) {
        const direction = dy > 0 ? 'down' : 'up';
        this.element.dispatchEvent(new CustomEvent('swipe', {
          detail: { direction, distance: Math.abs(dy) }
        }));
      }
    }
  }
}

// Usage
const swipe = new SwipeDetector(element);
element.addEventListener('swipe', (e) => {
  console.log('Swiped:', e.detail.direction);
});
```

### Long Press Detection

```javascript
class LongPressDetector {
  constructor(element, options = {}) {
    this.element = element;
    this.delay = options.delay || 500;
    this.threshold = options.threshold || 10;

    this.timer = null;
    this.startX = 0;
    this.startY = 0;

    element.addEventListener('touchstart', this.handleStart.bind(this));
    element.addEventListener('touchmove', this.handleMove.bind(this));
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }

  handleStart(e) {
    const touch = e.touches[0];
    this.startX = touch.clientX;
    this.startY = touch.clientY;

    this.timer = setTimeout(() => {
      this.element.dispatchEvent(new CustomEvent('longpress', {
        detail: { x: this.startX, y: this.startY }
      }));
      // Haptic feedback
      navigator.vibrate?.(20);
    }, this.delay);
  }

  handleMove(e) {
    const touch = e.touches[0];
    const dx = touch.clientX - this.startX;
    const dy = touch.clientY - this.startY;

    if (Math.abs(dx) > this.threshold || Math.abs(dy) > this.threshold) {
      this.cancel();
    }
  }

  handleEnd() {
    this.cancel();
  }

  cancel() {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }
  }
}
```

### Pull-to-Refresh

```javascript
class PullToRefresh {
  constructor(element, options = {}) {
    this.element = element;
    this.threshold = options.threshold || 80;
    this.onRefresh = options.onRefresh || (() => {});

    this.startY = 0;
    this.pulling = false;

    element.addEventListener('touchstart', this.handleStart.bind(this));
    element.addEventListener('touchmove', this.handleMove.bind(this), { passive: false });
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }

  handleStart(e) {
    if (window.scrollY === 0) {
      this.startY = e.touches[0].clientY;
      this.pulling = true;
    }
  }

  handleMove(e) {
    if (!this.pulling) return;

    const y = e.touches[0].clientY;
    const distance = y - this.startY;

    if (distance > 0) {
      e.preventDefault();
      this.updateUI(distance);
    }
  }

  handleEnd(e) {
    if (!this.pulling) return;

    const y = e.changedTouches[0].clientY;
    const distance = y - this.startY;

    if (distance >= this.threshold) {
      this.refresh();
    }

    this.reset();
    this.pulling = false;
  }

  updateUI(distance) {
    const progress = Math.min(distance / this.threshold, 1);
    // Update spinner rotation or progress indicator
  }

  refresh() {
    this.onRefresh();
  }

  reset() {
    // Reset UI
  }
}
```

## Swipeable List Item

```javascript
class SwipeableItem {
  constructor(element, options = {}) {
    this.element = element;
    this.threshold = options.threshold || 100;
    this.onSwipeLeft = options.onSwipeLeft;
    this.onSwipeRight = options.onSwipeRight;

    this.startX = 0;
    this.currentX = 0;
    this.swiping = false;

    element.addEventListener('touchstart', this.handleStart.bind(this));
    element.addEventListener('touchmove', this.handleMove.bind(this), { passive: false });
    element.addEventListener('touchend', this.handleEnd.bind(this));
  }

  handleStart(e) {
    this.startX = e.touches[0].clientX;
    this.swiping = true;
    this.element.style.transition = 'none';
  }

  handleMove(e) {
    if (!this.swiping) return;

    this.currentX = e.touches[0].clientX - this.startX;

    // Limit swipe distance
    const maxSwipe = 120;
    this.currentX = Math.max(-maxSwipe, Math.min(maxSwipe, this.currentX));

    this.element.style.transform = `translateX(${this.currentX}px)`;
  }

  handleEnd() {
    if (!this.swiping) return;
    this.swiping = false;

    this.element.style.transition = 'transform 0.2s ease-out';

    if (Math.abs(this.currentX) >= this.threshold) {
      if (this.currentX > 0 && this.onSwipeRight) {
        this.onSwipeRight();
      } else if (this.currentX < 0 && this.onSwipeLeft) {
        this.onSwipeLeft();
      }
    }

    // Reset position
    this.element.style.transform = 'translateX(0)';
    this.currentX = 0;
  }
}
```

## Haptic Feedback

```javascript
const Haptics = {
  // Light tap
  light: () => navigator.vibrate?.(10),

  // Medium feedback
  medium: () => navigator.vibrate?.(20),

  // Heavy feedback
  heavy: () => navigator.vibrate?.([30, 50, 30]),

  // Success
  success: () => navigator.vibrate?.([10, 50, 10]),

  // Error
  error: () => navigator.vibrate?.([50, 100, 50]),

  // Selection tick
  tick: () => navigator.vibrate?.(5),
};

// Usage
button.addEventListener('click', () => {
  Haptics.light();
});
```

## CSS Touch States

```css
/* Touch feedback */
.touchable {
  -webkit-tap-highlight-color: transparent;
  touch-action: manipulation;
  cursor: pointer;
}

.touchable:active {
  transform: scale(0.97);
  opacity: 0.8;
}

/* Ripple effect */
.ripple {
  position: relative;
  overflow: hidden;
}

.ripple::after {
  content: '';
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  transform: scale(0);
  opacity: 0;
}

.ripple:active::after {
  animation: ripple-effect 0.4s ease-out;
}

@keyframes ripple-effect {
  0% {
    transform: scale(0);
    opacity: 1;
  }
  100% {
    transform: scale(2.5);
    opacity: 0;
  }
}
```

## Touch Target Guidelines

| Element | Minimum Size | Recommended |
|---------|-------------|-------------|
| Button | 44x44px | 48x48px |
| Link | 44x44px | 48x48px |
| Icon button | 44x44px | 48x48px |
| List item | 48px height | 56px height |
| Input field | 44px height | 48px height |

```css
/* Ensure minimum touch target */
.touch-target {
  min-width: 44px;
  min-height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```
