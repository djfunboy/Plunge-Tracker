# Technical Architecture Document
## Plunge Tracker v1.0

**Document Version:** 1.0  
**Last Updated:** February 2, 2026  
**Status:** Draft

---

## 1. System Overview

Plunge Tracker is a mobile-first Progressive Web App (PWA) built with a modern JAMstack architecture. The system prioritizes simplicity, speed, and offline capability.

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER DEVICE                              │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    React PWA (Vercel)                      │  │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────────┐   │  │
│  │  │  Timer  │  │ Session │  │ History │  │ Share Card  │   │  │
│  │  │  View   │  │  Form   │  │  List   │  │ Generator   │   │  │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────────┘   │  │
│  │                         │                                  │  │
│  │              ┌──────────┴──────────┐                      │  │
│  │              │   Local Storage     │                      │  │
│  │              │   (Offline Cache)   │                      │  │
│  │              └──────────┬──────────┘                      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                            │                                     │
│         ┌──────────────────┼──────────────────┐                 │
│         │                  │                  │                 │
│         ▼                  ▼                  ▼                 │
│  ┌────────────┐    ┌────────────┐    ┌──────────────┐          │
│  │  Browser   │    │  Browser   │    │   Canvas     │          │
│  │ Geolocation│    │   Timer    │    │  (Image Gen) │          │
│  │    API     │    │    API     │    │              │          │
│  └────────────┘    └────────────┘    └──────────────┘          │
└─────────────────────────────────────────────────────────────────┘
                             │
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                        EXTERNAL SERVICES                         │
│                                                                  │
│  ┌─────────────────────┐          ┌─────────────────────────┐   │
│  │      Supabase       │          │  OpenStreetMap Nominatim │   │
│  │  ┌───────────────┐  │          │  (Reverse Geocoding)     │   │
│  │  │  PostgreSQL   │  │          └─────────────────────────┘   │
│  │  │   Database    │  │                                        │
│  │  └───────────────┘  │                                        │
│  │  ┌───────────────┐  │                                        │
│  │  │ Edge Functions│  │                                        │
│  │  │  (if needed)  │  │                                        │
│  │  └───────────────┘  │                                        │
│  │  ┌───────────────┐  │                                        │
│  │  │   Auth (v2)   │  │                                        │
│  │  └───────────────┘  │                                        │
│  └─────────────────────┘                                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Tech Stack Details

### 2.1 Frontend

| Technology | Purpose | Why |
|------------|---------|-----|
| **React 18+** | UI Framework | Industry standard, large ecosystem |
| **Vite** | Build tool | Fast dev server, simple config |
| **Tailwind CSS** | Styling | Rapid prototyping, mobile-first |
| **React Router** | Navigation | Simple client-side routing |
| **html2canvas** | Image generation | Shareable card creation |
| **Zustand** (optional) | State management | Lightweight, simple API |

**Alternative:** Next.js if you want SSR/SSG, but Vite is simpler for a PWA.

### 2.2 Backend

| Technology | Purpose | Why |
|------------|---------|-----|
| **Supabase** | BaaS | PostgreSQL + API + Auth (for v2) |
| **Supabase JS Client** | Data access | Direct DB queries from frontend |
| **Edge Functions** | Server logic | Only if needed (geocoding proxy, etc.) |

### 2.3 Infrastructure

| Service | Purpose |
|---------|---------|
| **Vercel** | Frontend hosting, CDN, auto-deploys |
| **Supabase Cloud** | Database hosting, API |
| **OpenStreetMap Nominatim** | Free reverse geocoding |

---

## 3. Frontend Architecture

### 3.1 Project Structure

```
plunge-tracker/
├── public/
│   ├── manifest.json        # PWA manifest
│   ├── sw.js                # Service worker
│   ├── icons/               # App icons (various sizes)
│   └── favicon.ico
├── src/
│   ├── components/
│   │   ├── Timer/
│   │   │   ├── Timer.jsx
│   │   │   ├── TimerDisplay.jsx
│   │   │   └── TimerControls.jsx
│   │   ├── Session/
│   │   │   ├── SessionForm.jsx
│   │   │   ├── TemperatureSlider.jsx
│   │   │   ├── RatingPicker.jsx
│   │   │   └── LocationDisplay.jsx
│   │   ├── History/
│   │   │   ├── HistoryList.jsx
│   │   │   └── SessionCard.jsx
│   │   ├── Streak/
│   │   │   └── StreakBanner.jsx
│   │   ├── Share/
│   │   │   ├── ShareCard.jsx
│   │   │   └── ShareCardGenerator.jsx
│   │   └── ui/
│   │       ├── Button.jsx
│   │       ├── Slider.jsx
│   │       └── Modal.jsx
│   ├── hooks/
│   │   ├── useTimer.js
│   │   ├── useGeolocation.js
│   │   ├── useSessions.js
│   │   └── useStreak.js
│   ├── lib/
│   │   ├── supabase.js      # Supabase client init
│   │   ├── geocoding.js     # Reverse geocoding helper
│   │   ├── streak.js        # Streak calculation logic
│   │   └── storage.js       # Local storage helpers
│   ├── pages/
│   │   ├── Home.jsx         # Timer + Streak
│   │   ├── Log.jsx          # Session form
│   │   ├── History.jsx      # Session list
│   │   └── Share.jsx        # Share card preview
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── .env.local               # Supabase keys (not committed)
├── package.json
├── tailwind.config.js
└── vite.config.js
```

### 3.2 Key Components

#### Timer Component
```
State:
- isRunning: boolean
- elapsedTime: number (milliseconds)
- startTime: timestamp

Methods:
- start() → records startTime, sets isRunning true
- stop() → pauses timer, calculates elapsed
- finish() → stops timer, navigates to session form
- reset() → clears all timer state
```

#### Session Form Component
```
Props:
- duration: number (from timer)

State:
- temperature: number (30-70)
- rating: number (1-5)
- location: { lat, lng, display } | null
- isLoading: boolean

Methods:
- fetchLocation() → calls geolocation API
- saveSession() → writes to Supabase
```

### 3.3 State Management

For v1, keep it simple:

| State Type | Storage |
|------------|---------|
| Timer state | React useState + localStorage (persist across refresh) |
| Session data | Supabase (source of truth) |
| User ID | localStorage (anonymous UUID) |
| Offline queue | localStorage (sync when online) |

**If complexity grows:** Add Zustand for global state.

---

## 4. Backend Architecture

### 4.1 Supabase Setup

Supabase provides:
- **PostgreSQL database** – Stores all session data
- **Auto-generated REST API** – CRUD operations via JS client
- **Row Level Security (RLS)** – Data isolation per device/user
- **Auth (v2)** – Ready when you add login

### 4.2 Database Schema

See separate Database Schema document for full details.

**Core Tables:**

```
users (v1: anonymous devices)
├── id (UUID, PK)
├── device_id (text, unique)
├── created_at (timestamp)
└── timezone (text)

sessions
├── id (UUID, PK)
├── user_id (UUID, FK → users)
├── duration_seconds (integer)
├── temperature_f (integer)
├── rating (integer, 1-5)
├── location_lat (decimal, nullable)
├── location_lng (decimal, nullable)
├── location_display (text, nullable)
├── started_at (timestamp)
├── created_at (timestamp)
└── updated_at (timestamp)
```

### 4.3 API Endpoints (via Supabase Client)

No custom backend needed for v1. Use Supabase JS client directly:

| Operation | Supabase Call |
|-----------|---------------|
| Create user | `supabase.from('users').insert({...})` |
| Get/create user by device | `supabase.from('users').upsert({...})` |
| Save session | `supabase.from('sessions').insert({...})` |
| Get all sessions | `supabase.from('sessions').select('*').eq('user_id', id).order('started_at', { ascending: false })` |
| Get streak data | `supabase.from('sessions').select('started_at').eq('user_id', id)` |

### 4.4 Row Level Security (RLS)

Enable RLS to ensure users only see their own data:

```sql
-- Users can only read/write their own data
CREATE POLICY "Users can view own sessions" ON sessions
  FOR SELECT USING (user_id = current_user_id());

CREATE POLICY "Users can insert own sessions" ON sessions
  FOR INSERT WITH CHECK (user_id = current_user_id());
```

**For v1 (anonymous):** Pass device_id as a header or use Supabase anonymous auth.

---

## 5. Data Flow

### 5.1 Session Creation Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Timer   │───▶│  Form    │───▶│ Validate │───▶│ Supabase │
│  Finish  │    │  Submit  │    │  + Geo   │    │  Insert  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
                                      │
                                      ▼
                               ┌──────────┐
                               │ Nominatim│
                               │ Geocode  │
                               └──────────┘
```

**Detailed Steps:**
1. User taps "Finish" on timer
2. App captures duration, navigates to form
3. User adjusts temperature, rating
4. App requests geolocation permission (if first time)
5. Browser returns lat/lng coordinates
6. App calls Nominatim API to get "City, State"
7. User taps "Save"
8. App writes session to Supabase
9. App recalculates streak
10. Success screen with share option

### 5.2 Offline Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│  Save    │───▶│  No      │───▶│  Queue   │
│  Session │    │  Network │    │  Local   │
└──────────┘    └──────────┘    └──────────┘
                                      │
                    (when online)     │
                                      ▼
                               ┌──────────┐
                               │  Sync    │
                               │ Supabase │
                               └──────────┘
```

---

## 6. Third-Party Integrations

### 6.1 OpenStreetMap Nominatim (Reverse Geocoding)

**Endpoint:**
```
GET https://nominatim.openstreetmap.org/reverse
  ?lat={latitude}
  &lon={longitude}
  &format=json
```

**Rate Limits:**
- 1 request per second (strict)
- Must include User-Agent header
- No API key required

**Response (simplified):**
```json
{
  "address": {
    "city": "Austin",
    "state": "Texas",
    "country": "United States"
  }
}
```

**Fallback:** If geocoding fails, store coordinates only, display "Location saved" or allow manual entry.

### 6.2 Browser APIs

| API | Purpose | Fallback |
|-----|---------|----------|
| Geolocation API | Get lat/lng | "Location unavailable" |
| Performance.now() | Precise timer | Date.now() |
| localStorage | Offline data, device ID | In-memory (session only) |
| Service Worker | PWA caching | Online-only mode |

---

## 7. PWA Configuration

### 7.1 Manifest (manifest.json)

```json
{
  "name": "Plunge Tracker",
  "short_name": "Plunge",
  "description": "Track your cold plunge sessions",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0f172a",
  "theme_color": "#0ea5e9",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### 7.2 Service Worker Strategy

| Resource | Strategy |
|----------|----------|
| App shell (HTML, CSS, JS) | Cache-first |
| API calls | Network-first, cache fallback |
| Images | Cache-first |
| Session data | Network-first, queue if offline |

---

## 8. Security Considerations

### 8.1 V1 Security (Anonymous Users)

| Concern | Mitigation |
|---------|------------|
| Data isolation | RLS policies on Supabase |
| Device spoofing | Accept risk for v1; auth in v2 |
| API abuse | Supabase rate limiting |
| XSS | React's built-in escaping |
| HTTPS | Enforced by Vercel |

### 8.2 Environment Variables

```
# .env.local (never commit)
VITE_SUPABASE_URL=https://xxxxx.supabase.co
VITE_SUPABASE_ANON_KEY=xxxxx
```

### 8.3 Future Security (V2)

- Supabase Auth (email/password, OAuth)
- JWT-based API access
- Proper user accounts replacing device IDs

---

## 9. Performance Targets

| Metric | Target |
|--------|--------|
| First Contentful Paint | < 1.5s |
| Time to Interactive | < 3s |
| Lighthouse Score | > 90 |
| Bundle Size | < 200KB gzipped |
| Offline Capability | Core features work offline |

---

## 10. Deployment Pipeline

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Code    │───▶│  GitHub  │───▶│  Vercel  │───▶│  Live    │
│  Push    │    │   Repo   │    │  Build   │    │  Site    │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

**Setup:**
1. Create GitHub repo
2. Connect to Vercel
3. Add environment variables in Vercel dashboard
4. Push to `main` = auto-deploy to production
5. Push to feature branch = preview deployment

---

## 11. Scalability Notes (Future)

This architecture supports growth:

| Scale Trigger | Solution |
|---------------|----------|
| 1,000+ users | Current stack handles fine |
| 10,000+ users | Add Supabase indexes, connection pooling |
| Heavy API use | Add Edge Functions as proxy/cache layer |
| Native app | Wrap with Capacitor (reuse 90% of code) |
| Auth needed | Enable Supabase Auth, migrate device_ids |

---

## 12. Development Environment

### Required Tools
- Node.js 18+
- npm or pnpm
- Git
- VS Code (recommended)

### Getting Started
```bash
# Clone repo
git clone https://github.com/you/plunge-tracker.git
cd plunge-tracker

# Install dependencies
npm install

# Set up environment
cp .env.example .env.local
# Add your Supabase keys

# Run dev server
npm run dev

# Build for production
npm run build
```

---

## 13. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2, 2026 | [Your Name] | Initial draft |

---

*This document should be updated as technical decisions evolve during development.*
