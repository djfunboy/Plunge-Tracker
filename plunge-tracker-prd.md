# Product Requirements Document (PRD)
## Plunge Tracker v1.0

**Document Version:** 1.0  
**Last Updated:** February 2, 2026  
**Author:** [Your Name]  
**Status:** Draft

---

## 1. Product Overview

### 1.1 Vision
Plunge Tracker is a mobile-friendly web application that helps cold plunge enthusiasts track their sessions, build consistency through streaks, and share their achievements on social media.

### 1.2 Problem Statement
Cold plunge practitioners lack a simple, dedicated tool to:
- Track their sessions (duration, temperature, how they felt)
- Build and maintain streaking habits
- Share their achievements in a visually appealing way

### 1.3 Solution
A clean, focused PWA that lets users time their plunges, log key data, view their history, and generate shareable social cards—all without requiring an account.

---

## 2. Target Users

### Primary Persona: "The Cold Plunge Enthusiast"
- Practices cold exposure regularly (daily or several times per week)
- Values tracking progress and building habits
- Active on social media (Instagram, Twitter/X, Strava community)
- Uses mobile device primarily
- Motivated by streaks and visible progress

### User Goals
1. Quickly start a timer when stepping into the plunge
2. Log session details with minimal friction
3. See progress over time and maintain streaks
4. Share achievements to social media

---

## 3. Core Features

### 3.1 Timer (Must Have)
**Description:** A prominent stopwatch for timing cold plunge sessions.

**Acceptance Criteria:**
- [ ] Large, easy-to-read time display (MM:SS format)
- [ ] "Start" button begins the timer
- [ ] "Stop" button pauses the timer
- [ ] "Finish" button ends session and opens the session logging form
- [ ] Timer continues if user navigates away or locks phone (background state)
- [ ] Visual feedback when timer is running (e.g., pulsing animation or color change)

**UI Notes:**
- Timer should be the hero element on the home screen
- Buttons should be large enough for cold, wet fingers

---

### 3.2 Session Logging (Must Have)
**Description:** After completing a plunge, users log session details.

**Data Captured:**
| Field | Input Type | Details |
|-------|-----------|---------|
| Duration | Automatic | From timer (editable if needed) |
| Date/Time | Automatic | Timestamp when session finished |
| Temperature | Slider | 30°F – 70°F, 1°F increments |
| Rating | 1-5 scale | "How did it feel?" |
| Location | Automatic (GPS) | Reverse geocoded to "City, State" |

**Acceptance Criteria:**
- [ ] Pre-filled with timer duration (editable)
- [ ] Temperature slider is smooth and displays current value
- [ ] Rating uses tappable icons (e.g., 1-5 snowflakes or faces)
- [ ] Location auto-detects on session save (with user permission)
- [ ] Falls back to "Location unavailable" if GPS denied/fails
- [ ] "Save Session" button stores to database
- [ ] Success confirmation after save

---

### 3.3 Session History (Must Have)
**Description:** A scrollable list of all past sessions.

**Acceptance Criteria:**
- [ ] Displays sessions in reverse chronological order (newest first)
- [ ] Each session card shows: Date, Duration, Temperature, Rating, Location
- [ ] Pull-to-refresh functionality
- [ ] Empty state message for new users ("No plunges yet—time to get cold! 🧊")

**UI Notes:**
- Keep cards compact but readable
- Consider subtle color coding by temperature (colder = more blue)

---

### 3.4 Streak Tracker (Must Have)
**Description:** Duolingo-style streak counter to encourage daily consistency.

**Acceptance Criteria:**
- [ ] Displays current streak prominently on home screen
- [ ] Streak = consecutive days with at least one session
- [ ] Shows "🔥 X day streak!" with flame icon
- [ ] Streak resets if a day is missed
- [ ] Also displays "Longest streak" stat
- [ ] Streak persists across browser sessions (stored in database)

**Streak Logic:**
- A "day" is defined as midnight-to-midnight in user's local timezone
- Multiple sessions in one day count as one day toward streak
- Grace period: None for v1 (strict streaks)

---

### 3.5 Shareable Session Card (Must Have)
**Description:** Spotify-style visual card for social sharing.

**Acceptance Criteria:**
- [ ] "Share" button on session detail or post-save screen
- [ ] Generates a branded image containing:
  - App logo/name
  - Duration
  - Temperature
  - Date
  - Current streak
  - Location (optional)
- [ ] Sized for Instagram Stories (1080x1920) or square (1080x1080)
- [ ] User can save image to device camera roll
- [ ] Clean, visually appealing design

**Technical Notes:**
- Use HTML Canvas or a library like html2canvas to generate image
- No external API required

---

### 3.6 Location Detection (Must Have)
**Description:** Automatic GPS-based location capture.

**Acceptance Criteria:**
- [ ] Requests location permission on first session save
- [ ] Captures lat/long coordinates
- [ ] Reverse geocodes to human-readable format ("Austin, TX")
- [ ] Graceful fallback if permission denied or API fails
- [ ] Uses free reverse geocoding service (OpenStreetMap Nominatim)

**Privacy Notes:**
- Only captures location at moment of save
- No continuous tracking
- Stored as city/state only (not exact coordinates) for privacy

---

## 4. Technical Requirements

### 4.1 Tech Stack
| Layer | Technology |
|-------|-----------|
| Frontend | React (Next.js or Vite) |
| Styling | Tailwind CSS |
| Backend | Supabase (PostgreSQL + Edge Functions if needed) |
| Hosting | Vercel |
| Location | Browser Geolocation API + OpenStreetMap Nominatim |
| Image Generation | html2canvas or similar |

### 4.2 Data Storage
- **V1:** All data stored in Supabase with anonymous/device-based identification
- **No authentication required** for v1
- Use device fingerprint or localStorage UUID to associate sessions with a "user"

### 4.3 PWA Requirements
- [ ] Manifest file for "Add to Home Screen"
- [ ] Works offline for viewing history (service worker caching)
- [ ] Timer functions offline
- [ ] Syncs when back online

### 4.4 Browser Support
- iOS Safari (primary)
- Chrome Mobile (Android)
- Desktop browsers (secondary)

---

## 5. Out of Scope (V1)

The following features are explicitly **NOT** included in v1:

| Feature | Reason | Future Version |
|---------|--------|----------------|
| User authentication | Adds complexity; not needed for MVP | V2 |
| Strava integration | OAuth complexity | V2 |
| Instagram direct posting | API complexity | V2+ |
| Apple HealthKit | Requires native app | V2+ |
| Social/community features | Scope creep | V2+ |
| Notifications/reminders | Adds complexity | V2 |
| Payments/billing | Not needed for MVP | V2 |
| Apple Watch app | Requires native development | V3 |
| Multiple temperature units | Keep simple; F only | V2 |
| Session editing/deletion | Nice to have | V1.1 |
| Data export | Nice to have | V1.1 |

---

## 6. User Flows

### 6.1 First-Time User Flow
```
1. User opens app
2. Sees home screen with timer and "0 day streak"
3. Taps "Start" to begin first plunge
4. Timer runs
5. Taps "Finish" when done
6. Session logging form appears (pre-filled duration)
7. Adjusts temperature slider
8. Taps rating (1-5)
9. App requests location permission
10. Taps "Save Session"
11. Success! Streak updates to "1 day 🔥"
12. Option to share or return home
```

### 6.2 Returning User Flow
```
1. User opens app
2. Sees current streak and recent sessions
3. Taps "Start" for new plunge
4. Completes session
5. Saves with details
6. Streak continues or resets based on timing
```

### 6.3 Share Flow
```
1. User completes session (or views past session)
2. Taps "Share" button
3. Preview of shareable card appears
4. Taps "Save to Photos" or "Copy"
5. Opens Instagram/Twitter manually to post
```

---

## 7. Success Metrics

### Primary Metrics (V1)
| Metric | Target | Measurement |
|--------|--------|-------------|
| Sessions logged | 100+ total | Database count |
| Return users | 30%+ use app more than once | Unique device tracking |
| Streak engagement | 20%+ maintain 3+ day streak | Database query |

### Qualitative Signals
- Users sharing cards on social media
- Positive feedback from test users
- Requests for specific features (signals product-market fit)

---

## 8. Design Principles

1. **Speed over features** – Get to the timer in zero taps
2. **One-handed operation** – Usable with cold, wet hands
3. **Motivating, not nagging** – Streaks encourage, not guilt
4. **Visually cool** – Cold/ice aesthetic that users want to share
5. **Offline-first** – Works even with spotty connection

---

## 9. Open Questions

- [ ] Exact branding/color palette (see Branding Guide)
- [ ] Specific rating icons (snowflakes? faces? numbers?)
- [ ] Session card design details
- [ ] Sound/haptic feedback on timer?

---

## 10. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2, 2026 | [Your Name] | Initial draft |

---

*This document will be updated as decisions are made during development.*
