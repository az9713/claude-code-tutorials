---
name: accessibility
description: Web accessibility (a11y) best practices and WCAG compliance
---

# Web Accessibility (a11y)

## Core Principles (POUR)

1. **Perceivable** - Users can perceive the content
2. **Operable** - Users can operate the interface
3. **Understandable** - Users can understand the content
4. **Robust** - Content works with assistive technologies

## Semantic HTML

```html
<!-- ❌ Div soup -->
<div class="header">
  <div class="nav">
    <div class="nav-item">Home</div>
  </div>
</div>

<!-- ✅ Semantic HTML -->
<header>
  <nav aria-label="Main navigation">
    <ul>
      <li><a href="/">Home</a></li>
    </ul>
  </nav>
</header>
```

### Landmark Regions

```html
<header>      <!-- Banner -->
<nav>         <!-- Navigation -->
<main>        <!-- Main content (one per page) -->
<aside>       <!-- Complementary -->
<footer>      <!-- Content info -->
<section>     <!-- Region (with aria-label) -->
<form>        <!-- Form -->
```

## Keyboard Navigation

### Focus Management

```tsx
// Trap focus in modals
function Modal({ isOpen, onClose, children }: ModalProps) {
  const modalRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!isOpen) return;

    const focusableElements = modalRef.current?.querySelectorAll(
      'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
    );

    const firstElement = focusableElements?.[0] as HTMLElement;
    const lastElement = focusableElements?.[focusableElements.length - 1] as HTMLElement;

    firstElement?.focus();

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key === 'Escape') onClose();
      if (e.key === 'Tab') {
        if (e.shiftKey && document.activeElement === firstElement) {
          e.preventDefault();
          lastElement?.focus();
        } else if (!e.shiftKey && document.activeElement === lastElement) {
          e.preventDefault();
          firstElement?.focus();
        }
      }
    };

    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen, onClose]);

  return isOpen ? <div ref={modalRef} role="dialog" aria-modal="true">{children}</div> : null;
}
```

### Skip Links

```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <header>...</header>
  <main id="main-content" tabindex="-1">...</main>
</body>

<style>
  .skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    z-index: 100;
  }
  .skip-link:focus {
    top: 0;
  }
</style>
```

## ARIA Patterns

### Buttons

```tsx
// ❌ Div as button
<div onClick={handleClick}>Submit</div>

// ✅ Proper button
<button onClick={handleClick}>Submit</button>

// ✅ If must use div, add ARIA
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
  Submit
</div>
```

### Forms

```tsx
<form>
  {/* Label association */}
  <label htmlFor="email">Email</label>
  <input
    id="email"
    type="email"
    aria-describedby="email-hint email-error"
    aria-invalid={!!error}
    aria-required="true"
  />
  <span id="email-hint">We'll never share your email</span>
  {error && <span id="email-error" role="alert">{error}</span>}

  {/* Fieldset for groups */}
  <fieldset>
    <legend>Notification preferences</legend>
    <label>
      <input type="checkbox" name="notifications" value="email" />
      Email notifications
    </label>
  </fieldset>
</form>
```

### Live Regions

```tsx
// Announce dynamic content to screen readers
<div aria-live="polite" aria-atomic="true">
  {status && <p>{status}</p>}
</div>

// For critical alerts
<div role="alert">
  Error: Please fix the form errors
</div>

// For status updates
<div role="status">
  Loading... 50% complete
</div>
```

## Images & Media

```html
<!-- Informative image -->
<img src="chart.png" alt="Sales increased 25% in Q4 2024" />

<!-- Decorative image -->
<img src="decoration.png" alt="" role="presentation" />

<!-- Complex image -->
<figure>
  <img src="complex-chart.png" alt="Quarterly sales chart" aria-describedby="chart-desc" />
  <figcaption id="chart-desc">
    Detailed description of the chart data...
  </figcaption>
</figure>

<!-- Video -->
<video controls>
  <source src="video.mp4" type="video/mp4" />
  <track kind="captions" src="captions.vtt" srclang="en" label="English" />
</video>
```

## Color & Contrast

```css
/* Minimum contrast ratios (WCAG AA) */
/* Normal text: 4.5:1 */
/* Large text (18pt+ or 14pt bold): 3:1 */
/* UI components: 3:1 */

/* Don't rely on color alone */
.error {
  color: #d32f2f;
  border-left: 4px solid #d32f2f; /* Visual indicator */
}

.error::before {
  content: "⚠ "; /* Icon indicator */
}
```

## Testing Checklist

- [ ] Navigate entire page with keyboard only
- [ ] Test with screen reader (NVDA, VoiceOver)
- [ ] Check color contrast (use DevTools or axe)
- [ ] Verify focus indicators are visible
- [ ] Test at 200% zoom
- [ ] Validate HTML (no duplicate IDs)
- [ ] Run automated tests (axe-core, Lighthouse)

## Automated Testing

```tsx
// Jest + Testing Library
import { render, screen } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';

expect.extend(toHaveNoViolations);

test('form is accessible', async () => {
  const { container } = render(<LoginForm />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```
