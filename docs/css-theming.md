# LibreChat CSS Theming Guide

*The v1-v10 journey: Everything we learned the hard way*

---

## The Critical Rule

> **Light mode took 1 version. Dark mode took 10. The difference? We tried to be clever with dark mode instead of inspecting the DOM.**

### ❌ NEVER DO THIS

```css
.dark main * {
  background-color: transparent !important;
}
```

Using broad wildcards with `*` and background changes **WILL BREAK YOUR LAYOUT**. This was the v5 disaster - broke scrolling, input fields, everything.

### ✅ DO THIS INSTEAD

```css
.dark .bg-presentation {
  background: #0F2537 !important;
}
```

Target ACTUAL semantic classes that you find by inspecting the DOM.

---

## The v10 Debugging Protocol

When CSS isn't working as expected:

1. **Deploy CSS to GitHub** - Push your changes
2. **Railway auto-rebuilds** (~2-3 min wait)
3. **Hard refresh browser** (Cmd+Shift+R / Ctrl+Shift+R)
4. **Test BOTH light and dark modes** - They behave differently!
5. **If element won't style:**
   - Right-click → Inspect
   - Get the EXACT class names
   - Write precision CSS for actual classes (not assumptions)

---

## How LibreChat Dark Mode Works

- Dark mode is triggered by the `.dark` class on the `<html>` element
- LibreChat uses Tailwind CSS with semantic class names
- Key semantic classes discovered through DOM inspection:
  - `bg-presentation` - Main presentation areas
  - `bg-surface-tertiary` - Tertiary surfaces
  - `bg-surface-secondary` - Secondary surfaces

---

## Version History (Learn from Our Mistakes)

| Version | What Happened |
|---------|---------------|
| v1-v4 | Learning fundamentals about Tailwind CSS |
| v5 | **DISASTER** - Wildcards broke everything |
| v6-v9 | Incremental fixes, still guessing at classes |
| v10 | **VICTORY** - DOM inspection revealed semantic classes |

---

## Complete v10 CSS Template

Copy-paste ready for LibreChat branding:

```css
/* ============================================
   LIBRECHAT CUSTOM BRANDING - v10
   Tested with LibreChat 0.7.x
   ============================================ */

/* === ROOT VARIABLES === */
:root {
  --brand-primary: #C4A052;
  --brand-dark-bg: #0F2537;
  --brand-darker-bg: #0a1a28;
  --brand-input-bg: #1a3347;
  --brand-text: #f5f5f5;
}

/* === LIGHT MODE === */

/* Header/Navigation */
header, nav {
  background: var(--brand-primary) !important;
}

/* Sidebar */
.bg-surface-secondary {
  background: #f8f8f8 !important;
}

/* Main content area */
.bg-presentation {
  background: #ffffff !important;
}

/* === DARK MODE === */

/* Main background */
.dark .bg-presentation {
  background: var(--brand-dark-bg) !important;
}

/* Sidebar */
.dark .bg-surface-secondary {
  background: var(--brand-darker-bg) !important;
}

/* Tertiary surfaces */
.dark .bg-surface-tertiary {
  background: var(--brand-input-bg) !important;
}

/* Input fields */
.dark textarea,
.dark input[type="text"] {
  background: var(--brand-input-bg) !important;
  color: var(--brand-text) !important;
  border-color: var(--brand-primary) !important;
}

/* Buttons - Primary */
.dark .btn-primary,
.dark button[data-variant="primary"] {
  background: var(--brand-primary) !important;
  color: #000 !important;
}

/* Code blocks */
.dark pre,
.dark code {
  background: var(--brand-darker-bg) !important;
}

/* Links */
.dark a {
  color: var(--brand-primary) !important;
}

/* Message bubbles - User */
.dark .message-user {
  background: var(--brand-input-bg) !important;
}

/* Message bubbles - Assistant */
.dark .message-assistant {
  background: transparent !important;
}

/* Scrollbar styling */
.dark ::-webkit-scrollbar {
  width: 8px;
}

.dark ::-webkit-scrollbar-track {
  background: var(--brand-darker-bg);
}

.dark ::-webkit-scrollbar-thumb {
  background: var(--brand-primary);
  border-radius: 4px;
}

/* === TYPOGRAPHY === */

/* Headings */
h1, h2, h3, h4, h5, h6 {
  font-family: 'Your-Brand-Font', sans-serif;
}

/* Body text */
body {
  font-family: 'Your-Body-Font', sans-serif;
}

/* === LOGO REPLACEMENT === */
/* Add your logo via environment variable or override here */
.logo-container img {
  content: url('/path/to/your/logo.png');
}
```

---

## Applying Custom CSS to LibreChat

### Method 1: Custom CSS File (Recommended)

1. Create `custom.css` in your LibreChat deployment
2. Mount it in your Docker/Railway config
3. Reference in LibreChat's configuration

### Method 2: Environment Variable

```env
CUSTOM_CSS_URL=https://your-cdn.com/custom.css
```

### Method 3: Direct File Override

Place CSS in `client/public/css/custom.css` (requires rebuild)

---

## Testing Checklist

Before deploying CSS changes:

- [ ] Test in Chrome AND Firefox
- [ ] Test light mode completely
- [ ] Test dark mode completely
- [ ] Check mobile responsiveness
- [ ] Verify input fields are usable
- [ ] Confirm scrolling works everywhere
- [ ] Test code block rendering
- [ ] Check link visibility
- [ ] Verify button hover states
