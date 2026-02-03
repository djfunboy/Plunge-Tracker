# Branding Guide
## Plunge Tracker v1.0

**Document Version:** 1.0  
**Last Updated:** February 2, 2026  
**Status:** Draft

---

## 1. Brand Overview

### 1.1 Brand Essence
Plunge Tracker is **bold, cool, and motivating**. It celebrates the mental toughness required for cold exposure while keeping the experience clean and modern.

### 1.2 Brand Personality
| Trait | Expression |
|-------|-----------|
| **Bold** | Strong typography, confident messaging |
| **Cool** | Ice/water color palette, clean design |
| **Motivating** | Streak celebrations, encouraging copy |
| **Simple** | Minimal UI, fast interactions |
| **Modern** | Contemporary design patterns, no clutter |

### 1.3 Brand Voice
- **Encouraging, not pushy** – "You've got this" not "Don't quit"
- **Casual but confident** – Like a supportive friend
- **Brief** – No long explanations; get to the point
- **Celebratory** – Acknowledge wins, big and small

**Examples:**
- ✅ "Let's go. Time to plunge."
- ✅ "7 days strong 🔥"
- ✅ "That's cold. Nice work."
- ❌ "You should really try to maintain your streak!"
- ❌ "Don't forget to log your session!"

---

## 2. Logo

### 2.1 Primary Logo
For v1, keep it simple: **wordmark only**.

**Wordmark:** "Plunge" or "Plunge Tracker"
- Font: Bold sans-serif (Inter Black, or similar)
- Can include a simple ice/water icon alongside

### 2.2 App Icon
A simple, recognizable icon for home screen:

**Concept Options:**
1. Snowflake ❄️ – Simple, universal cold symbol
2. Water droplet – Clean, minimal
3. Thermometer with cold indicator
4. Stylized "P" with ice/frost effect
5. Timer + snowflake combo

**Recommended:** Stylized snowflake or water droplet in brand colors. Keep it simple—should be recognizable at 48x48px.

### 2.3 Logo Usage
- Minimum size: 32px height
- Always maintain padding around logo
- Don't stretch, rotate, or add effects
- Use on dark backgrounds (primary) or light backgrounds (inverted)

---

## 3. Color Palette

### 3.1 Primary Colors

| Color | Hex | Usage |
|-------|-----|-------|
| **Deep Navy** | `#0f172a` | Primary background, dark mode |
| **Ice Blue** | `#0ea5e9` | Primary accent, buttons, highlights |
| **Glacier White** | `#f1f5f9` | Text on dark, light backgrounds |

### 3.2 Secondary Colors

| Color | Hex | Usage |
|-------|-----|-------|
| **Frost** | `#38bdf8` | Lighter accent, hover states |
| **Arctic** | `#7dd3fc` | Gradients, decorative elements |
| **Slate** | `#64748b` | Secondary text, borders |

### 3.3 Semantic Colors

| Color | Hex | Usage |
|-------|-----|-------|
| **Success Green** | `#22c55e` | Confirmations, completed states |
| **Streak Orange** | `#f97316` | Streak fire, achievements 🔥 |
| **Warning Amber** | `#eab308` | Caution states |
| **Error Red** | `#ef4444` | Errors, destructive actions |

### 3.4 Temperature Gradient
For visual temperature indicators:

```
Cold (30°F) ← ─────────────────── → Mild (70°F)
   #1e3a5f      #0ea5e9      #38bdf8      #7dd3fc
   Deep Blue    Ice Blue     Frost        Light Blue
```

### 3.5 Color Usage Examples

**Dark Mode (Primary):**
- Background: Deep Navy (`#0f172a`)
- Text: Glacier White (`#f1f5f9`)
- Accent: Ice Blue (`#0ea5e9`)

**Light Mode (Optional for v2):**
- Background: Glacier White (`#f1f5f9`)
- Text: Deep Navy (`#0f172a`)
- Accent: Ice Blue (`#0ea5e9`)

---

## 4. Typography

### 4.1 Font Family

**Primary Font:** Inter
- Free, open-source
- Excellent readability on screens
- Available on Google Fonts

**Fallback Stack:**
```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```

### 4.2 Type Scale

| Element | Size | Weight | Usage |
|---------|------|--------|-------|
| **Hero/Timer** | 64-80px | Bold (700) | Main timer display |
| **H1** | 32px | Bold (700) | Page titles |
| **H2** | 24px | Semibold (600) | Section headers |
| **H3** | 20px | Semibold (600) | Card titles |
| **Body** | 16px | Regular (400) | Main content |
| **Body Small** | 14px | Regular (400) | Secondary content |
| **Caption** | 12px | Medium (500) | Labels, metadata |

### 4.3 Timer Typography
The timer is the hero element—make it bold and unmissable:

```css
.timer-display {
  font-family: 'Inter', sans-serif;
  font-size: 72px;
  font-weight: 700;
  font-variant-numeric: tabular-nums; /* Prevents jumping */
  letter-spacing: -0.02em;
}
```

---

## 5. Iconography

### 5.1 Icon Style
- **Style:** Outlined or minimal filled
- **Stroke:** 2px
- **Corners:** Rounded
- **Library:** Lucide Icons (free, consistent with modern apps)

### 5.2 Core Icons Needed

| Icon | Usage |
|------|-------|
| Play ▶️ | Start timer |
| Pause ⏸️ | Stop timer |
| Check ✓ | Finish/Save |
| Snowflake ❄️ | Temperature, cold |
| Fire 🔥 | Streak indicator |
| Clock 🕐 | Duration, history |
| Map Pin 📍 | Location |
| Share ↗️ | Share card |
| List 📋 | History |
| Home 🏠 | Home navigation |

### 5.3 Rating Icons
For the 1-5 "How did it feel?" rating:

**Option A: Snowflakes**
- 1 = 😰 (one melting snowflake – struggling)
- 5 = 🧊 (five solid snowflakes – crushed it)

**Option B: Simple numbers**
- 1-5 in circles, highlight selected

**Option C: Faces/Emojis**
- 1 = 😵 → 5 = 💪

**Recommended for v1:** Simple numbers in circles (easiest to implement)

---

## 6. UI Components

### 6.1 Buttons

**Primary Button (CTA)**
```css
.btn-primary {
  background: #0ea5e9;
  color: white;
  padding: 16px 32px;
  border-radius: 12px;
  font-weight: 600;
  font-size: 18px;
}
```

**Secondary Button**
```css
.btn-secondary {
  background: transparent;
  border: 2px solid #0ea5e9;
  color: #0ea5e9;
  padding: 14px 30px;
  border-radius: 12px;
}
```

**Button Sizes:**
- Large: 56px height (timer controls)
- Medium: 48px height (form buttons)
- Small: 36px height (secondary actions)

### 6.2 Cards

```css
.card {
  background: #1e293b; /* Slightly lighter than bg */
  border-radius: 16px;
  padding: 20px;
  border: 1px solid #334155;
}
```

### 6.3 Slider (Temperature)

```css
.slider {
  accent-color: #0ea5e9;
  height: 8px;
  border-radius: 4px;
  background: #334155;
}
```

### 6.4 Input Fields

```css
.input {
  background: #1e293b;
  border: 1px solid #334155;
  border-radius: 8px;
  padding: 12px 16px;
  color: #f1f5f9;
}
.input:focus {
  border-color: #0ea5e9;
  outline: none;
}
```

---

## 7. Layout & Spacing

### 7.1 Spacing Scale
Use consistent spacing based on 4px increments:

| Token | Value | Usage |
|-------|-------|-------|
| `xs` | 4px | Tight spacing |
| `sm` | 8px | Icon gaps |
| `md` | 16px | Component padding |
| `lg` | 24px | Section spacing |
| `xl` | 32px | Major sections |
| `2xl` | 48px | Page padding |

### 7.2 Mobile Layout
- Max content width: 100% (full mobile width)
- Horizontal padding: 16-24px
- Safe area consideration for notched phones

### 7.3 Component Spacing
- Between cards: 16px
- Within cards: 16-20px padding
- Between form fields: 16px
- Button margin top: 24px

---

## 8. Shareable Card Design

### 8.1 Card Dimensions
- **Instagram Story:** 1080 x 1920px (9:16)
- **Square:** 1080 x 1080px (1:1)

### 8.2 Card Layout

```
┌────────────────────────────────┐
│                                │
│         [App Logo]             │
│                                │
│     ┌──────────────────┐       │
│     │                  │       │
│     │     2:45         │       │
│     │    duration      │       │
│     │                  │       │
│     └──────────────────┘       │
│                                │
│         42°F 🧊                │
│                                │
│      Austin, TX 📍             │
│                                │
│    ──────────────────          │
│                                │
│      🔥 7 day streak           │
│                                │
│        Feb 2, 2026             │
│                                │
│      [Plunge Tracker]          │
│                                │
└────────────────────────────────┘
```

### 8.3 Card Styling
- Background: Gradient from `#0f172a` to `#1e3a5f`
- Text: White (`#f1f5f9`)
- Accent elements: Ice Blue (`#0ea5e9`)
- Subtle ice/frost texture or pattern (optional)

---

## 9. Motion & Animation

### 9.1 Principles
- **Purposeful:** Animation should guide, not distract
- **Quick:** 150-300ms for most transitions
- **Smooth:** Use ease-out for entering, ease-in for exiting

### 9.2 Key Animations

| Element | Animation |
|---------|-----------|
| Timer running | Subtle pulse on the display |
| Button tap | Quick scale down/up (0.95 → 1) |
| Page transitions | Fade + slight slide (150ms) |
| Streak fire | Gentle flicker animation |
| Success save | Checkmark draw-in animation |

### 9.3 Timer Pulse
```css
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.8; }
}
.timer-running {
  animation: pulse 2s ease-in-out infinite;
}
```

---

## 10. Sample UI Mockup (Text-Based)

### Home Screen
```
┌─────────────────────────────┐
│                             │
│   🔥 7 day streak           │
│                             │
│  ┌───────────────────────┐  │
│  │                       │  │
│  │        02:45          │  │
│  │                       │  │
│  └───────────────────────┘  │
│                             │
│     [ ▶️ START PLUNGE ]     │
│                             │
│                             │
│  ─────────────────────────  │
│                             │
│   Recent: Today, 3:12, 38°F │
│                             │
├─────────────────────────────┤
│    🏠 Home      📋 History  │
└─────────────────────────────┘
```

### Session Form
```
┌─────────────────────────────┐
│                             │
│   Log Your Plunge 🧊        │
│                             │
│   Duration                  │
│   ┌───────────────────────┐ │
│   │       2:45            │ │
│   └───────────────────────┘ │
│                             │
│   Temperature               │
│   30°────●────────────70°F  │
│          42°F               │
│                             │
│   How'd it feel?            │
│   ① ② ③ ④ ⑤               │
│                             │
│   📍 Austin, TX             │
│                             │
│     [ 💾 SAVE SESSION ]     │
│                             │
└─────────────────────────────┘
```

---

## 11. Asset Checklist

### Required for Launch
- [ ] App icon (192x192, 512x512 PNG)
- [ ] Favicon (32x32)
- [ ] Open Graph image (1200x630) for link sharing
- [ ] Share card template

### Nice to Have
- [ ] Loading spinner/animation
- [ ] Empty state illustrations
- [ ] Onboarding graphics
- [ ] Custom snowflake icons

---

## 12. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2, 2026 | [Your Name] | Initial draft |

---

*This guide should evolve as the brand develops. Update as you make design decisions during development.*
