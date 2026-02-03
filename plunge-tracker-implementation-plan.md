# Implementation Plan
## Plunge Tracker v1.0

**Document Version:** 1.0  
**Last Updated:** February 2, 2026  
**Estimated Build Time:** 4-6 hours (beginner) | 2-3 hours (experienced)

---

## Overview

This plan breaks the build into 7 phases. Each phase ends with a working checkpoint—something you can see and test before moving on.

**Philosophy:** Get something working fast, then iterate. Don't perfect Phase 1 before starting Phase 2.

---

## Pre-Build Checklist

Before you write any code, set up your accounts and tools:

- [ ] **Node.js installed** (v18+) → [nodejs.org](https://nodejs.org)
- [ ] **Code editor** → VS Code recommended
- [ ] **GitHub account** → [github.com](https://github.com)
- [ ] **Vercel account** → [vercel.com](https://vercel.com) (sign up with GitHub)
- [ ] **Supabase account** → [supabase.com](https://supabase.com)

**Time:** 15-30 minutes (if you don't have these already)

---

## Phase 1: Project Setup
**Time Estimate:** 20-30 minutes

### Goals
- Create React project
- Install dependencies
- Connect to GitHub
- Deploy "Hello World" to Vercel

### Steps

1. **Create the project**
   ```bash
   npm create vite@latest plunge-tracker -- --template react
   cd plunge-tracker
   npm install
   ```

2. **Install dependencies**
   ```bash
   npm install @supabase/supabase-js
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init -p
   ```

3. **Configure Tailwind** (tailwind.config.js)
   ```javascript
   content: ["./index.html", "./src/**/*.{js,jsx}"]
   ```

4. **Add Tailwind to CSS** (src/index.css)
   ```css
   @tailwind base;
   @tailwind components;
   @tailwind utilities;
   ```

5. **Test locally**
   ```bash
   npm run dev
   ```
   → Should see Vite welcome page at localhost:5173

6. **Push to GitHub**
   ```bash
   git init
   git add .
   git commit -m "Initial setup"
   # Create repo on GitHub, then:
   git remote add origin https://github.com/YOU/plunge-tracker.git
   git push -u origin main
   ```

7. **Deploy to Vercel**
   - Go to vercel.com → "Add New Project"
   - Import your GitHub repo
   - Click Deploy
   - Wait ~1 minute

### ✅ Checkpoint
- [ ] App runs locally
- [ ] App is live on Vercel (yourproject.vercel.app)
- [ ] Tailwind is working (test with a colored div)

---

## Phase 2: Supabase Setup
**Time Estimate:** 20-30 minutes

### Goals
- Create Supabase project
- Set up database tables
- Connect frontend to Supabase

### Steps

1. **Create Supabase project**
   - Go to supabase.com → "New Project"
   - Name: "plunge-tracker"
   - Choose region closest to you
   - Save the password!

2. **Create database tables** (SQL Editor in Supabase)
   ```sql
   -- Users table (anonymous device tracking)
   CREATE TABLE users (
     id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
     device_id TEXT UNIQUE NOT NULL,
     timezone TEXT,
     created_at TIMESTAMPTZ DEFAULT NOW()
   );

   -- Sessions table
   CREATE TABLE sessions (
     id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
     user_id UUID REFERENCES users(id) ON DELETE CASCADE,
     duration_seconds INTEGER NOT NULL,
     temperature_f INTEGER NOT NULL,
     rating INTEGER CHECK (rating >= 1 AND rating <= 5),
     location_lat DECIMAL(10, 8),
     location_lng DECIMAL(11, 8),
     location_display TEXT,
     started_at TIMESTAMPTZ NOT NULL,
     created_at TIMESTAMPTZ DEFAULT NOW()
   );

   -- Index for faster queries
   CREATE INDEX sessions_user_id_idx ON sessions(user_id);
   CREATE INDEX sessions_started_at_idx ON sessions(started_at DESC);
   ```

3. **Enable Row Level Security**
   ```sql
   ALTER TABLE users ENABLE ROW LEVEL SECURITY;
   ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;

   -- For v1 (anonymous), allow all operations
   -- Tighten this in v2 with proper auth
   CREATE POLICY "Allow all for now" ON users FOR ALL USING (true);
   CREATE POLICY "Allow all for now" ON sessions FOR ALL USING (true);
   ```

4. **Get your API keys**
   - Go to Settings → API
   - Copy: Project URL and anon/public key

5. **Add to your project**
   Create `.env.local` in project root:
   ```
   VITE_SUPABASE_URL=https://xxxxx.supabase.co
   VITE_SUPABASE_ANON_KEY=xxxxx
   ```

6. **Create Supabase client** (src/lib/supabase.js)
   ```javascript
   import { createClient } from '@supabase/supabase-js'

   export const supabase = createClient(
     import.meta.env.VITE_SUPABASE_URL,
     import.meta.env.VITE_SUPABASE_ANON_KEY
   )
   ```

7. **Add env vars to Vercel**
   - Vercel dashboard → Settings → Environment Variables
   - Add both VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY

### ✅ Checkpoint
- [ ] Supabase project created
- [ ] Tables exist (check Table Editor)
- [ ] Can connect from local app (no errors in console)

---

## Phase 3: Core Timer Feature
**Time Estimate:** 45-60 minutes

### Goals
- Build the timer UI
- Start/Stop/Finish functionality
- Timer persists if page refreshes

### Steps

1. **Create timer hook** (src/hooks/useTimer.js)
   - Track: isRunning, elapsedTime, startTime
   - Methods: start(), stop(), finish(), reset()
   - Persist startTime to localStorage

2. **Create Timer component** (src/components/Timer.jsx)
   - Large time display (MM:SS)
   - Start button (green)
   - Stop button (yellow) - appears when running
   - Finish button (blue) - appears when stopped
   - Visual indicator when running (pulse animation)

3. **Create Home page** (src/pages/Home.jsx)
   - Timer component centered
   - "Let's Plunge" or similar headline
   - Eventually: streak banner at top

4. **Style with Tailwind**
   - Large touch targets (min 48px)
   - High contrast colors
   - Mobile-first layout

### ✅ Checkpoint
- [ ] Timer starts and counts up
- [ ] Timer stops and holds time
- [ ] Refreshing page doesn't lose running timer
- [ ] Looks decent on mobile

---

## Phase 4: Session Logging
**Time Estimate:** 45-60 minutes

### Goals
- Build session form
- Temperature slider
- Rating picker
- Save to Supabase

### Steps

1. **Create session form** (src/components/SessionForm.jsx)
   - Pre-filled duration (from timer)
   - Temperature slider (30-70°F)
   - Rating picker (1-5)
   - Save button

2. **Create Temperature Slider**
   - Use native HTML range input
   - Style with Tailwind
   - Show current value prominently

3. **Create Rating Picker**
   - 5 tappable elements (icons or numbers)
   - Clear selected state

4. **Create device ID utility** (src/lib/device.js)
   ```javascript
   export function getDeviceId() {
     let id = localStorage.getItem('device_id')
     if (!id) {
       id = crypto.randomUUID()
       localStorage.setItem('device_id', id)
     }
     return id
   }
   ```

5. **Create/get user on app load**
   - Check if user exists with this device_id
   - If not, create one
   - Store user_id for session saves

6. **Save session to Supabase**
   ```javascript
   const { error } = await supabase
     .from('sessions')
     .insert({
       user_id: userId,
       duration_seconds: duration,
       temperature_f: temperature,
       rating: rating,
       started_at: new Date().toISOString()
     })
   ```

7. **Add routing** (if not already)
   ```bash
   npm install react-router-dom
   ```
   - Home → /
   - Log → /log
   - Finish button navigates to /log with duration

### ✅ Checkpoint
- [ ] Finish timer → see session form
- [ ] Slider works smoothly
- [ ] Rating selection works
- [ ] Save button → data appears in Supabase table
- [ ] Success message shows

---

## Phase 5: Location & History
**Time Estimate:** 45-60 minutes

### Goals
- Add GPS location to sessions
- Build history list
- Show past sessions

### Steps

1. **Create geolocation hook** (src/hooks/useGeolocation.js)
   ```javascript
   // Request permission
   // Get lat/lng
   // Handle errors gracefully
   ```

2. **Create geocoding utility** (src/lib/geocoding.js)
   ```javascript
   export async function reverseGeocode(lat, lng) {
     const res = await fetch(
       `https://nominatim.openstreetmap.org/reverse?lat=${lat}&lon=${lng}&format=json`,
       { headers: { 'User-Agent': 'PlungeTracker/1.0' } }
     )
     const data = await res.json()
     return `${data.address.city}, ${data.address.state}`
   }
   ```

3. **Integrate location into session save**
   - Request location when form loads
   - Show "Detecting location..." state
   - Display location or "Location unavailable"
   - Save with session

4. **Create History page** (src/pages/History.jsx)
   - Fetch all sessions for user
   - Display in reverse chronological order
   - Show: date, duration, temp, rating, location

5. **Create SessionCard component**
   - Compact card for each session
   - Nice formatting (e.g., "2:45" not "165 seconds")
   - Color coding by temperature (optional)

6. **Add navigation**
   - Bottom nav or header with Home / History

### ✅ Checkpoint
- [ ] Location permission prompt works
- [ ] Location shows as "City, State"
- [ ] Graceful fallback if denied
- [ ] History page shows all sessions
- [ ] Sessions sorted newest first

---

## Phase 6: Streak Tracker
**Time Estimate:** 30-45 minutes

### Goals
- Calculate current streak
- Calculate longest streak
- Display streak on home page

### Steps

1. **Create streak utility** (src/lib/streak.js)
   ```javascript
   export function calculateStreak(sessions) {
     // Sort sessions by date
     // Group by day (user's timezone)
     // Count consecutive days from today backwards
     // Return { current: X, longest: Y }
   }
   ```

2. **Create StreakBanner component**
   - "🔥 X day streak!"
   - Show longest streak below
   - Encouraging message if streak is 0

3. **Add to Home page**
   - Fetch sessions on load
   - Calculate streak
   - Display banner above timer

4. **Update after each session save**
   - Recalculate streak
   - Show updated count

### ✅ Checkpoint
- [ ] Streak shows on home page
- [ ] Streak increments after saving session
- [ ] Streak resets if day is missed
- [ ] Longest streak tracks correctly

---

## Phase 7: Shareable Cards
**Time Estimate:** 45-60 minutes

### Goals
- Generate Spotify-style share image
- Save to camera roll
- Polish and launch!

### Steps

1. **Install html2canvas**
   ```bash
   npm install html2canvas
   ```

2. **Create ShareCard component** (src/components/ShareCard.jsx)
   - Design a nice card layout
   - Include: duration, temp, date, streak, location
   - Brand it with app name/logo
   - Size for Instagram Stories (1080x1920) or square

3. **Create card generator utility**
   ```javascript
   import html2canvas from 'html2canvas'

   export async function generateShareImage(element) {
     const canvas = await html2canvas(element)
     return canvas.toDataURL('image/png')
   }
   ```

4. **Add Share button to session success screen**
   - "Share Your Plunge" button
   - Opens preview of card
   - "Save to Photos" button

5. **Implement save functionality**
   ```javascript
   // Create download link
   const link = document.createElement('a')
   link.download = 'my-plunge.png'
   link.href = imageDataUrl
   link.click()
   ```

6. **Polish pass**
   - Test on real phone
   - Fix any mobile issues
   - Add loading states
   - Add error handling
   - Empty states for history

7. **Final deploy**
   ```bash
   git add .
   git commit -m "v1.0 complete"
   git push
   ```
   → Vercel auto-deploys

### ✅ Checkpoint
- [ ] Share card generates correctly
- [ ] Image saves to device
- [ ] Card looks good (test on real phone)
- [ ] All features working end-to-end
- [ ] App is live and shareable!

---

## Post-Launch Checklist

After v1 is live:

- [ ] Test on multiple devices (iOS Safari, Android Chrome)
- [ ] Share with 3-5 friends for feedback
- [ ] Monitor Supabase for any errors
- [ ] Make note of feature requests
- [ ] Celebrate! 🎉

---

## Quick Reference: File Structure

```
plunge-tracker/
├── src/
│   ├── components/
│   │   ├── Timer.jsx
│   │   ├── SessionForm.jsx
│   │   ├── TemperatureSlider.jsx
│   │   ├── RatingPicker.jsx
│   │   ├── SessionCard.jsx
│   │   ├── StreakBanner.jsx
│   │   └── ShareCard.jsx
│   ├── hooks/
│   │   ├── useTimer.js
│   │   ├── useGeolocation.js
│   │   └── useSessions.js
│   ├── lib/
│   │   ├── supabase.js
│   │   ├── device.js
│   │   ├── geocoding.js
│   │   └── streak.js
│   ├── pages/
│   │   ├── Home.jsx
│   │   ├── Log.jsx
│   │   └── History.jsx
│   ├── App.jsx
│   └── main.jsx
├── .env.local
└── package.json
```

---

## Troubleshooting Common Issues

| Problem | Solution |
|---------|----------|
| Supabase connection fails | Check env vars, make sure they're prefixed with VITE_ |
| Location not working | Must be on HTTPS (works on Vercel, not localhost without config) |
| Timer resets on refresh | Make sure startTime is saved to localStorage |
| Styles not applying | Check Tailwind content config includes your files |
| Vercel deploy fails | Check build logs, usually missing dependency or env var |

---

## What's Next (V1.1 and V2)

Once v1 is stable, consider:

**V1.1 (Quick Wins)**
- Edit/delete sessions
- Celsius toggle
- PWA manifest for "Add to Home Screen"
- Basic offline support

**V2 (Bigger Features)**
- User authentication (Supabase Auth)
- Social login (Google, Apple)
- Strava integration
- Instagram direct share
- Data export
- Web dashboard

---

## Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2, 2026 | [Your Name] | Initial draft |

---

*You've got this. Ship it!* 🧊🔥
