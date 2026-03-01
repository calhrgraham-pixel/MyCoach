# MyCoach — AI Fitness Coach

A single-page web app that uses the Anthropic Claude API to generate personalized workout plans, and Supabase for user accounts, saving plans, and tracking workout sessions.

---

## Features

### Build Program
- Fill in your goal, fitness level, days per week, equipment, age, and any limitations
- Claude generates a complete, periodized workout plan in seconds
- View your weekly schedule with expandable day cards showing exercises, sets, reps, and rest times
- Includes a cardio protocol and tips for nutrition, recovery, progression, and supplements
- Save generated plans to your account to load or reference later

### Track Workouts
- Log workout sessions with a date picker
- Add exercises dynamically with sets, reps, weight, and unit (lbs/kg)
- Pre-populate a session from any day of a saved workout plan
- Automatically detects and celebrates new personal records (PRs) when you save
- Full session history with collapsible cards showing every set logged

### Accounts
- Email/password sign up and sign in
- Google OAuth sign in
- All data (plans, sessions) is private to each user via Supabase Row Level Security

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| AI | [Anthropic Claude API](https://www.anthropic.com) (`claude-sonnet-4-20250514`) |
| Auth | [Supabase Auth](https://supabase.com/auth) (email + Google OAuth) |
| Database | [Supabase](https://supabase.com) (PostgreSQL via PostgREST) |
| Fonts | Bebas Neue, DM Sans (Google Fonts) |

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/your-username/MyCoach.git
cd MyCoach
```

### 2. Create a Supabase project

1. Go to [supabase.com](https://supabase.com) and create a free project
2. In the SQL Editor, run the following to create the required tables:

```sql
-- Saved workout plans
CREATE TABLE workout_plans (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at timestamptz DEFAULT now(),
  plan_title text NOT NULL,
  plan_data jsonb NOT NULL,
  form_inputs jsonb
);

ALTER TABLE workout_plans ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_plans" ON workout_plans
  FOR ALL USING (auth.uid() = user_id);

-- Workout tracking
CREATE TABLE workout_sessions (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id uuid REFERENCES auth.users(id) ON DELETE CASCADE,
  session_date date DEFAULT current_date,
  created_at timestamptz DEFAULT now()
);

CREATE TABLE workout_entries (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  session_id uuid REFERENCES workout_sessions(id) ON DELETE CASCADE,
  exercise_name text NOT NULL,
  set_number int NOT NULL,
  reps int,
  weight numeric,
  weight_unit text DEFAULT 'lbs'
);

ALTER TABLE workout_sessions ENABLE ROW LEVEL SECURITY;
ALTER TABLE workout_entries  ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users_own_sessions" ON workout_sessions
  FOR ALL USING (auth.uid() = user_id);

CREATE POLICY "users_own_entries" ON workout_entries
  FOR ALL USING (
    session_id IN (SELECT id FROM workout_sessions WHERE user_id = auth.uid())
  );
```

3. (Optional) Enable Google OAuth in **Authentication → Providers → Google**

### 3. Add your credentials to `index.html`

Open `index.html` and fill in your Supabase project URL and anon key (found in **Settings → API**):

```js
const SUPABASE_URL      = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

You'll also need an [Anthropic API key](https://console.anthropic.com) — enter it directly in the app when generating a plan. It is never stored or sent anywhere except directly to Anthropic.

### 4. Serve the app

The app must be served over HTTP (not opened as a `file://` URL) for Supabase auth to work.

**Option A — VS Code Live Server extension:**
Right-click `index.html` → Open with Live Server

**Option B — Node.js:**
```bash
npx serve .
```

Then open `http://localhost:3000` (or the URL shown) in your browser.

---

## Project Structure

```
MyCoach/
└── index.html     # Entire app — HTML, CSS, and JS in one file
```

---

## Notes

- The Anthropic API key is entered by the user in the browser and sent directly to `api.anthropic.com`. It is never stored server-side.
- Supabase Row Level Security ensures each user can only access their own data.
- The app is entirely client-side with no build step required.
