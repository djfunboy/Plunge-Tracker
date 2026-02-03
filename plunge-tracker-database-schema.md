# Database Schema
## Plunge Tracker v1.0

**Document Version:** 1.0  
**Last Updated:** February 2, 2026  
**Database:** PostgreSQL (via Supabase)  
**Status:** Draft

---

## 1. Schema Overview

The v1 database is intentionally simple: two tables to support anonymous session tracking with device-based user identification.

```
┌─────────────────┐       ┌─────────────────────────┐
│     users       │       │        sessions         │
├─────────────────┤       ├─────────────────────────┤
│ id (PK)         │──────<│ user_id (FK)            │
│ device_id       │       │ id (PK)                 │
│ timezone        │       │ duration_seconds        │
│ created_at      │       │ temperature_f           │
│                 │       │ rating                  │
│                 │       │ location_lat            │
│                 │       │ location_lng            │
│                 │       │ location_display        │
│                 │       │ started_at              │
│                 │       │ created_at              │
└─────────────────┘       └─────────────────────────┘
```

---

## 2. Tables

### 2.1 users

Stores anonymous user records tied to device identifiers. Will be upgraded to full auth in v2.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `UUID` | PRIMARY KEY, DEFAULT `gen_random_uuid()` | Unique user identifier |
| `device_id` | `TEXT` | UNIQUE, NOT NULL | Client-generated device fingerprint |
| `timezone` | `TEXT` | NULLABLE | User's timezone (e.g., "America/Chicago") |
| `created_at` | `TIMESTAMPTZ` | DEFAULT `NOW()` | Account creation timestamp |

**SQL:**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id TEXT UNIQUE NOT NULL,
    timezone TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Index for device lookup
CREATE INDEX users_device_id_idx ON users(device_id);
```

**Notes:**
- `device_id` is generated client-side using `crypto.randomUUID()` and stored in localStorage
- In v2, this table will be replaced/augmented with Supabase Auth

---

### 2.2 sessions

Stores individual cold plunge session records.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | `UUID` | PRIMARY KEY, DEFAULT `gen_random_uuid()` | Unique session identifier |
| `user_id` | `UUID` | FOREIGN KEY → users(id), NOT NULL | Owner of the session |
| `duration_seconds` | `INTEGER` | NOT NULL, CHECK >= 0 | Length of plunge in seconds |
| `temperature_f` | `INTEGER` | NOT NULL, CHECK 30-70 | Water temperature in Fahrenheit |
| `rating` | `INTEGER` | CHECK 1-5 | "How did it feel?" rating |
| `location_lat` | `DECIMAL(10,8)` | NULLABLE | GPS latitude |
| `location_lng` | `DECIMAL(11,8)` | NULLABLE | GPS longitude |
| `location_display` | `TEXT` | NULLABLE | Human-readable location ("Austin, TX") |
| `started_at` | `TIMESTAMPTZ` | NOT NULL | When the session began |
| `created_at` | `TIMESTAMPTZ` | DEFAULT `NOW()` | When the record was created |

**SQL:**
```sql
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    duration_seconds INTEGER NOT NULL CHECK (duration_seconds >= 0),
    temperature_f INTEGER NOT NULL CHECK (temperature_f >= 30 AND temperature_f <= 70),
    rating INTEGER CHECK (rating >= 1 AND rating <= 5),
    location_lat DECIMAL(10, 8),
    location_lng DECIMAL(11, 8),
    location_display TEXT,
    started_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes for common queries
CREATE INDEX sessions_user_id_idx ON sessions(user_id);
CREATE INDEX sessions_started_at_idx ON sessions(started_at DESC);
CREATE INDEX sessions_user_started_idx ON sessions(user_id, started_at DESC);
```

**Notes:**
- `duration_seconds` stores raw seconds; format as MM:SS in the UI
- `temperature_f` constrained to valid range; expand if needed for extreme plungers
- `location_lat/lng` stored separately for potential future mapping features
- `started_at` is when the timer was started; `created_at` is when the record was saved

---

## 3. Row Level Security (RLS)

Supabase RLS ensures users can only access their own data.

### 3.1 V1 Policies (Anonymous/Permissive)

For v1 without auth, we use permissive policies. Tighten in v2.

```sql
-- Enable RLS on both tables
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;

-- Users table: Allow all operations (v1 only)
CREATE POLICY "users_allow_all" ON users
    FOR ALL
    USING (true)
    WITH CHECK (true);

-- Sessions table: Allow all operations (v1 only)
CREATE POLICY "sessions_allow_all" ON sessions
    FOR ALL
    USING (true)
    WITH CHECK (true);
```

### 3.2 V2 Policies (With Auth)

When you add Supabase Auth, replace with these:

```sql
-- Users can only see their own record
CREATE POLICY "users_select_own" ON users
    FOR SELECT
    USING (auth.uid() = id);

-- Users can only insert their own record
CREATE POLICY "users_insert_own" ON users
    FOR INSERT
    WITH CHECK (auth.uid() = id);

-- Sessions: users can only see their own
CREATE POLICY "sessions_select_own" ON sessions
    FOR SELECT
    USING (user_id = auth.uid());

-- Sessions: users can only insert their own
CREATE POLICY "sessions_insert_own" ON sessions
    FOR INSERT
    WITH CHECK (user_id = auth.uid());

-- Sessions: users can only update their own
CREATE POLICY "sessions_update_own" ON sessions
    FOR UPDATE
    USING (user_id = auth.uid());

-- Sessions: users can only delete their own
CREATE POLICY "sessions_delete_own" ON sessions
    FOR DELETE
    USING (user_id = auth.uid());
```

---

## 4. Common Queries

### 4.1 Create or Get User by Device ID

```javascript
// Upsert user (create if not exists, return if exists)
const { data: user, error } = await supabase
    .from('users')
    .upsert(
        { device_id: deviceId, timezone: userTimezone },
        { onConflict: 'device_id' }
    )
    .select()
    .single();
```

### 4.2 Save New Session

```javascript
const { data, error } = await supabase
    .from('sessions')
    .insert({
        user_id: userId,
        duration_seconds: 165, // 2:45
        temperature_f: 42,
        rating: 4,
        location_lat: 30.2672,
        location_lng: -97.7431,
        location_display: 'Austin, TX',
        started_at: new Date().toISOString()
    })
    .select()
    .single();
```

### 4.3 Get All Sessions for User

```javascript
const { data: sessions, error } = await supabase
    .from('sessions')
    .select('*')
    .eq('user_id', userId)
    .order('started_at', { ascending: false });
```

### 4.4 Get Sessions for Streak Calculation

```javascript
// Get just dates for streak calculation (lighter query)
const { data: sessions, error } = await supabase
    .from('sessions')
    .select('started_at')
    .eq('user_id', userId)
    .order('started_at', { ascending: false });
```

### 4.5 Get Session Count and Stats

```javascript
// Total sessions
const { count } = await supabase
    .from('sessions')
    .select('*', { count: 'exact', head: true })
    .eq('user_id', userId);

// Average duration
const { data } = await supabase
    .from('sessions')
    .select('duration_seconds')
    .eq('user_id', userId);

const avgDuration = data.reduce((sum, s) => sum + s.duration_seconds, 0) / data.length;
```

### 4.6 Get Recent Sessions (Last 7 Days)

```javascript
const weekAgo = new Date();
weekAgo.setDate(weekAgo.getDate() - 7);

const { data: sessions, error } = await supabase
    .from('sessions')
    .select('*')
    .eq('user_id', userId)
    .gte('started_at', weekAgo.toISOString())
    .order('started_at', { ascending: false });
```

---

## 5. Streak Calculation Logic

Streaks are calculated client-side from session data. Here's the algorithm:

```javascript
function calculateStreak(sessions, userTimezone = 'UTC') {
    if (!sessions || sessions.length === 0) {
        return { current: 0, longest: 0 };
    }

    // Get unique dates (in user's timezone)
    const dates = sessions.map(s => {
        const date = new Date(s.started_at);
        return date.toLocaleDateString('en-CA', { timeZone: userTimezone }); // YYYY-MM-DD
    });
    
    const uniqueDates = [...new Set(dates)].sort().reverse(); // Most recent first
    
    // Calculate current streak
    let currentStreak = 0;
    const today = new Date().toLocaleDateString('en-CA', { timeZone: userTimezone });
    const yesterday = new Date(Date.now() - 86400000).toLocaleDateString('en-CA', { timeZone: userTimezone });
    
    // Check if streak is active (plunged today or yesterday)
    if (uniqueDates[0] === today || uniqueDates[0] === yesterday) {
        currentStreak = 1;
        for (let i = 1; i < uniqueDates.length; i++) {
            const prevDate = new Date(uniqueDates[i - 1]);
            const currDate = new Date(uniqueDates[i]);
            const diffDays = (prevDate - currDate) / 86400000;
            
            if (diffDays === 1) {
                currentStreak++;
            } else {
                break;
            }
        }
    }
    
    // Calculate longest streak
    let longestStreak = 1;
    let tempStreak = 1;
    
    for (let i = 1; i < uniqueDates.length; i++) {
        const prevDate = new Date(uniqueDates[i - 1]);
        const currDate = new Date(uniqueDates[i]);
        const diffDays = (prevDate - currDate) / 86400000;
        
        if (diffDays === 1) {
            tempStreak++;
            longestStreak = Math.max(longestStreak, tempStreak);
        } else {
            tempStreak = 1;
        }
    }
    
    return { 
        current: currentStreak, 
        longest: Math.max(longestStreak, currentStreak) 
    };
}
```

---

## 6. Data Types Reference

### 6.1 Supabase/PostgreSQL Types Used

| Type | Description | Example |
|------|-------------|---------|
| `UUID` | 128-bit unique identifier | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11` |
| `TEXT` | Variable-length string | `"Austin, TX"` |
| `INTEGER` | 32-bit signed integer | `165` |
| `DECIMAL(p,s)` | Exact numeric with precision | `30.26720000` |
| `TIMESTAMPTZ` | Timestamp with timezone | `2026-02-02T14:30:00Z` |

### 6.2 JavaScript ↔ Database Mapping

| Database | JavaScript | Notes |
|----------|------------|-------|
| `UUID` | `string` | Use `crypto.randomUUID()` to generate |
| `TEXT` | `string` | - |
| `INTEGER` | `number` | - |
| `DECIMAL` | `number` | May lose precision for very large values |
| `TIMESTAMPTZ` | `Date` / `string` | Supabase returns ISO strings |

---

## 7. Future Schema Additions (V2+)

### 7.1 User Profile Extension

```sql
-- Add to users table for full profiles
ALTER TABLE users ADD COLUMN email TEXT UNIQUE;
ALTER TABLE users ADD COLUMN name TEXT;
ALTER TABLE users ADD COLUMN avatar_url TEXT;
ALTER TABLE users ADD COLUMN updated_at TIMESTAMPTZ;
```

### 7.2 Goals Table

```sql
CREATE TABLE goals (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    goal_type TEXT NOT NULL, -- 'streak', 'duration', 'temperature', 'sessions'
    target_value INTEGER NOT NULL,
    current_value INTEGER DEFAULT 0,
    achieved_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 7.3 Achievements/Badges Table

```sql
CREATE TABLE achievements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    achievement_type TEXT NOT NULL, -- 'first_plunge', '7_day_streak', 'ice_bath_club'
    achieved_at TIMESTAMPTZ DEFAULT NOW()
);
```

### 7.4 Social Sharing Tracking

```sql
-- Track shares for analytics
ALTER TABLE sessions ADD COLUMN shared_at TIMESTAMPTZ;
ALTER TABLE sessions ADD COLUMN share_platform TEXT; -- 'instagram', 'twitter', etc.
```

---

## 8. Database Maintenance

### 8.1 Indexes

Current indexes are sufficient for v1. Monitor query performance and add as needed:

```sql
-- If filtering by temperature becomes common
CREATE INDEX sessions_temperature_idx ON sessions(temperature_f);

-- If filtering by rating becomes common
CREATE INDEX sessions_rating_idx ON sessions(rating);

-- For location-based queries (v2+)
CREATE INDEX sessions_location_idx ON sessions(location_lat, location_lng);
```

### 8.2 Backups

Supabase provides automatic daily backups on paid plans. For free tier:
- Export data periodically via Supabase dashboard
- Consider implementing client-side export feature

### 8.3 Data Cleanup

For orphaned data or testing:

```sql
-- Delete sessions without valid user (shouldn't happen with FK)
DELETE FROM sessions WHERE user_id NOT IN (SELECT id FROM users);

-- Delete users with no sessions older than 30 days (cleanup inactive)
-- Use with caution!
DELETE FROM users 
WHERE id NOT IN (SELECT DISTINCT user_id FROM sessions)
AND created_at < NOW() - INTERVAL '30 days';
```

---

## 9. Setup Script (Complete)

Run this in Supabase SQL Editor to set up the entire schema:

```sql
-- =============================================
-- PLUNGE TRACKER v1.0 - DATABASE SETUP
-- =============================================

-- 1. Create users table
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    device_id TEXT UNIQUE NOT NULL,
    timezone TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. Create sessions table
CREATE TABLE IF NOT EXISTS sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    duration_seconds INTEGER NOT NULL CHECK (duration_seconds >= 0),
    temperature_f INTEGER NOT NULL CHECK (temperature_f >= 30 AND temperature_f <= 70),
    rating INTEGER CHECK (rating >= 1 AND rating <= 5),
    location_lat DECIMAL(10, 8),
    location_lng DECIMAL(11, 8),
    location_display TEXT,
    started_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Create indexes
CREATE INDEX IF NOT EXISTS users_device_id_idx ON users(device_id);
CREATE INDEX IF NOT EXISTS sessions_user_id_idx ON sessions(user_id);
CREATE INDEX IF NOT EXISTS sessions_started_at_idx ON sessions(started_at DESC);
CREATE INDEX IF NOT EXISTS sessions_user_started_idx ON sessions(user_id, started_at DESC);

-- 4. Enable Row Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE sessions ENABLE ROW LEVEL SECURITY;

-- 5. Create permissive policies for v1
CREATE POLICY "users_allow_all" ON users FOR ALL USING (true) WITH CHECK (true);
CREATE POLICY "sessions_allow_all" ON sessions FOR ALL USING (true) WITH CHECK (true);

-- =============================================
-- SETUP COMPLETE!
-- =============================================
```

---

## 10. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | Feb 2, 2026 | [Your Name] | Initial draft |

---

*This schema is designed to be simple for v1 and extensible for future versions.*
