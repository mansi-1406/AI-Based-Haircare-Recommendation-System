# KeshCare — AI-Based Hair Care Recommendation System
## Complete Technical Documentation

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Architecture Overview](#3-architecture-overview)
4. [Directory Structure](#4-directory-structure)
5. [Core Algorithm & AI Logic](#5-core-algorithm--ai-logic)
6. [Data Flow](#6-data-flow)
7. [State Management](#7-state-management)
8. [Form & Validation System](#8-form--validation-system)
9. [Firebase Integration](#9-firebase-integration)
10. [Styling Architecture](#10-styling-architecture)
11. [Routing](#11-routing)
12. [Environment Setup](#12-environment-setup)
13. [Running the Project](#13-running-the-project)
14. [Component Reference](#14-component-reference)
15. [Key Files Reference](#15-key-files-reference)

---

## 1. Project Overview

**KeshCare** is a personalized, AI-powered hair care recommendation web application. It analyses a user's hair type, scalp condition, lifestyle, and environmental factors to generate customized Ayurvedic and natural hair care routines.

- **Tagline:** "Your Hair, Understood by AI"
- **Core Purpose:** Deliver personalized, chemical-free, natural/Ayurvedic remedies specific to each user's hair profile
- **Target Audience:** Users seeking ingredient-specific, natural solutions grounded in Ayurveda without requiring a consultation

### Key Features

| Feature | Description |
|---|---|
| 4-Step Hair Assessment | Guided form capturing hair type, scalp, problems, and lifestyle |
| AI-Powered Recommendations | Google Gemini AI generates Ayurvedic remedy plans |
| Hardcoded Fallback Engine | Deterministic recommendations if Gemini is unavailable |
| Weekly Schedule | Personalized day-by-day hair care routine |
| Google Calendar Integration | Adds weekly routines as calendar events |
| History Tracking | Stores last 10 assessments locally |
| Share & Print | Native share API + print-to-PDF support |
| Firebase Auth (Optional) | Email/password authentication with Firestore profile persistence |
| Feedback Widget | Collects user ratings on remedy effectiveness |

---

## 2. Tech Stack

| Layer | Technology | Version |
|---|---|---|
| **Frontend Framework** | React | 18.3.1 |
| **Build Tool** | Vite | 8.0.10 |
| **Routing** | React Router DOM | 6.28.0 |
| **Styling** | Tailwind CSS | 3.4.14 |
| **Animations** | Framer Motion | 11.11.17 |
| **State Management** | Zustand (with persist) | 5.0.1 |
| **AI / LLM** | Google Generative AI SDK | 0.21.0 |
| **LLM Model** | Gemini 1.5 Flash | — |
| **Forms** | React Hook Form | 7.53.0 |
| **Validation** | Zod | 3.23.8 |
| **Auth & Database** | Firebase (Auth + Firestore + Analytics) | 11.10.0 |
| **Icons** | Lucide React | 0.469.0 |
| **Notifications** | React Hot Toast | 2.4.1 |
| **Linting** | ESLint | — |
| **Formatting** | Prettier | — |

---

## 3. Architecture Overview

KeshCare is a **client-side SPA (Single Page Application)** with optional Firebase backend services. There is no custom server — all logic runs in the browser.

```
┌─────────────────────────────────────────────────────────┐
│                        Browser                          │
│                                                         │
│  React App (Vite SPA)                                   │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Pages & Components (UI Layer)                   │  │
│  │  └─ React Router (client-side routing)           │  │
│  │                                                  │  │
│  │  Zustand Store (global state + localStorage)     │  │
│  │                                                  │  │
│  │  React Hook Form + Zod (form & validation)       │  │
│  │                                                  │  │
│  │  geminiService.js (recommendation engine)        │  │
│  │  ├─ Google Gemini API  (primary)                 │  │
│  │  └─ Hardcoded Plan    (fallback)                 │  │
│  └──────────────────────────────────────────────────┘  │
│                         │                               │
│            ┌────────────┴───────────┐                  │
│            ▼                        ▼                   │
│   Google Gemini API           Firebase Services         │
│   (external, HTTPS)           (Auth + Firestore)        │
└─────────────────────────────────────────────────────────┘
```

### Design Principles

- **Local-first:** All core features work without login; Zustand persists data in localStorage
- **Progressive enhancement:** Firebase features are optional — the app degrades gracefully without them
- **Dual-engine recommendations:** Gemini API is attempted first; hardcoded engine fires if Gemini is rate-limited or unavailable
- **Privacy-first:** No user data is sent anywhere except Gemini API (for recommendation) and Firebase (if user opts in)

---

## 4. Directory Structure

```
AI_Based_HairCare_Recommendation_System/
├── index.html                        # HTML entry point
├── vite.config.js                    # Vite bundler configuration
├── tailwind.config.js                # Tailwind CSS with custom design tokens
├── eslint.config.js                  # ESLint rules
├── package.json                      # Dependencies and scripts
│
└── src/
    ├── main.jsx                      # React root — wraps App with Router + AuthProvider
    ├── App.jsx                       # Root component — API key check + route definitions
    ├── App.css                       # App-level styles
    ├── index.css                     # Global CSS custom properties + base styles
    │
    ├── pages/
    │   ├── LandingPage.jsx           # Marketing homepage (hero, features, CTA)
    │   ├── AssessmentPage.jsx        # 4-step hair profile form
    │   ├── ResultsPage.jsx           # Displays generated recommendation plan
    │   ├── ProfilePage.jsx           # User profile + history
    │   ├── AboutPage.jsx             # About the product
    │   ├── NotFoundPage.jsx          # 404 fallback
    │   └── SetupRequiredPage.jsx     # Shown when VITE_GEMINI_API_KEY is missing
    │
    ├── components/
    │   ├── form/
    │   │   ├── FormStepper.jsx       # Step indicator (1–4 progress dots/labels)
    │   │   ├── StepOne.jsx           # Hair type + texture selection
    │   │   ├── StepTwo.jsx           # Scalp condition selection
    │   │   ├── StepThree.jsx         # Hair problems multi-select
    │   │   └── StepFour.jsx          # Lifestyle, wash frequency, water type, notes
    │   │
    │   ├── layout/
    │   │   ├── Navbar.jsx            # Top navigation bar
    │   │   └── Footer.jsx            # Page footer
    │   │
    │   ├── results/
    │   │   ├── RemedyCard.jsx        # Individual remedy display card
    │   │   ├── IngredientTag.jsx     # Inline ingredient chip/tag
    │   │   ├── InstructionSteps.jsx  # Numbered instruction list for a remedy
    │   │   └── FeedbackWidget.jsx    # Star rating / feedback collector
    │   │
    │   └── ui/
    │       ├── Button.jsx            # Reusable button (variants: primary, secondary, ghost)
    │       ├── Card.jsx              # Wrapper card with optional hover effect
    │       ├── Badge.jsx             # Status/category badge chip
    │       ├── Loader.jsx            # Animated loading spinner
    │       ├── ProgressBar.jsx       # Horizontal progress bar
    │       └── Tooltip.jsx           # Hover tooltip wrapper
    │
    ├── context/
    │   └── AuthContext.jsx           # Firebase Auth context + Firestore user profile sync
    │
    ├── hooks/
    │   ├── useRecommendation.js      # Orchestrates recommendation API call + error handling
    │   └── useLocalHistory.js        # Read/write assessment history from Zustand store
    │
    ├── services/
    │   ├── geminiService.js          # Core recommendation engine (Gemini + fallback)
    │   └── firebase.js               # Firebase app init + exports (auth, db, analytics)
    │
    ├── store/
    │   └── useKeshCareStore.js       # Zustand global store with localStorage persistence
    │
    ├── constants/
    │   └── hairData.js               # Enum maps for all form options
    │
    └── utils/
        ├── promptBuilder.js          # Builds structured prompt string for Gemini API
        └── validators.js             # Zod schemas for each assessment step
```

---

## 5. Core Algorithm & AI Logic

The recommendation engine lives in [src/services/geminiService.js](src/services/geminiService.js). It exports a single function:

```js
getHaircareRecommendations(userProfile) → Promise<RecommendationPlan>
```

### 5.1 Dual-Engine Strategy

```
getHaircareRecommendations(userProfile)
        │
        ▼
Is Gemini API key configured?
        │
   Yes  ├──────────────────────────────────────────────────►
        │                                          Call Gemini API
        │                                                │
        │                                    Success?   │
        │                                        │      │
        │                                   Yes ▼   No  ▼
        │                              Parse JSON  buildHardcodedPlan()
        │                                   │
        │                               Return plan
   No   ▼
buildHardcodedPlan(userProfile)
        │
        ▼
    Return plan
```

---

### 5.2 Gemini AI Engine (Primary)

**File:** [src/utils/promptBuilder.js](src/utils/promptBuilder.js)

The `buildGeminiPrompt(userProfile)` function assembles a detailed system + user prompt that instructs Gemini to act as an **Ayurvedic and natural haircare specialist**.

**Prompt enforces:**
- Only 100% natural, Ayurvedic ingredients (available at home or kirana stores)
- No chemicals, no brand names
- Ranked remedies by impact level
- Specific ingredient quantities, application durations, and safety warnings
- Response must be valid JSON matching this schema:

```json
{
  "summary": "string",
  "primaryConcern": "string",
  "tips": ["string"],
  "remedies": [
    {
      "id": "string",
      "name": "string",
      "category": "string",
      "targetProblems": ["string"],
      "difficulty": "easy | medium | hard",
      "frequency": "string",
      "prepTime": "string",
      "resultTimeline": "string",
      "ingredients": [
        { "name": "string", "quantity": "string", "role": "string" }
      ],
      "instructions": [
        { "step": 1, "action": "string" }
      ],
      "benefits": ["string"],
      "warnings": ["string"],
      "ayurvedicBasis": "string"
    }
  ],
  "dietaryAdvice": "string",
  "avoidList": ["string"],
  "weeklySchedule": [
    { "day": "string", "morning": "string", "evening": "string" }
  ],
  "followUpIn": "string"
}
```

**Model used:** `gemini-1.5-flash` (fast, cost-effective, handles structured JSON output)

---

### 5.3 Hardcoded Fallback Engine

**File:** [src/services/geminiService.js](src/services/geminiService.js) → `buildHardcodedPlan()`

A deterministic recommendation builder that fires when Gemini is unavailable. Uses **conditional branching** based on user profile.

#### Step-by-step algorithm:

```
INPUT: userProfile
        { hairType, hairTexture, scalpCondition, problems[], lifestyle[],
          washFrequency, waterType, notes }

STEP 1 — Label Mapping
  Convert enum IDs → human-readable strings using hairData.js constants.
  e.g.  "hair_fall" → "Hair Fall"
        "oily"      → "Oily"

STEP 2 — Primary Concern Detection
  if   problems includes 'hair_fall' OR 'thinning'
       → primaryConcern = "Hair fall and root weakness"
  elif problems includes 'dandruff' OR scalpCondition is 'dandruff'
       → primaryConcern = "Scalp imbalance and dandruff recurrence"
  elif problems includes 'dryness' OR 'split_ends'
       → primaryConcern = "Dryness, rough texture, and breakage"
  elif problems includes 'oiliness' OR scalpCondition is 'oily'
       → primaryConcern = "Excess oil and scalp buildup"
  else → primaryConcern = "Scalp and strand balance"

STEP 3 — Remedy Selection (conditional on detected flags)
  hasDandruff  = problems.includes('dandruff') || scalpCondition === 'dandruff'
  hasHairFall  = problems.includes('hair_fall') || problems.includes('thinning')
  hasOiliness  = problems.includes('oiliness') || scalpCondition === 'oily'
  hasDryness   = problems.includes('dryness')  || problems.includes('split_ends')

  Remedy 1 (Mask):
    hasDandruff || hasOiliness ? Neem-Aloe Scalp Mask
                               : Amla-Aloe Root Nourishing Mask

  Remedy 2 (Oil):
    hasHairFall  ? Bhringraj-Methi Oil
                 : Coconut-Hibiscus Oil

  Remedy 3 (Rinse):
    hasDandruff  ? Diluted ACV Rinse
                 : Rosemary-Mint Rinse

  Remedy 4:  Daily Nutrition & Recovery Plan  (always included)
  Remedy 5:  Damage-Control Styling Routine   (always included)

STEP 4 — Weekly Schedule Generation
  For each day (Monday–Sunday):
    Generate morning routine  (oil treatment, mask day, protective style)
    Generate evening routine  (scalp check, nutrition reminder, rest position)
  Personalised with user's washFrequency and waterType.

OUTPUT: Full RecommendationPlan object matching the same schema as Gemini output
```

Each remedy object contains:
- `id`, `name`, `category`, `targetProblems[]`
- `difficulty`, `frequency`, `prepTime`, `resultTimeline`
- `ingredients[]` → `{ name, quantity, role }`
- `instructions[]` → `{ step, action }`
- `benefits[]`, `warnings[]`, `ayurvedicBasis`

---

### 5.4 Recommendation Hook (`useRecommendation`)

**File:** [src/hooks/useRecommendation.js](src/hooks/useRecommendation.js)

Orchestrates the full lifecycle around calling `getHaircareRecommendations`:

| Concern | Implementation |
|---|---|
| **Rate limiting** | Tracks last error timestamp; enforces 60-second cooldown after `RATE_LIMIT` errors |
| **Debouncing** | 500 ms delay before API call to absorb rapid re-triggers |
| **Error classification** | Distinguishes `RATE_LIMIT`, `MODEL_NOT_FOUND`, and generic errors |
| **History** | Saves `{ profile, result, timestamp }` to Zustand store (max 10 entries) |
| **UI state** | Updates `isLoading`, `recommendations`, `error` for downstream components |

---

## 6. Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ AssessmentPage.jsx (4 steps)                                    │
│   StepOne → StepTwo → StepThree → StepFour                      │
│   Validated by Zod per step via React Hook Form                 │
│   Form values synced to Zustand on every change                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │ formData
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ useKeshCareStore (Zustand)                                      │
│   formData, currentStep, isLoading, recommendations, history   │
└──────────────────────────┬──────────────────────────────────────┘
                           │ profile extracted on submit
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ useRecommendation hook                                          │
│   Rate limit check → Debounce (500ms) → API call               │
└──────────┬───────────────────────────────────────┬─────────────┘
           │                                       │
           ▼                                       ▼
┌──────────────────────┐             ┌─────────────────────────┐
│ Gemini API           │    error /  │ buildHardcodedPlan()    │
│ (geminiService.js)   │ unavailable │ (deterministic fallback)│
│ Returns JSON string  │────────────►│ Returns same schema     │
└──────────────────────┘             └─────────────────────────┘
           │                                       │
           └──────────────┬────────────────────────┘
                          │ recommendations object
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│ Zustand store updated: recommendations + history entry added    │
│ Toast notification shown                                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│ ResultsPage.jsx                                                 │
│  ├─ Summary & Primary Concern                                   │
│  ├─ Quick Tips list                                             │
│  ├─ RemedyCard × 5 (ingredients + step-by-step instructions)   │
│  ├─ Dietary Advice                                              │
│  ├─ Avoid List                                                  │
│  ├─ Weekly Schedule (+ Google Calendar integration)             │
│  ├─ FeedbackWidget (star rating)                                │
│  └─ Share / Print / Start Over actions                         │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. State Management

**File:** [src/store/useKeshCareStore.js](src/store/useKeshCareStore.js)

Zustand is used as the single global store. It holds:

| State Key | Type | Description |
|---|---|---|
| `formData` | Object | Current assessment form values |
| `currentStep` | Number | Active step (1–4) |
| `totalSteps` | Number | Always 4 |
| `recommendations` | Object \| null | Generated recommendation plan |
| `isLoading` | Boolean | True while API call is in flight |
| `error` | String \| null | Last error message |
| `history` | Array | Past assessments, max 10 entries |

**Persistence:** Only `history` is persisted to `localStorage` via Zustand's `partialize` middleware under the key `"keshcare-storage"`. All other state resets on page refresh intentionally to avoid stale form state.

---

## 8. Form & Validation System

### Multi-Step Form

The assessment form spans 4 steps managed by `AssessmentPage.jsx`:

| Step | Component | Fields Collected |
|---|---|---|
| 1 | `StepOne.jsx` | `hairType` (straight/wavy/curly/coily), `hairTexture` (fine/medium/thick) |
| 2 | `StepTwo.jsx` | `scalpCondition` (normal/oily/dry/sensitive/dandruff/combination) |
| 3 | `StepThree.jsx` | `problems[]` (multi-select from 10 options) |
| 4 | `StepFour.jsx` | `lifestyle[]`, `washFrequency`, `waterType`, `notes` (optional) |

**Hair problems options:** hair fall, dandruff, dryness, frizz, scalp infection, premature greying, slow growth, split ends, oiliness, thinning

### Zod Validation

**File:** [src/utils/validators.js](src/utils/validators.js)

Step-wise schemas validate only the current step's fields, enabling progressive form completion:

```js
// Full schema
assessmentSchema = z.object({
  hairType:       z.string().min(1),
  hairTexture:    z.string().min(1),
  scalpCondition: z.string().min(1),
  problems:       z.array(z.string()).min(1),
  lifestyle:      z.array(z.string()).optional(),
  washFrequency:  z.string().min(1),
  waterType:      z.string().min(1),
  notes:          z.string().max(500).optional(),
})
```

React Hook Form handles registration, error state, and submission; Zod handles schema validation via `zodResolver`.

---

## 9. Firebase Integration

**File:** [src/services/firebase.js](src/services/firebase.js)

Firebase is **optional**. A `isFirebaseConfigured` boolean flag is exported and checked throughout the app. If Firebase env vars are absent, the app runs in local-only mode with no backend.

| Service | Usage |
|---|---|
| **Firebase Auth** | Email/password sign-up and login |
| **Firestore** | Stores user profile and recommendation history per UID |
| **Firebase Analytics** | Tracks page views and key user events |

**File:** [src/context/AuthContext.jsx](src/context/AuthContext.jsx)

Provides:
- `currentUser` — authenticated Firebase user object
- `signUp(email, password)` — creates account
- `login(email, password)` — signs in
- `logout()` — signs out
- `userProfile` — Firestore document for the current user

---

## 10. Styling Architecture

### Tailwind CSS + CSS Custom Properties

All design tokens are defined as CSS variables in `index.css` and consumed via Tailwind's extended theme in `tailwind.config.js`:

```css
/* index.css — design tokens (examples) */
:root {
  --color-primary: ...;
  --color-surface: ...;
  --shadow-card: ...;
  --radius-xl: ...;
}
```

```js
// tailwind.config.js extends these into Tailwind utilities
theme: {
  extend: {
    colors:     { primary: 'var(--color-primary)', ... },
    boxShadow:  { card: 'var(--shadow-card)', ... },
    borderRadius: { xl: 'var(--radius-xl)', ... },
  }
}
```

### Custom Animations (Tailwind)

| Class | Effect |
|---|---|
| `animate-float` | Gentle vertical float |
| `animate-shimmer` | Loading shimmer sweep |
| `animate-pulse-ring` | Expanding ring pulse |
| `animate-hair-flow` | Flowing wave motion |
| `animate-gradient-shift` | Gradient color shift |
| `animate-slide-up` | Enter from bottom |
| `animate-fade-in` | Opacity fade in |
| `animate-scale-in` | Scale from 95% to 100% |
| `animate-leaf-sway` | Gentle side-to-side sway |
| `animate-spin-slow` | Slow continuous rotation |

### Typography

- **Display / Headings:** Lora, Playfair Display (serif — signals premium, Ayurvedic feel)
- **Body / UI:** Inter, -apple-system (sans-serif — clean readability)

### Framer Motion

Used for:
- Page-level transitions via `AnimatePresence`
- Staggered list animations (remedy cards, feature sections)
- Scroll-triggered fade-ins via `useInView`
- Hero section particle and entrance animations
- Smooth form step transitions

---

## 11. Routing

**File:** [src/App.jsx](src/App.jsx)

React Router DOM v6 with `BrowserRouter` wrapping from `main.jsx`.

| Path | Page | Description |
|---|---|---|
| `/` | `LandingPage` | Marketing homepage |
| `/assess` | `AssessmentPage` | 4-step hair assessment form |
| `/results` | `ResultsPage` | Recommendation results display |
| `/about` | `AboutPage` | About KeshCare |
| `/profile` | `ProfilePage` | User profile + assessment history |
| `*` | `NotFoundPage` | 404 fallback |
| — | `SetupRequiredPage` | Rendered app-wide if `VITE_GEMINI_API_KEY` is missing |

**App-level guard:** Before rendering any routes, `App.jsx` checks `import.meta.env.VITE_GEMINI_API_KEY`. If absent, `SetupRequiredPage` is shown instead of the entire app.

---

## 12. Environment Setup

Create a `.env` file in the project root:

```env
# Required — Gemini AI
VITE_GEMINI_API_KEY=your_gemini_api_key_here

# Optional — Firebase (app works without these)
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
VITE_FIREBASE_MEASUREMENT_ID=
```

**Getting a Gemini API key:**
1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Create a new API key
3. Paste it as `VITE_GEMINI_API_KEY` in your `.env` file

> All env variables must be prefixed with `VITE_` for Vite to expose them to the browser.

---

## 13. Running the Project

### Prerequisites

- Node.js v18+
- npm v9+ (or pnpm / yarn)

### Install dependencies

```bash
npm install
```

### Start development server

```bash
npm run dev
```

App runs at `http://localhost:5173` (default Vite port).

### Build for production

```bash
npm run build
```

Output goes to `dist/`. Preview with:

```bash
npm run preview
```

### Lint

```bash
npm run lint
```

### Format

```bash
npm run format
```

---

## 14. Component Reference

### Pages

| Component | Path | Purpose |
|---|---|---|
| `LandingPage` | `pages/LandingPage.jsx` | Hero section, feature highlights, testimonials, CTA |
| `AssessmentPage` | `pages/AssessmentPage.jsx` | Renders `FormStepper` + current Step component; syncs to Zustand |
| `ResultsPage` | `pages/ResultsPage.jsx` | Reads `recommendations` from Zustand; renders remedy cards, schedule, actions |
| `ProfilePage` | `pages/ProfilePage.jsx` | Displays user info + past assessment history from Zustand |
| `AboutPage` | `pages/AboutPage.jsx` | Static informational page |
| `SetupRequiredPage` | `pages/SetupRequiredPage.jsx` | Shown when API key env var is missing |
| `NotFoundPage` | `pages/NotFoundPage.jsx` | 404 catch-all |

### Form Components

| Component | Purpose |
|---|---|
| `FormStepper` | Visual step indicator showing current step out of 4 |
| `StepOne` | Hair type radio cards + hair texture radio cards |
| `StepTwo` | Scalp condition selection (6 options) |
| `StepThree` | Multi-select checkbox grid for hair problems (10 options) |
| `StepFour` | Lifestyle multi-select + wash frequency + water type dropdowns + notes textarea |

### Result Components

| Component | Purpose |
|---|---|
| `RemedyCard` | Full remedy display: name, difficulty badge, ingredients, instructions, benefits, warnings |
| `IngredientTag` | Chip showing ingredient name + quantity |
| `InstructionSteps` | Numbered list of action steps for applying a remedy |
| `FeedbackWidget` | 1–5 star rating collector with optional text comment |

### UI Primitives

| Component | Props |
|---|---|
| `Button` | `variant` (primary/secondary/ghost), `size`, `disabled`, `loading`, `onClick` |
| `Card` | `className`, `hover`, `children` |
| `Badge` | `variant`, `label` |
| `Loader` | `size`, `color` |
| `ProgressBar` | `value` (0–100), `color` |
| `Tooltip` | `content`, `children` |

---

## 15. Key Files Reference

| File | What it does |
|---|---|
| [src/services/geminiService.js](src/services/geminiService.js) | Main recommendation engine. Exports `getHaircareRecommendations()`. Calls Gemini API or falls back to `buildHardcodedPlan()`. |
| [src/utils/promptBuilder.js](src/utils/promptBuilder.js) | Constructs the full Gemini prompt with user profile, Ayurvedic constraints, and JSON output schema. |
| [src/store/useKeshCareStore.js](src/store/useKeshCareStore.js) | Zustand global store. Holds form data, loading state, recommendations, and persisted history. |
| [src/hooks/useRecommendation.js](src/hooks/useRecommendation.js) | Custom hook orchestrating API calls with debounce, rate limiting, error handling, and history saves. |
| [src/context/AuthContext.jsx](src/context/AuthContext.jsx) | Firebase Auth + Firestore context provider. Optional — app works without it. |
| [src/constants/hairData.js](src/constants/hairData.js) | Enum maps: HAIR_TYPES, HAIR_TEXTURES, SCALP_CONDITIONS, HAIR_PROBLEMS, LIFESTYLE_FACTORS, WATER_TYPES, WASH_FREQUENCIES. |
| [src/utils/validators.js](src/utils/validators.js) | Zod schemas for each assessment step; enables per-step progressive validation. |
| [src/services/firebase.js](src/services/firebase.js) | Firebase app init. Exports `auth`, `db`, `analytics`, and `isFirebaseConfigured` flag. |
| [src/App.jsx](src/App.jsx) | Root component. Checks for API key; sets up all routes with React Router. |
| [src/main.jsx](src/main.jsx) | React root. Mounts app with `BrowserRouter` and `AuthProvider`. |
| [tailwind.config.js](tailwind.config.js) | Tailwind theme extending CSS custom properties for colors, shadows, radii, and custom animations. |
| [vite.config.js](vite.config.js) | Vite configuration with React plugin. |

---

*Documentation generated for KeshCare v1.0.0 — April 2026*
