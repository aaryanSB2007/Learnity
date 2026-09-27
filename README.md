# Learnity 🎓

> **An AI-powered adaptive learning platform** for engineering students — featuring intelligent quizzes, real-time analytics, curriculum DAG progression, and live AI tutoring via Groq & Gemini.

---

## Table of Contents

- [Overview](#overview)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [How It Works — Full Flow](#how-it-works--full-flow)
  - [1. Landing & Subject Selection](#1-landing--subject-selection)
  - [2. Quiz Engine](#2-quiz-engine)
  - [3. AI Rationale](#3-ai-rationale)
  - [4. Post-Quiz Analytics](#4-post-quiz-analytics)
  - [5. Dashboard](#5-dashboard)
  - [6. Student Profile](#6-student-profile)
  - [7. Curriculum Timeline](#7-curriculum-timeline)
  - [8. AI Learning Path](#8-ai-learning-path)
- [API Reference](#api-reference)
- [AI Integration](#ai-integration)
- [Data & Database](#data--database)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)

---

## Overview

Learnity is built for engineering students studying across four core subjects:

| Subject | Code | Topics Covered |
|---|---|---|
| Analysis of Algorithms | `AOA` | Sorting, Greedy, DP, Graph Algorithms |
| Computer Organization & Architecture | `COA` | CPU, Pipelines, Memory Hierarchy |
| Full Stack Java & Web | `FSJP` | Java OOP, Spring Boot, Web Dev |
| Mathematics | `MATHS` | Discrete Math, Linear Algebra, Calculus |

The platform **adapts** to each student — tracking what they know, what they've missed, and intelligently steering them toward the concepts they need most.

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | Next.js 16 (App Router) |
| **UI** | React 19, Tailwind CSS v4, Framer Motion, GSAP |
| **Charts** | Recharts |
| **State Management** | Zustand |
| **Data Fetching** | TanStack React Query |
| **AI Providers** | Groq (`llama-3.1-70b-versatile`) → Gemini (`gemini-3.8-flash`) fallback |
| **Schema Validation** | Zod |
| **Database** | JSON flat-file (`learnity-db.json`) via `src/lib/db.ts` |
| **PDF Export** | jsPDF + html2canvas |
| **Animations** | tsParticles, Lenis smooth-scroll |

---

## Project Structure

```
src/
├── app/
│   ├── (root)/            # Landing page
│   ├── (dashboard)/
│   │   ├── dashboard/     # Analytics dashboard
│   │   └── profile/       # Student profile & neural grid
│   ├── (learn)/
│   │   └── quiz/[quizId]/ # Quiz engine per subject
│   └── api/
│       ├── ai/
│       │   ├── explain/   # POST — AI misconception explanation
│       │   ├── example/   # POST — AI concrete code example
│       │   └── practice/  # POST — AI practice question generator
│       ├── quiz/
│       │   ├── questions/ # GET  — Sampled questions for topic
│       │   └── submit/    # POST — Submit attempt & update DB
│       ├── profile/
│       │   └── analytics/ # GET  — Live subject analytics
│       └── learning-path/
│           └── [userId]/  # GET  — Gap-prioritized learning path
│
├── components/
│   ├── features/
│   │   ├── QuizEngine.tsx         # Core quiz UI & state machine
│   │   ├── AITutorOverlay.tsx     # Slide-in AI explanation panel
│   │   ├── CurriculumTimeline.tsx # DAG-based topic progression
│   │   ├── LearningPathTree.tsx   # Visual learning path tree
│   │   └── SubjectSelector.tsx    # Subject picker on quiz landing
│   └── dashboard/
│       ├── PerformanceGraph.tsx   # Line chart — score over time
│       ├── SubjectRadarChart.tsx  # Radar — all 4 subjects
│       ├── StreakGrid.tsx         # Neural consistency heatmap
│       └── SubjectRecommendationsGrid.tsx # AI topic recommendations
│
├── lib/
│   ├── ai.ts     # All AI calls (Groq → Gemini → Static fallback)
│   └── db.ts     # JSON DB read/write + question sampling logic
│
├── data/
│   ├── curriculum.ts    # Full subject/topic/concept DAG definitions
│   ├── question-bank.ts # Static curated question bank (all subjects)
│   ├── quizzes.ts       # Quiz metadata per subject
│   └── db/
│       └── learnity-db.json  # Persistent flat-file database
│
├── store/              # Zustand stores
│   ├── useUserStore.ts    # Profile, analytics, mastery
│   ├── useQuizStore.ts    # Active quiz session state
│   ├── useSubjectStore.ts # Selected subject
│   └── useUIStore.ts      # Dark mode, sidebar
│
└── types/              # TypeScript interfaces for all entities
```

---

## How It Works — Full Flow

```
┌─────────────┐    ┌──────────────┐    ┌───────────────┐    ┌─────────────┐
│   Landing   │───▶│ Quiz Engine  │───▶│  AI Rationale │───▶│  Dashboard  │
│    Page     │    │ (per subject)│    │  (Groq/Gemini)│    │  Analytics  │
└─────────────┘    └──────────────┘    └───────────────┘    └─────────────┘
                          │                                        │
                          ▼                                        ▼
                   ┌──────────────┐                      ┌─────────────────┐
                   │  Submit API  │                      │ Student Profile │
                   │  (DB write)  │                      │  Neural Grid    │
                   └──────────────┘                      └─────────────────┘
                          │                                        │
                          ▼                                        ▼
                   ┌──────────────┐                      ┌─────────────────┐
                   │  Curriculum  │                      │ Learning Path   │
                   │  DAG Unlock  │                      │ AI Recommender  │
                   └──────────────┘                      └─────────────────┘
```

---

### 1. Landing & Subject Selection

**Page:** `/` → `/quiz/[subjectId]`

- Animated hero with particle field (`HeroParticles`) and smooth scroll (`Lenis`)
- Student clicks **"Begin Academy"** → routed to subject selector
- `SubjectSelector` presents the 4 subjects as cards; clicking one loads the quiz

---

### 2. Quiz Engine

**Page:** `/quiz/aoa` | `/quiz/coa` | `/quiz/fsjp` | `/quiz/maths`  
**Component:** [`QuizEngine.tsx`](src/components/features/QuizEngine.tsx)

```
GET /api/quiz/questions?subjectId=AOA&count=5
         │
         ▼
  sampleQuestionsForTopic()  ← src/lib/db.ts
         │
         ├── Checks "seen_questions" in DB for this user+subject
         ├── Filters out recently-seen question IDs
         ├── Randomly selects 5–8 from the pool
         └── Falls back to full bank if pool is small
```

**What the student sees:**
- Progress bar across the top
- MCQ question with 4 options
- After selecting → immediate correct/incorrect feedback
- "AI Rationale" button appears on wrong answers
- Navigation to next question; final screen shows score summary

**Anti-repeat logic:** Each question ID is recorded in `seen_questions` per `userId+subjectId`. Future sessions deprioritize recently-seen questions, so the bank rotates naturally over time.

---

### 3. AI Rationale

**Triggered by:** Clicking "AI Rationale" after a wrong answer  
**Component:** [`AITutorOverlay.tsx`](src/components/features/AITutorOverlay.tsx)  
**API:** `POST /api/ai/explain`

```
Student clicks "AI Rationale"
         │
         ▼
POST /api/ai/explain
  { subtopic, questionText, studentAnswer, correctAnswer, masteryScore }
         │
         ▼
  explainMisconception() ← src/lib/ai.ts
         │
         ├── 1️⃣  Try Groq (llama-3.1-70b-versatile) — ~100ms target
         │        If fails →
         ├── 2️⃣  Try Gemini (gemini-3.8-flash) — ~4s target
         │        If fails →
         └── 3️⃣  Static fallback explanation — instant, always works
         │
         ▼
  Returns:
  {
    explanation: "Why the wrong answer is wrong...",
    keyConcept: "Core principle to remember",
    breakdown: ["Step 1", "Step 2", "Takeaway"],
    level: "beginner" | "intermediate" | "advanced"
  }
```

The `level` is automatically calibrated to the student's `masteryScore` for that subject — beginners get simpler language, advanced learners get theory-heavy breakdowns.

---

### 4. Post-Quiz Analytics (Submit)

**Triggered by:** Completing the quiz  
**API:** `POST /api/quiz/submit`

```
Quiz completed
         │
         ▼
POST /api/quiz/submit
  { userId, subjectId, moduleId, answers[] }
         │
         ▼
  For each answer:
    ├── Write to attempt_history[] in learnity-db.json
    └── Update seen_questions[] to track seen question IDs
         │
         ▼
  Compute:
    ├── Score (correct / total)
    ├── Module mastery score (rolling average of attempts)
    ├── Subtopic-level accuracy breakdown
    └── Trigger DAG unlock check
         │
         ▼
  DAG Unlock Logic:
    - If module mastery ≥ threshold → mark as COMPLETED
    - Unlock prerequisite-satisfied downstream modules → IN_PROGRESS
```

---

### 5. Dashboard

**Page:** `/dashboard`

Pulls live data from `GET /api/profile/analytics?userId=...`

| Widget | What it shows |
|---|---|
| **Performance Graph** | Score over time per subject (line chart) |
| **Subject Radar Chart** | All 4 subjects plotted on one radar — instantly spot weak areas |
| **Neural Consistency Grid** | Heatmap of daily activity intensity (like GitHub contributions) |
| **Subject Recommendations** | AI-prioritized list of topics to focus on next |
| **Active Curriculum** | Snapshot of current DAG progress across all subjects |

---

### 6. Student Profile

**Page:** `/profile`

```
GET /api/profile/analytics
         │
         ▼
  computeLiveAnalytics()
         │
         ├── Aggregate attempt_history by subject
         ├── Calculate per-subject mastery, accuracy, streak
         ├── Build consistencyGrid (last 52 weeks of activity)
         └── Return subjectsAnalytics[] + overall Sync Level
```

**Displayed as:**
- Avatar + Sync Level badge + Total Sync %
- Per-subject mastery cards (AOA / COA / FSJP / MATHS)
- Neural Consistency grid (activity heatmap)
- Radar chart covering all 4 subjects
- Export Report button (PDF via jsPDF)

---

### 7. Curriculum Timeline

**Component:** [`CurriculumTimeline.tsx`](src/components/features/CurriculumTimeline.tsx)

The curriculum is modeled as a **Directed Acyclic Graph (DAG):**

```
asymptotic-notation ──▶ binary-search-analysis
        │
        ▼
recurrence-relations ──▶ master-theorem ──▶ merge-sort
                                       └──▶ quick-sort
```

Each concept node has:
- `prerequisites: string[]` — must be `COMPLETED` before this unlocks
- `status`: `LOCKED` → `IN_PROGRESS` → `COMPLETED`

A topic unlocks (`IN_PROGRESS`) only when **all its prerequisites** are `COMPLETED`. Achieving a passing score on that topic marks it `COMPLETED` and cascades unlocks downstream.

---

### 8. AI Learning Path

**API:** `GET /api/learning-path/[userId]`

```
GET /api/learning-path/default-user
         │
         ▼
  Build gap-prioritized learning path:
         │
         ├── Load all attempt_history for user
         ├── Identify subtopics with repeated failures (gap detection)
         ├── Score each module: gap_weight + prerequisite_depth + recency
         ├── Sort by priority score descending
         └── Return ordered list of recommended modules
         │
         ▼
  Returns:
  [
    { moduleId, subjectId, priority, reason: "Repeated misses on X" },
    ...
  ]
```

This drives the **AI Recommendations** widget on the dashboard — students always see _what to study next_ based on their actual performance gaps, not a generic syllabus order.

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/quiz/questions` | Sample quiz questions for a subject/module |
| `POST` | `/api/quiz/submit` | Submit quiz answers, update DB, unlock DAG |
| `POST` | `/api/ai/explain` | Generate AI explanation for a wrong answer |
| `POST` | `/api/ai/example` | Generate a concrete code/math example |
| `POST` | `/api/ai/practice` | Generate a single practice question |
| `GET` | `/api/profile/analytics` | Compute live analytics for a user |
| `GET` | `/api/learning-path/[userId]` | Return gap-prioritized learning path |

### Example: Submit Quiz

```bash
curl -X POST http://localhost:3000/api/quiz/submit \
  -H "Content-Type: application/json" \
  -d '{
    "userId": "default-user",
    "subjectId": "AOA",
    "moduleId": "algorithm-fundamentals",
    "answers": [
      { "questionId": "q1", "correct": true, "subtopic": "sorting" },
      { "questionId": "q2", "correct": false, "subtopic": "greedy" }
    ]
  }'
```

### Example: Get AI Explanation

```bash
curl -X POST http://localhost:3000/api/ai/explain \
  -H "Content-Type: application/json" \
  -d '{
    "subtopic": "Sorting",
    "questionText": "What is quicksort?",
    "studentAnswer": "A sorting algorithm",
    "correctAnswer": "A divide and conquer sorting algorithm",
    "masteryScore": 70
  }'
```

---

## AI Integration

**File:** [`src/lib/ai.ts`](src/lib/ai.ts)

All AI calls follow a **3-tier waterfall**:

```
Request
   │
   ├─ 1️⃣  Groq API (llama-3.1-70b-versatile)
   │       Fast LPU inference, ~100–300ms
   │       If FAILED (timeout / error / bad JSON) →
   │
   ├─ 2️⃣  Gemini API (gemini-3.8-flash)
   │       Google's latest flash model, ~3–7s
   │       If FAILED →
   │
   └─ 3️⃣  Static Fallback
           Pre-written template explanation
           Always succeeds — no external dependency
```

All responses are validated with **Zod schemas** before being returned. Malformed JSON from models is safely caught and falls through to the next tier.

**Key functions:**

| Function | Purpose |
|---|---|
| `generateScopedQuestions()` | Generate 5–8 MCQs scoped to a module |
| `explainMisconception()` | Explain why a student got a question wrong |
| `generateExample()` | Produce a concrete code/math example |
| `generatePracticeQuestion()` | Create a single targeted practice MCQ |

**API keys** are read from `.env.local` on every request (no restart needed):

```env
GROQ_API_KEY=gsk_...
GEMINI_API_KEY=AIza...
```

---

## Data & Database

**File:** [`src/data/db/learnity-db.json`](src/data/db/learnity-db.json)

Learnity uses a **JSON flat-file database** (no external DB required). The schema is:

```json
{
  "attempt_history": [
    {
      "id": "att-001",
      "userId": "default-user",
      "subjectId": "AOA",
      "moduleId": "algorithm-fundamentals",
      "questionId": "aoa-bnk-01",
      "correct": true,
      "timestamp": "2026-09-24T09:44:35Z",
      "subtopic": "master-theorem"
    }
  ],
  "seen_questions": {
    "default-user|AOA": ["aoa-bnk-01", "aoa-bnk-02"]
  },
  "user_curriculum_progress": {
    "default-user": {
      "asymptotic-notation": "COMPLETED",
      "master-theorem": "IN_PROGRESS"
    }
  }
}
```

**All DB operations** go through [`src/lib/db.ts`](src/lib/db.ts):
- `sampleQuestionsForTopic()` — anti-repeat question selection
- `recordAttempt()` — write quiz answer to history
- `computeLiveAnalytics()` — aggregate all metrics for a user
- `getOrUnlockCurriculumProgress()` — DAG unlock cascade logic

---

## Getting Started

### Prerequisites

- Node.js 20+
- npm or yarn

### Install

```bash
git clone <repo-url>
cd Learnity-main
npm install
```

### Configure Environment

```bash
cp .env.example .env.local
```

Edit `.env.local` and fill in your API keys:

```env
GROQ_API_KEY=your-groq-key-here
GEMINI_API_KEY=your-gemini-key-here
```

> **Note:** Get a free Groq key at [console.groq.com/keys](https://console.groq.com/keys)  
> Get a free Gemini key at [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey)

### Run

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

### Build for Production

```bash
npm run build
npm start
```

---

## Environment Variables

| Variable | Required | Description |
|---|---|---|
| `GROQ_API_KEY` | Optional | Enables fast Groq LLM inference (primary AI) |
| `GEMINI_API_KEY` | Optional | Enables Google Gemini inference (fallback AI) |

Both keys are optional — the platform works fully without them using the static question bank and pre-written explanations. AI features become live when at least one key is present.

---

## Dark Mode

Learnity supports full dark mode via `next-themes`. Toggle with the 🌙 button in the navbar. All pages, charts, and components are theme-aware using CSS variables defined in [`src/app/globals.css`](src/app/globals.css).

---

## PDF Report Export

On the Profile page, clicking **"Export Report"** generates a PDF snapshot of the student's analytics (radar chart, mastery scores, consistency grid) using `jsPDF` + `html2canvas`.

---

*Built with ❤️ for engineering students.*
