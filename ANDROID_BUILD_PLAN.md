# MONOLITH — Android APK Build Plan

> **App**: Personal Performance Tracker (Habits, Workout, Calories, History, Stats)  
> **Current State**: Single HTML file with React (CDN), React Router, Tailwind CSS (CDN)  
> **Target**: Installable Android APK for personal use  
> **Stack Decision**: Capacitor + Vite + React  
> **Experience Level**: Intermediate  

---

## Why Capacitor?

Capacitor wraps your existing React/HTML app in a native Android shell. You keep almost all of your current code, and get access to native Android features (notifications, biometrics, camera, etc.) through simple plugin APIs. Alternatives considered:

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| Raw WebView APK | Fastest setup | No native features, no plugins | Too limited |
| Cordova/PhoneGap | Mature, lots of plugins | Outdated, slow builds | Skip |
| **Capacitor** | Modern, native plugins, keeps your React code | Requires Node/npm project setup | **USE THIS** |
| React Native | Most native feel | Full rewrite of all components | Overkill |
| PWA + TWA | Very clean | Needs a live domain/server | Complicated for personal use |

---

## Overview of All Phases

| Phase | Name | Effort | What You Get |
|---|---|---|---|
| 1 | Dev Environment | 1–2 hrs | Everything installed, ready to build |
| 2 | Project Migration | 2–3 hrs | HTML → proper Vite+React project |
| 3 | Data Layer | 3–5 hrs | All data persists across app restarts |
| 4 | Habit Management | 2–3 hrs | Add/edit/delete/reorder habits |
| 5 | Workout Features | 2–3 hrs | Real exercise tracking with history |
| 6 | Calorie Features | 2–3 hrs | Real meal logging with totals |
| 7 | History & Stats | 2 hrs | Charts and logs from real data |
| 8 | Capacitor Android | 2–3 hrs | First working APK on your phone |
| 9 | Push Notifications | 2 hrs | Daily reminders at custom times |
| 10 | Biometric Lock | 1–2 hrs | Fingerprint protection |
| 11 | Build & Sign APK | 1 hr | Distributable signed APK |
| 12 | Polish & QA | Ongoing | Bug fixes, UX improvements |

---

## Phase 1 — Development Environment Setup

### Goal
Install all tools needed to build and run the project.

### Prerequisites
- Windows 11 (you have this)
- Internet connection
- ~15 GB free disk space (Android Studio is large)

### Step-by-Step

#### 1.1 — Install Node.js
1. Go to https://nodejs.org and download the **LTS version** (e.g., 20.x)
2. Run the installer, accept all defaults
3. Verify in a new terminal:
   ```bash
   node --version   # Should print v20.x.x
   npm --version    # Should print 10.x.x
   ```

#### 1.2 — Install Android Studio
1. Go to https://developer.android.com/studio and download Android Studio
2. Install with defaults — **include the Android SDK** (checked by default)
3. On first launch, run the Setup Wizard, choose "Standard" install
4. It will download Android SDK, SDK Tools, etc. (~5 GB)
5. Once open, go to **SDK Manager** (top right gear icon):
   - SDK Platforms tab: Install **Android 14 (API 34)**
   - SDK Tools tab: Make sure **Android SDK Build-Tools**, **Android Emulator**, **Android SDK Platform-Tools** are all checked
6. Note your **Android SDK path** (shown at top of SDK Manager window, e.g. `C:\Users\YourName\AppData\Local\Android\Sdk`)

#### 1.3 — Set Environment Variables
1. Open Windows Settings → System → About → Advanced System Settings → Environment Variables
2. Under **System Variables**, click **New**:
   - Name: `ANDROID_HOME`
   - Value: your SDK path (e.g. `C:\Users\e430288.SPI-GLOBAL\AppData\Local\Android\Sdk`)
3. Find the `Path` variable → Edit → Add New:
   - `%ANDROID_HOME%\platform-tools`
   - `%ANDROID_HOME%\tools`
4. Close and reopen any terminals
5. Verify:
   ```bash
   adb --version  # Should print Android Debug Bridge version...
   ```

#### 1.4 — Install Java Development Kit (JDK)
Android Studio usually bundles a JDK, but you need it on PATH:
1. Go to **File → Settings → Build, Execution, Deployment → Build Tools → Gradle** in Android Studio
2. Note the "Gradle JDK" path (usually inside Android Studio's install folder)
3. Or download **JDK 17** from https://adoptium.net and install it
4. Add `JAVA_HOME` environment variable pointing to JDK folder

#### 1.5 — Set Up Android Device (for testing)
Option A — Physical phone (recommended):
1. On your Android phone: Settings → About Phone → tap "Build Number" 7 times
2. Settings → Developer Options → enable "USB Debugging"
3. Connect phone via USB, accept the RSA key prompt
4. Run `adb devices` — your phone should appear

Option B — Emulator:
1. In Android Studio → Device Manager → Create Virtual Device
2. Choose Pixel 7 → Android 14 → Finish
3. Start the emulator

### Expected Outcome
- `node`, `npm`, `adb` work in terminal
- Android Studio is open with an SDK installed
- A device shows up in `adb devices`

### Potential Issues
- **JDK not found**: Make sure `JAVA_HOME` points to JDK, not JRE
- **adb not found**: Path variables weren't set or terminal wasn't restarted
- **USB device not showing**: Try a different USB cable (data cable, not charge-only), enable MTP on phone
- **Hyper-V conflict with emulator**: Emulator requires Hyper-V on Windows; if disabled, enable it in Windows Features

---

## Phase 2 — Migrate HTML to a Vite + React Project

### Goal
Convert the single `monolith_performance_tracker.html` file into a proper npm project structure that Capacitor can wrap.

### Why Not Just Use the HTML File Directly?
- Capacitor needs a build output (a `/dist` folder), not a raw HTML file
- The CDN-based React (`unpkg.com`) and Babel transpilation in the browser is slow and fragile
- A proper Vite project gives you hot reloading, proper bundling, and future flexibility

### Step-by-Step

#### 2.1 — Create the Vite Project
Open a terminal in `C:\Users\e430288.SPI-GLOBAL\Desktop\Projects\`:

```bash
npm create vite@latest monolith-app -- --template react
cd monolith-app
npm install
```

This creates:
```
monolith-app/
├── index.html
├── package.json
├── vite.config.js
├── src/
│   ├── main.jsx
│   ├── App.jsx
│   └── index.css
└── public/
```

#### 2.2 — Install Dependencies
```bash
# React Router (same version your HTML uses, or upgrade to v6 latest)
npm install react-router-dom

# Tailwind CSS (proper build-time version, not CDN)
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

#### 2.3 — Configure Tailwind
Edit `tailwind.config.js`:
```js
/** @type {import('tailwindcss').Config} */
export default {
  darkMode: "class",
  content: ["./index.html", "./src/**/*.{js,jsx}"],
  theme: {
    extend: {
      colors: {
        // Paste ALL the colors from your HTML's tailwind.config here
        "tertiary-fixed-dim": "#454747",
        "secondary": "#c7c6c6",
        // ... (all the custom colors from the HTML file)
      },
      borderRadius: {
        "DEFAULT": "0.125rem",
        "lg": "0.25rem",
        "xl": "0.5rem",
        "full": "0.75rem"
      },
      fontFamily: {
        "headline": ["Space Grotesk"],
        "body": ["Inter"],
        "label": ["Inter"]
      }
    }
  },
  plugins: []
}
```

Edit `src/index.css` — replace everything with:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

body {
  background-color: #131313;
  color: #e5e2e1;
  min-height: 100dvh;
  overflow-x: hidden;
}

.material-symbols-outlined {
  font-variation-settings: 'FILL' 0, 'wght' 400, 'GRAD' 0, 'opsz' 24;
}

.no-scrollbar::-webkit-scrollbar { display: none; }
```

#### 2.4 — Update index.html
Replace the content of `index.html` with:
```html
<!DOCTYPE html>
<html class="dark" lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Monolith</title>
  <!-- Fonts (CDN — requires internet, which is fine per your choice) -->
  <link href="https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Inter:ital,wght@0,300;0,400;0,500;0,600;0,700;1,400&display=swap" rel="stylesheet"/>
  <link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:wght,FILL@100..700,0..1&display=swap" rel="stylesheet"/>
</head>
<body class="font-body antialiased">
  <div id="root"></div>
  <script type="module" src="/src/main.jsx"></script>
</body>
</html>
```

#### 2.5 — Split HTML into Component Files
Create the following file structure in `src/`:
```
src/
├── main.jsx
├── App.jsx
├── index.css
├── components/
│   ├── TopAppBar.jsx
│   └── BottomNavBar.jsx
└── screens/
    ├── Home.jsx
    ├── Workout.jsx
    ├── Calories.jsx
    ├── History.jsx
    └── Stats.jsx
```

Move the code from the HTML `<script type="text/babel">` block into these files. Remove all `const { useState } = React;` style imports — replace with proper ES module imports:
```jsx
// Before (CDN style):
const { useState } = React;

// After (proper module):
import { useState } from 'react';
```

**`src/main.jsx`**:
```jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import App from './App.jsx'
import './index.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
)
```

**`src/App.jsx`**:
```jsx
import { MemoryRouter, Routes, Route } from 'react-router-dom'
import TopAppBar from './components/TopAppBar'
import BottomNavBar from './components/BottomNavBar'
import Home from './screens/Home'
import Workout from './screens/Workout'
import Calories from './screens/Calories'
import History from './screens/History'
import Stats from './screens/Stats'

export default function App() {
  return (
    <MemoryRouter>
      <TopAppBar />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/workout" element={<Workout />} />
        <Route path="/calories" element={<Calories />} />
        <Route path="/history" element={<History />} />
        <Route path="/stats" element={<Stats />} />
      </Routes>
      <BottomNavBar />
    </MemoryRouter>
  )
}
```

Copy each screen's JSX from the HTML into its corresponding file.

#### 2.6 — Test in Browser
```bash
npm run dev
```
Open `http://localhost:5173` — you should see your app running exactly as before.

#### 2.7 — Build for Production
```bash
npm run build
```
This creates a `/dist` folder — this is what Capacitor will bundle into the APK.

### Expected Outcome
- Running app in browser via `npm run dev`
- `npm run build` produces a `/dist` folder with no errors
- All 5 screens visible and navigable

### Potential Issues
- **JSX syntax errors**: The HTML Babel version is lenient; proper JSX is strict. Watch for: className attribute (already correct), self-closing tags `<br />`, `<input />`, etc.
- **Missing imports**: Every component must import React and hooks explicitly
- **Tailwind classes not working**: Check that `tailwind.config.js` content paths match your file locations
- **Custom colors not applied**: Make sure you copied ALL colors from the HTML's tailwind config

---

## Phase 3 — Data Layer: Making Everything Actually Work

### Goal
Replace all hardcoded data with real state that persists between app sessions. Currently, the app resets every time you close it.

### Architecture Decision
Use **React Context + localStorage** — simple, no extra dependencies, works perfectly for a personal app with this data volume.

### What Needs to Persist
| Data | Where | Format |
|---|---|---|
| Habit definitions (name, pts, desc) | localStorage | JSON array |
| Daily habit completions | localStorage | `{ "2024-10-24": { habitId: true/false } }` |
| Workout log entries | localStorage | Array of `{ date, exercise, sets, reps, weight }` |
| Meal log entries | localStorage | Array of `{ date, name, calories, time }` |
| Daily notes | localStorage | `{ "2024-10-24": "note text" }` |
| Daily score history | Computed | Derived from completions + habit points |
| Streak | Computed | Derived from score history |

### Step-by-Step

#### 3.1 — Create the App Context
Create `src/context/AppContext.jsx`:

```jsx
import { createContext, useContext, useState, useEffect } from 'react'

const AppContext = createContext(null)

// Helper to read/write localStorage safely
function useLocalStorage(key, defaultValue) {
  const [state, setState] = useState(() => {
    try {
      const stored = localStorage.getItem(key)
      return stored ? JSON.parse(stored) : defaultValue
    } catch { return defaultValue }
  })
  
  const set = (value) => {
    setState(value)
    localStorage.setItem(key, JSON.stringify(value))
  }
  
  return [state, set]
}

// Get today's date as "YYYY-MM-DD"
export function todayKey() {
  return new Date().toISOString().split('T')[0]
}

export function AppProvider({ children }) {
  // Default habits matching your current design
  const defaultHabits = [
    { id: 'wake', name: 'Wake up 7AM', desc: 'Biological clock synchronization', pts: 20 },
    { id: 'nonap', name: 'No nap', desc: 'Maintaining daytime alertness', pts: 10 },
    { id: 'workout', name: 'Workout done', desc: 'Heavy lifting or high intensity', pts: 25 },
    { id: 'diet', name: 'Diet followed', desc: 'Macros within precision range', pts: 15 },
    { id: 'cutoff', name: 'Phone cutoff 11:30PM', desc: 'Zero blue light exposure', pts: 10 },
    { id: 'hustle', name: 'Side hustle 1hr', desc: 'Uninterrupted deep work session', pts: 30 },
    { id: 'noscroll', name: 'No late scrolling', desc: 'Preserving mental dopamine levels', pts: 10 },
  ]

  const [habits, setHabits] = useLocalStorage('monolith_habits', defaultHabits)
  const [completions, setCompletions] = useLocalStorage('monolith_completions', {})
  const [workoutLog, setWorkoutLog] = useLocalStorage('monolith_workouts', [])
  const [mealLog, setMealLog] = useLocalStorage('monolith_meals', [])
  const [notes, setNotes] = useLocalStorage('monolith_notes', {})

  // Toggle a habit for today
  const toggleHabit = (habitId) => {
    const today = todayKey()
    const todayCompletions = completions[today] || {}
    setCompletions({
      ...completions,
      [today]: {
        ...todayCompletions,
        [habitId]: !todayCompletions[habitId]
      }
    })
  }

  // Calculate daily score (0–100) for a given date
  const getDayScore = (dateKey) => {
    const dayCompletions = completions[dateKey] || {}
    const maxPts = habits.reduce((sum, h) => sum + h.pts, 0)
    if (maxPts === 0) return 0
    const earnedPts = habits.reduce((sum, h) => {
      return sum + (dayCompletions[h.id] ? h.pts : 0)
    }, 0)
    return Math.round((earnedPts / maxPts) * 100)
  }

  // Calculate current streak (consecutive days with score >= 50)
  const getStreak = () => {
    let streak = 0
    let date = new Date()
    date.setDate(date.getDate() - 1) // Start from yesterday
    while (true) {
      const key = date.toISOString().split('T')[0]
      if (getDayScore(key) >= 50) {
        streak++
        date.setDate(date.getDate() - 1)
      } else break
      if (streak > 365) break // Safety limit
    }
    return streak
  }

  // Add a workout entry
  const addWorkout = (exercise, sets, reps, weight) => {
    const entry = {
      id: Date.now(),
      date: todayKey(),
      exercise,
      sets: Number(sets),
      reps: Number(reps),
      weight: Number(weight),
    }
    setWorkoutLog([...workoutLog, entry])
  }

  // Delete a workout entry
  const deleteWorkout = (id) => {
    setWorkoutLog(workoutLog.filter(w => w.id !== id))
  }

  // Add a meal entry
  const addMeal = (name, calories) => {
    const entry = {
      id: Date.now(),
      date: todayKey(),
      name,
      calories: Number(calories),
      time: new Date().toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' }),
    }
    setMealLog([...mealLog, entry])
  }

  // Delete a meal entry
  const deleteMeal = (id) => {
    setMealLog(mealLog.filter(m => m.id !== id))
  }

  // Save/update a daily note
  const saveNote = (dateKey, text) => {
    setNotes({ ...notes, [dateKey]: text })
  }

  // Add a new habit
  const addHabit = (name, desc, pts) => {
    const newHabit = {
      id: `habit_${Date.now()}`,
      name, desc,
      pts: Number(pts)
    }
    setHabits([...habits, newHabit])
  }

  // Edit a habit
  const editHabit = (id, updates) => {
    setHabits(habits.map(h => h.id === id ? { ...h, ...updates } : h))
  }

  // Delete a habit
  const deleteHabit = (id) => {
    setHabits(habits.filter(h => h.id !== id))
  }

  // Reorder habits (for drag-and-drop later)
  const reorderHabits = (newOrder) => {
    setHabits(newOrder)
  }

  return (
    <AppContext.Provider value={{
      habits, completions, workoutLog, mealLog, notes,
      toggleHabit, getDayScore, getStreak,
      addWorkout, deleteWorkout,
      addMeal, deleteMeal,
      saveNote,
      addHabit, editHabit, deleteHabit, reorderHabits,
      todayKey,
    }}>
      {children}
    </AppContext.Provider>
  )
}

export const useApp = () => useContext(AppContext)
```

#### 3.2 — Wrap App with Provider
In `src/main.jsx`:
```jsx
import { AppProvider } from './context/AppContext'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <AppProvider>
      <App />
    </AppProvider>
  </React.StrictMode>
)
```

#### 3.3 — Connect Home Screen to Real Data
In `src/screens/Home.jsx`:
```jsx
import { useApp, todayKey } from '../context/AppContext'

export default function Home() {
  const { habits, completions, toggleHabit, getDayScore, getStreak, notes, saveNote } = useApp()
  const today = todayKey()
  const todayCompletions = completions[today] || {}
  const score = getDayScore(today)
  const maxPts = habits.reduce((sum, h) => sum + h.pts, 0)
  const earnedPts = habits.reduce((sum, h) => todayCompletions[h.id] ? sum + h.pts : sum, 0)
  const [note, setNote] = useState(notes[today] || "")
  
  // Calculate stroke offset for the circular progress
  const circumference = 2 * Math.PI * 88  // r=88
  const strokeDashoffset = circumference - (score / 100) * circumference

  return (
    // ... your existing JSX, but now using real data:
    // score, getStreak(), habits.map(...), toggleHabit(habit.id)
    // earnedPts / maxPts for the progress circle
  )
}
```

Replace the hardcoded habit array with `habits.map(habit => ...)` and call `toggleHabit(habit.id)` on click.

#### 3.4 — Connect All Other Screens
Repeat the same pattern for:
- `Workout.jsx` — use `workoutLog`, `addWorkout`, `deleteWorkout`
- `Calories.jsx` — use `mealLog`, `addMeal`, `deleteMeal`
- `History.jsx` — compute from `completions`, `notes`, `getDayScore`
- `Stats.jsx` — compute from `completions`, `workoutLog`, `mealLog`

### Expected Outcome
- Checking a habit stays checked after navigating away
- Checking a habit stays checked after closing and reopening the app
- Daily score updates in real time as you check habits
- Notes save and reload correctly

### Potential Issues
- **Score always 0**: Check that `habit.id` in completions matches `habit.id` in the habits array
- **Data lost on refresh**: Make sure `localStorage.setItem` is being called in the setter
- **Circular import**: Ensure `AppContext.jsx` doesn't import anything from screens
- **localStorage limit**: localStorage is ~5MB — more than enough for years of daily data

---

## Phase 4 — Habit Management Screen

### Goal
Let the user add, edit, delete, and reorder habits without touching code.

### New Screen: `/habits`

Create `src/screens/HabitManager.jsx` with:

#### UI Components Needed
1. **Habit list** — each row shows name, points, with Edit and Delete icons
2. **Add habit form** — name, description, points fields + Save button
3. **Edit habit modal** — inline or overlay form that pre-fills existing values
4. **Reorder** — Up/Down arrow buttons on each row (drag-and-drop is possible but complex; arrows are simpler)
5. **Total points display** — show current max possible score

#### Key Logic
```jsx
// Add new habit
const [name, setName] = useState("")
const [desc, setDesc] = useState("")
const [pts, setPts] = useState(10)

const handleAdd = () => {
  if (!name.trim()) return
  addHabit(name.trim(), desc.trim(), pts)
  setName(""); setDesc(""); setPts(10)
}

// Move habit up in list
const moveUp = (index) => {
  if (index === 0) return
  const newHabits = [...habits]
  ;[newHabits[index - 1], newHabits[index]] = [newHabits[index], newHabits[index - 1]]
  reorderHabits(newHabits)
}
```

#### Add to Navigation
Add the Habit Manager to the nav bar or as a settings option accessible from the TopAppBar menu icon.

### Features of This Screen
- List of all habits with: name, description, point value
- Inline edit: tap pencil icon → fields become editable inline → tap checkmark to save
- Delete with confirmation prompt (to prevent accidental deletion)
- Reorder with up/down buttons
- "Reset to defaults" button to restore original 7 habits
- Running total: "Max daily score: 120 pts"

### Potential Issues
- **Deleting a habit loses its completion history**: The history data uses habit IDs. If you delete a habit, its completions remain orphaned in localStorage but won't show. This is acceptable for personal use — you can note this as known behavior.
- **Point total > 100**: Your scoring is percentage-based, so total points can be anything — the score stays 0–100. Make this clear in the UI.

---

## Phase 5 — Workout Screen: Full Implementation

### Goal
Full exercise logging with history, PR detection, and today's workout type.

### Current State
- Form exists but doesn't save anything
- Logged activities are hardcoded

### Enhancements Needed

#### 5.1 — "Today's Mission" (workout type)
Add a workout type selector: Gym / Swim / Rest / Cardio / Other  
Store in `localStorage` per day: `{ "2024-10-24": "GYM + SWIM" }`

#### 5.2 — Exercise Logging Form
Wire up the existing form to `addWorkout()`:
```jsx
const [exercise, setExercise] = useState("")
const [sets, setSets] = useState("")
const [reps, setReps] = useState("")
const [weight, setWeight] = useState("")

const handleSave = () => {
  if (!exercise || !sets || !reps) return
  addWorkout(exercise, sets, reps, weight || 0)
  setExercise(""); setSets(""); setReps(""); setWeight("")
}
```

#### 5.3 — PR Detection
When adding a workout, compare against the best previous performance for the same exercise:
```jsx
const checkPR = (exercise, weight, reps) => {
  const previous = workoutLog
    .filter(w => w.exercise.toLowerCase() === exercise.toLowerCase() && w.date !== todayKey())
    .sort((a, b) => b.date.localeCompare(a.date))
  if (!previous.length) return false
  const bestWeight = Math.max(...previous.map(w => w.weight))
  return Number(weight) > bestWeight
}
```

#### 5.4 — Previous Week Comparison
For each exercise logged today, show last week's performance:
```jsx
const getLastPerformance = (exerciseName) => {
  const past = workoutLog
    .filter(w => w.exercise === exerciseName && w.date !== todayKey())
    .sort((a, b) => b.date.localeCompare(a.date))
  return past[0] || null  // Most recent previous entry
}
```

#### 5.5 — Workout Completion Toggle
The "66% COMPLETE" bar on the workout screen — this can be derived from:
- User sets a target number of exercises for the day OR
- Simply: `(exercises logged today) / (typical average)`

Simplest approach: Let user tap a "Mark Workout Complete" button that sets a flag for the day.

#### 5.6 — Delete Exercise Entry
Add a delete button (swipe or long-press alternative: tap the `more_vert` icon → show delete option).

### Potential Issues
- **Exercise name matching for PR detection**: "bench press" vs "Bench Press" vs "bench" will all be treated as different. Add `.toLowerCase().trim()` normalization.
- **Weight units**: Currently using KG. Add a toggle in settings for KG/LBS if needed (store preference in localStorage).

---

## Phase 6 — Calorie Screen: Full Implementation

### Goal
Accurate daily calorie tracking with weekly visualization.

### Enhancements Needed

#### 6.1 — Calorie Goal Setting
Store a user-defined calorie goal in localStorage (default: 2500).  
Add a simple "Edit Goal" button that opens an input field.

#### 6.2 — Wire Up Meal Logging Form
```jsx
const [mealName, setMealName] = useState("")
const [calories, setCalories] = useState("")

const handleAddMeal = () => {
  if (!mealName || !calories) return
  addMeal(mealName, Number(calories))
  setMealName(""); setCalories("")
}
```

#### 6.3 — Today's Total
```jsx
const today = todayKey()
const todayMeals = mealLog.filter(m => m.date === today)
const todayCalories = todayMeals.reduce((sum, m) => sum + m.calories, 0)
const calorieGoal = Number(localStorage.getItem('monolith_calorie_goal') || 2500)
const remaining = calorieGoal - todayCalories
const progressPct = Math.min((todayCalories / calorieGoal) * 100, 100)
```

#### 6.4 — Weekly Bar Chart
Calculate totals for the last 7 days:
```jsx
const getLast7Days = () => {
  return Array.from({ length: 7 }, (_, i) => {
    const d = new Date()
    d.setDate(d.getDate() - (6 - i))
    const key = d.toISOString().split('T')[0]
    const total = mealLog
      .filter(m => m.date === key)
      .reduce((sum, m) => sum + m.calories, 0)
    return {
      day: ['M','T','W','T','F','S','S'][d.getDay() === 0 ? 6 : d.getDay() - 1],
      total,
      pct: Math.min((total / calorieGoal) * 100, 100),
      isToday: key === todayKey(),
      overGoal: total > calorieGoal,
    }
  })
}
```

#### 6.5 — Delete Meal Entry
Swipe to delete (complex) OR tap meal → show a delete button/icon.

#### 6.6 — Macro Tracking (Optional Enhancement)
Extend meal logging to include: Protein (g), Carbs (g), Fat (g).  
Show macro breakdown: "P: 180g | C: 220g | F: 65g"  
This requires adding 3 more fields to the log meal form.

### Potential Issues
- **Overeating visualization**: When calories exceed goal, the progress bar should change color to red. The weekly chart already handles this with `color: 'bg-error'` for overages.
- **Time zones**: `new Date().toISOString()` uses UTC. If you're in a timezone like GMT+5:30, midnight UTC could be a different calendar day. Fix: use `new Date().toLocaleDateString('en-CA')` to get local date.

---

## Phase 7 — History & Stats: Real Data

### Goal
History screen shows real past days from actual data. Stats screen shows real analytics.

### History Screen

#### Real Day Generation
```jsx
// Generate last 30 days of history from real data
const getLast30Days = () => {
  return Array.from({ length: 30 }, (_, i) => {
    const d = new Date()
    d.setDate(d.getDate() - i)
    const key = d.toISOString().split('T')[0]
    const score = getDayScore(key)
    const dayCompletions = completions[key] || {}
    const completedHabits = habits.filter(h => dayCompletions[h.id])
    return {
      key,
      date: d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' }).toUpperCase(),
      score,
      completedHabits,
      note: notes[key] || "",
      workouts: workoutLog.filter(w => w.date === key),
      meals: mealLog.filter(m => m.date === key),
    }
  })
}
```

#### Expandable Day Cards
The current expand/collapse behavior already works — just replace with real data from `getLast30Days()`.

### Stats Screen

#### Real Calculations
```jsx
const last30 = getLast30Days()
const scores = last30.map(d => d.score).filter(s => s > 0)  // Only days with any data

const avgScore = scores.length 
  ? Math.round(scores.reduce((a, b) => a + b, 0) / scores.length)
  : 0

const bestDay = scores.length ? Math.max(...scores) : 0
const worstDay = scores.length ? Math.min(...scores) : 0

// Habit efficiency (% of days each habit was completed, in last 30)
const habitEfficiency = habits.map(habit => {
  const completed = last30.filter(d => (completions[d.key] || {})[habit.id]).length
  return {
    name: habit.name,
    pct: Math.round((completed / 30) * 100)
  }
}).sort((a, b) => b.pct - a.pct)
```

#### Add More Stat Visualizations
- **30-day score trend**: Line of bars (already have this UI)
- **Best streak**: Track longest consecutive streak in history
- **Total workouts**: Count all workout entries
- **Calorie accuracy**: % of days within 10% of goal
- **Most consistent habit**: Habit with highest 30-day completion %
- **Most missed habit**: Habit with lowest 30-day completion %

### Potential Issues
- **No data yet**: Stats will show all zeros/empty until you use the app for a few days. Add an empty state message: "Start tracking to see your performance trends."
- **Performance**: Calculating stats over 365 days across a large completion object could be slow. For 30 days it's instant. If you extend to 1-year views, consider memoizing with `useMemo`.

---

## Phase 8 — Capacitor Android Integration

### Goal
Wrap the working Vite+React app inside a native Android shell. Get a working APK installed on your phone.

### Step-by-Step

#### 8.1 — Install Capacitor
In your `monolith-app` folder:
```bash
npm install @capacitor/core @capacitor/cli
npx cap init
```

When prompted:
- App name: `Monolith`
- App ID: `com.yourname.monolith` (reverse domain, e.g. `com.john.monolith`)
- Web dir: `dist`

This creates `capacitor.config.ts`.

#### 8.2 — Add Android Platform
```bash
npm install @capacitor/android
npx cap add android
```

This creates an `android/` folder — a real Android Studio project.

#### 8.3 — Build + Sync
Every time you make code changes, you need to:
```bash
npm run build          # Build the web app to /dist
npx cap sync android   # Copy /dist into android project + sync plugins
```

Shorthand (add to `package.json` scripts):
```json
"scripts": {
  "android": "npm run build && npx cap sync android && npx cap open android"
}
```

Then just run: `npm run android`

#### 8.4 — Open in Android Studio
```bash
npx cap open android
```

Android Studio opens with the `android/` project.

#### 8.5 — Run on Device
In Android Studio:
1. Select your device in the top toolbar dropdown
2. Press the green **Run** button (▶)
3. The app will build, install, and launch on your phone

First build takes 3–5 minutes. Subsequent builds are faster.

#### 8.6 — Configure App Icon
1. Create a 1024×1024 PNG of your app icon
2. In Android Studio: right-click `app/res` → New → Image Asset
3. Choose "Launcher Icons" → upload your PNG
4. This generates all the required icon sizes automatically

#### 8.7 — Configure App Colors / Theme
In `android/app/src/main/res/values/styles.xml`:
```xml
<style name="AppTheme" parent="Theme.AppCompat.NoActionBar">
  <item name="android:windowBackground">@color/black</item>
  <item name="android:statusBarColor">#131313</item>
  <item name="android:navigationBarColor">#131313</item>
</style>
```

This makes the status bar and navigation bar match your dark theme.

#### 8.8 — Handle Status Bar Overlap
In `capacitor.config.ts`:
```typescript
import { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.yourname.monolith',
  appName: 'Monolith',
  webDir: 'dist',
  plugins: {
    StatusBar: {
      style: 'dark',
      backgroundColor: '#131313',
      overlaysWebView: false,
    },
  },
};

export default config;
```

Install Status Bar plugin:
```bash
npm install @capacitor/status-bar
npx cap sync android
```

#### 8.9 — Handle Android Back Button
On Android, the hardware/gesture back button should navigate back. In `src/App.jsx`:
```jsx
import { App as CapApp } from '@capacitor/app'
import { useNavigate } from 'react-router-dom'
import { useEffect } from 'react'

// Inside your App component:
const navigate = useNavigate()
useEffect(() => {
  const backHandler = CapApp.addListener('backButton', () => {
    navigate(-1)
  })
  return () => backHandler.remove()
}, [])
```

Install:
```bash
npm install @capacitor/app
npx cap sync android
```

#### 8.10 — Enable Haptic Feedback (optional but nice)
```bash
npm install @capacitor/haptics
```

On habit toggle:
```jsx
import { Haptics, ImpactStyle } from '@capacitor/haptics'
const toggleWithHaptics = async (habitId) => {
  await Haptics.impact({ style: ImpactStyle.Light })
  toggleHabit(habitId)
}
```

### Expected Outcome
- App installs on your phone
- All 5 screens work correctly
- Data persists between app closes
- Back button works naturally
- Dark status/nav bars match the app theme

### Potential Issues
- **"INSTALL_FAILED_UPDATE_INCOMPATIBLE"**: Uninstall any previous version of the app from your phone first
- **App crashes immediately**: Check Android Studio's Logcat for the error. Usually a missing permission or failed plugin init.
- **White flash on startup**: The WebView loads a moment after the native shell. Fix: set a matching splash screen background color in `styles.xml`
- **Fonts look different**: System fonts may render differently on Android than in your browser. Test on device early.
- **Keyboard pushes content**: Add `android:windowSoftInputMode="adjustResize"` to the Activity in `AndroidManifest.xml`
- **HTTP requests blocked**: Android blocks plain HTTP. If any API calls use `http://`, switch to `https://` or add `usesCleartextTraffic="true"` to manifest (not recommended for production).

---

## Phase 9 — Push Notifications & Daily Reminders

### Goal
Receive a notification at custom times reminding you to check your habits, log your workout, etc.

### Architecture
Use **@capacitor/local-notifications** — notifications fire from the device itself, no server needed.

### Step-by-Step

#### 9.1 — Install Plugin
```bash
npm install @capacitor/local-notifications
npx cap sync android
```

#### 9.2 — Add Permissions to AndroidManifest.xml
In `android/app/src/main/AndroidManifest.xml`, add inside `<manifest>`:
```xml
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"/>
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
```

#### 9.3 — Create Notification Service
Create `src/services/notifications.js`:
```js
import { LocalNotifications } from '@capacitor/local-notifications'

export async function requestNotificationPermission() {
  const { display } = await LocalNotifications.requestPermissions()
  return display === 'granted'
}

export async function scheduleHabitReminder(hour, minute, label = 'MONOLITH') {
  await LocalNotifications.cancel({ notifications: [{ id: 1 }] }) // Cancel old
  
  await LocalNotifications.schedule({
    notifications: [{
      id: 1,
      title: 'MONOLITH',
      body: `Daily discipline check-in time. ${label}`,
      schedule: {
        on: { hour, minute },
        allowWhileIdle: true,
      },
      sound: null,
      smallIcon: 'ic_stat_monolith', // Create this icon in Android Studio
    }]
  })
}

export async function scheduleWorkoutReminder(hour, minute) {
  await LocalNotifications.schedule({
    notifications: [{
      id: 2,
      title: 'WORKOUT',
      body: 'You have not logged your workout today.',
      schedule: { on: { hour, minute }, allowWhileIdle: true },
    }]
  })
}

export async function cancelAllReminders() {
  const pending = await LocalNotifications.getPending()
  if (pending.notifications.length) {
    await LocalNotifications.cancel({ notifications: pending.notifications })
  }
}
```

#### 9.4 — Create Settings Screen
Add `src/screens/Settings.jsx` with:
- **Habit Reminder**: Toggle on/off, time picker (hour + minute)
- **Workout Reminder**: Toggle on/off, time picker
- **Meal Reminder**: Toggle on/off, time picker
- **Calorie Goal**: Number input
- **Reset All Data**: Danger zone button with confirmation

#### 9.5 — Time Picker (Android-Style)
For simplicity, use two number selects (hour: 0–23, minute: 0–59).  
Or install a React time picker: `npm install rc-time-picker`

#### 9.6 — Re-schedule on App Launch
Notifications are canceled if the user clears app data. Re-schedule on every app open:
```jsx
// In App.jsx useEffect:
useEffect(() => {
  const savedReminderHour = localStorage.getItem('reminder_hour')
  if (savedReminderHour) {
    scheduleHabitReminder(
      Number(savedReminderHour),
      Number(localStorage.getItem('reminder_minute') || 0)
    )
  }
}, [])
```

### Potential Issues
- **Notification not firing on newer Android (12+)**: Exact alarms require explicit permission on Android 12+. Add `SCHEDULE_EXACT_ALARM` permission and check for it at runtime.
- **Notifications stop after reboot**: Add a `RECEIVE_BOOT_COMPLETED` broadcast receiver in the Android project (Capacitor handles this if configured correctly).
- **Notification icon is white box**: You need to create a proper monochrome small icon. In Android Studio: right-click res → New → Image Asset → Notification Icons.
- **Testing notifications**: Manually trigger with a 5-second delay first, then switch to daily schedule.

---

## Phase 10 — Biometric / PIN Lock

### Goal
Protect the app with a fingerprint prompt (or PIN fallback) on every open.

### Step-by-Step

#### 10.1 — Install Plugin
```bash
npm install capacitor-biometric-auth
# or the newer:
npm install @aparajita/capacitor-biometric-auth
npx cap sync android
```

#### 10.2 — Add Permission to AndroidManifest.xml
```xml
<uses-permission android:name="android.permission.USE_BIOMETRIC"/>
<uses-permission android:name="android.permission.USE_FINGERPRINT"/>
```

#### 10.3 — Create Lock Screen Component
Create `src/components/LockScreen.jsx`:
```jsx
import { useState, useEffect } from 'react'
import { BiometricAuth } from '@aparajita/capacitor-biometric-auth'

export default function LockScreen({ onUnlock }) {
  const [error, setError] = useState("")
  
  const authenticate = async () => {
    try {
      await BiometricAuth.authenticate({
        reason: 'Access Monolith',
        cancelTitle: 'Cancel',
        allowDeviceCredential: true, // Fallback to PIN/pattern
      })
      onUnlock()
    } catch (e) {
      setError("Authentication failed. Try again.")
    }
  }
  
  useEffect(() => {
    authenticate()
  }, [])
  
  return (
    <div className="fixed inset-0 bg-background flex flex-col items-center justify-center gap-8 z-[100]">
      <h1 className="font-headline text-4xl font-bold text-white tracking-widest">MONOLITH</h1>
      <button 
        onClick={authenticate}
        className="flex flex-col items-center gap-3 text-secondary hover:text-white transition-colors"
      >
        <span className="material-symbols-outlined text-6xl">fingerprint</span>
        <span className="font-label text-xs tracking-widest uppercase">Tap to unlock</span>
      </button>
      {error && <p className="text-error text-sm font-body">{error}</p>}
    </div>
  )
}
```

#### 10.4 — Add Lock to App
In `src/App.jsx`:
```jsx
const [isLocked, setIsLocked] = useState(true)
const biometricEnabled = localStorage.getItem('biometric_enabled') === 'true'

// On app focus (returning from background), re-lock
useEffect(() => {
  const listener = CapApp.addListener('appStateChange', ({ isActive }) => {
    if (!isActive) setIsLocked(true)
  })
  return () => listener.remove()
}, [])

return (
  <>
    {biometricEnabled && isLocked && <LockScreen onUnlock={() => setIsLocked(false)} />}
    <MemoryRouter>
      {/* rest of app */}
    </MemoryRouter>
  </>
)
```

#### 10.5 — Add Toggle in Settings
In Settings screen:
```jsx
const [biometricEnabled, setBiometricEnabled] = useState(
  localStorage.getItem('biometric_enabled') === 'true'
)

const toggleBiometric = async () => {
  const available = await BiometricAuth.checkBiometry()
  if (!available.isAvailable && !biometricEnabled) {
    alert("No biometric sensor found on this device.")
    return
  }
  const newVal = !biometricEnabled
  setBiometricEnabled(newVal)
  localStorage.setItem('biometric_enabled', String(newVal))
}
```

### Potential Issues
- **Device has no fingerprint sensor**: `allowDeviceCredential: true` falls back to PIN/pattern. If no screen lock is set at all, the auth will fail. Handle gracefully.
- **Biometric plugin not available in browser**: Wrap all biometric calls in `try/catch` and skip in web mode.
- **App locked on first install**: First launch, don't show lock screen — only lock after first time the user exits and returns.

---

## Phase 11 — Build, Sign & Install APK

### Goal
Create a distributable `.apk` file you can sideload onto any Android device.

### Why Signing?
Android requires all apps to be digitally signed. For personal use, a self-signed certificate is fine (you don't need to pay Google Play $25).

### Step-by-Step

#### 11.1 — Generate a Keystore (one-time)
In Android Studio terminal (or any terminal with Java on PATH):
```bash
keytool -genkey -v \
  -keystore monolith-release-key.jks \
  -alias monolith \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

You'll be prompted for:
- Keystore password (remember this!)
- Key password
- Your name, organization, location (can all be fake for personal use)

**IMPORTANT**: Store `monolith-release-key.jks` and your passwords somewhere safe. If you lose these, you cannot update the app — you'd have to uninstall and reinstall.

#### 11.2 — Configure Signing in Android Studio
1. In Android Studio: Build → Generate Signed Bundle/APK
2. Choose **APK** (not Bundle — Bundle is for Play Store)
3. Browse to your `.jks` keystore file
4. Enter passwords
5. Choose **release** build variant
6. Click **Create**

#### 11.3 — Find the APK
The APK will be at:
```
android/app/release/app-release.apk
```

#### 11.4 — Install on Your Phone
Option A — USB:
```bash
adb install android/app/release/app-release.apk
```

Option B — Transfer the file:
1. Copy the `.apk` to your phone via USB/Google Drive/email
2. On your phone: Settings → Security → "Install from Unknown Sources" (or "Install Unknown Apps")
3. Open Files app → tap the APK → Install

#### 11.5 — Automate the Build
Add to `package.json`:
```json
"scripts": {
  "build:android": "npm run build && npx cap sync android",
  "release": "npm run build:android && cd android && ./gradlew assembleRelease"
}
```

Run `npm run release` to build a release APK without opening Android Studio.

### APK Size Optimization
- Enable R8 minification in `android/app/build.gradle`
- Enable Proguard: `minifyEnabled true` in release build type
- Expected final APK size: **3–8 MB** (most of it is the WebView wrapper)

### Potential Issues
- **"App not installed"**: APK is corrupted or signing failed. Rebuild.
- **App crashes on launch**: Check Logcat. Usually a missing manifest permission.
- **"Parse error"**: APK targetSdkVersion doesn't match Android version. Update `targetSdkVersion` in `build.gradle` to 34.
- **Outdated APK won't install over new**: You must uninstall the old version first if the keystore changed. With the same keystore, updates install over each other.

---

## Phase 12 — Polish & Quality Assurance

### Performance
- [ ] Add `useMemo` to expensive calculations (stats, 30-day history)
- [ ] Lazy-load the Stats and History screens (React.lazy + Suspense)
- [ ] Add loading states to data-dependent screens

### UX Improvements
- [ ] Splash screen matching the dark theme (no white flash)
- [ ] Animations: habit checkbox toggle, score circle update
- [ ] Pull-to-refresh gesture on History screen
- [ ] Swipe-to-delete on meal/workout entries (react-swipeable)
- [ ] Keyboard avoidance: ensure form inputs aren't hidden by Android keyboard
- [ ] Long-press on habit in History to copy note text

### Error States
- [ ] Empty state on History when no data exists yet
- [ ] Empty state on Stats ("Start tracking to see trends")
- [ ] Validation on all forms (no empty names, no negative points)
- [ ] Graceful error if localStorage is full (extremely rare)

### Data Management
- [ ] Export all data as JSON (for backup)
- [ ] Import JSON backup
- [ ] Clear all data option (with "Type CONFIRM to proceed" safety)

### App Polish
- [ ] Custom app icon (1024×1024 PNG with your design)
- [ ] Splash screen (matching app theme)
- [ ] App name on home screen: "Monolith"
- [ ] Deep link handling (optional)

---

## Known Problems & Solutions Reference

| Problem | Cause | Solution |
|---|---|---|
| Data lost on app update | localStorage cleared on uninstall | Use `@capacitor/preferences` instead of raw localStorage — it persists across updates |
| Date mismatch (yesterday shows as today) | `toISOString()` uses UTC | Use `new Date().toLocaleDateString('en-CA')` for local date |
| Tailwind CDN version is slow in WebView | Browser must fetch and compile CSS | Migrate to `@tailwindcss/vite` for build-time CSS |
| Fonts not loaded without internet | Google Fonts requires network | Either self-host fonts in `/public` or accept font fallback |
| Notification not firing on Android 12+ | Exact alarm restrictions tightened | Request `SCHEDULE_EXACT_ALARM` permission explicitly at runtime |
| App blank after backgrounding | React state lost | This shouldn't happen — localStorage persists. If it does, check for JS errors in Logcat |
| Biometric fails on emulator | Emulators have no fingerprint hardware | Test on real device. Can simulate fingerprint in some AVDs via extended controls |
| APK too large | All web assets bundled | Enable minification + tree shaking in vite.config.js |
| Android back button closes app | No navigation handler | Add `backButton` listener via `@capacitor/app` |
| Status bar overlaps content | WebView extends under status bar | Add `@capacitor/status-bar` and set `overlaysWebView: false` |
| React Router navigation broken on Android | MemoryRouter doesn't sync with native back | Already using MemoryRouter — this is correct; handle via the `backButton` listener |
| Input zoom on iOS (if testing on iOS later) | Font size < 16px triggers auto-zoom | Set font-size minimum to 16px on inputs, or disable user-scalable in viewport meta |

---

## Folder Structure (Final)

```
monolith-app/
├── android/                    # Android Studio project (Capacitor generated)
│   ├── app/
│   │   ├── src/main/
│   │   │   ├── AndroidManifest.xml
│   │   │   └── res/
│   │   └── build.gradle
│   └── build.gradle
├── public/
│   └── favicon.ico
├── src/
│   ├── components/
│   │   ├── TopAppBar.jsx
│   │   ├── BottomNavBar.jsx
│   │   └── LockScreen.jsx
│   ├── context/
│   │   └── AppContext.jsx       # All state + localStorage
│   ├── screens/
│   │   ├── Home.jsx
│   │   ├── Workout.jsx
│   │   ├── Calories.jsx
│   │   ├── History.jsx
│   │   ├── Stats.jsx
│   │   ├── HabitManager.jsx    # Phase 4
│   │   └── Settings.jsx        # Phase 9–10
│   ├── services/
│   │   └── notifications.js    # Phase 9
│   ├── App.jsx
│   ├── main.jsx
│   └── index.css
├── dist/                        # Built web app (Capacitor reads this)
├── capacitor.config.ts
├── package.json
├── tailwind.config.js
├── postcss.config.js
└── vite.config.js
```

---

## Quick Command Reference

```bash
# Start dev server (browser)
npm run dev

# Build web app
npm run build

# Sync web build to Android
npx cap sync android

# Open in Android Studio
npx cap open android

# Run directly on connected device
npx cap run android

# Build release APK (from android/ folder)
cd android && ./gradlew assembleRelease

# Install APK via ADB
adb install android/app/release/app-release.apk

# View device logs
adb logcat | grep -i chromium
```

---

## Recommended Build Order

Start here if you're following this plan top-to-bottom:

1. **Phase 1** → Verify `node --version` and `adb devices` work before anything else
2. **Phase 2** → Get the browser version working first, before touching Android at all
3. **Phase 3** → Wire up data persistence — this is the most impactful change
4. **Phase 8** → Get a first APK on your phone as early as possible (use hardcoded data if needed)
5. **Phases 4–7** → Add features iteratively, syncing to Android after each one
6. **Phases 9–10** → Add notifications and biometrics last (most Android-specific)
7. **Phase 11** → Final signed APK when you're happy with the app
8. **Phase 12** → Polish is ongoing — do it whenever something bothers you

---

*Plan written for: MONOLITH Performance Tracker*  
*Stack: Vite + React + Capacitor + Android*  
*Target: Personal APK, sideloaded, personal use only*
