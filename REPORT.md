# AI Visibility Platform — Full Design & Build Report

> **Permanent reference document** — paste this into Claude Code to build the app.

---

## Project Overview

Build a **SaaS AI Visibility Tracking Platform** that monitors how AI search engines (ChatGPT, Claude, Gemini, Perplexity, Grok, NVIDIA, Google AI Overview) respond to brand/business-related prompts. The platform tracks visibility scores, sentiment, citation sources, and competitor comparisons over time.

---

## Design System

### Colour Palette
- **Primary Accent:** Amber — `#F59E0B` / `#D97706`
- **Background:** Deep dark navy — `#0A0F1E`
- **Card Background:** `#111827` / `#1F2937`
- **Borders:** `#374151`
- **Text Primary:** `#F9FAFB`
- **Text Secondary:** `#9CA3AF`
- **Success:** `#10B981`
- **Warning:** `#F59E0B`
- **Error/Danger:** `#EF4444`

### Typography
- **Headings:** Inter Bold / Semi-Bold
- **Body:** Inter Regular
- **Monospace/Data:** JetBrains Mono (for scores, percentages)

### Design Principles
- Clean, minimal sidebar navigation
- Card-based layout with subtle borders
- Data-first — numbers are hero elements
- Amber/gold used ONLY for key highlights, CTAs, and active states
- Dark mode by default
- Micro-animations on score changes
- Desktop-first, mobile-responsive

---

## App Architecture & Page Structure

### Sidebar Navigation
```
Logo (top)
─────────────
Dashboard
Prompts
AI Engines
Visibility Score
Sentiment
Competitors
Reports
Settings
─────────────
[User Avatar + Name]
[Credits Remaining: 47]
```

---

### 1. Dashboard Page

**Top Row — 4 KPI Cards:**
- Overall Visibility Score (large %, amber ring chart)
- AI Engines Monitored (count + engine logos)
- Prompts Tracked (number)
- Last Scan + countdown to next scan

**Next Scan Timer:**
- "Next Scan in: 18:42:33" (24hr countdown)
- Progress bar depleting
- Hits 0 → auto-triggers workflow → resets to 24:00:00
- Manual Run button: "Run Now (costs 1 credit per engine per prompt)"

**Middle Row — Visibility Over Time Chart:**
- Line chart, one line per AI engine
- X-axis: dates | Y-axis: visibility score 0–100
- Engine line colours: ChatGPT=green, Claude=purple, Gemini=blue, Perplexity=orange, Grok=red, Google=amber

**Bottom Row:**
- Left: Top Performing Prompts table
- Right: Sentiment breakdown donut chart (Positive / Neutral / Negative)

---

### 2. Prompts Page

Table columns: Prompt Text | Category | Last Score | Trend | Engines | Status

- Add New Prompt button (amber CTA)
- Filter by category, engine, date range
- Click row → expands to per-engine breakdown
- Edit / Delete / Pause per prompt
- Credit logic: "1 prompt × 5 engines = 5 credits per run"

---

### 3. AI Engines Page

Grid of engine cards showing:
- Engine logo (dynamic — only show active engines)
- Engine name
- Current visibility score
- Response count this month
- Toggle ON/OFF
- Last tested timestamp

**Active Engines:** ChatGPT (GPT-5 Mini), Claude (Haiku 4.5), Gemini (3 Flash), Perplexity (Sonar), Grok (Fast), NVIDIA (Nemotron), Google (ValueSERP/AI Overview)

---

### 4. Visibility Score Page

- Heatmap table: Prompts (rows) x Engines (columns) — score in each cell
- Colour scale: red (0–40) → yellow (40–70) → green (70–100)
- Date range filter
- Export to CSV

**Score Formula:**
```
Score = (Mentioned x 40) + (Position Score x 30) + (Sentiment Score x 20) + (Citation x 10)

Mentioned: 1 if brand appears, 0 if not
Position Score: 1.0 if #1 | 0.7 if #2-3 | 0.4 if #4-5 | 0 if not listed
Sentiment Score: 1.0 positive | 0.5 neutral | 0 negative
Citation: 1 if website cited | 0 if not
```

---

### 5. Sentiment Page

Three-column layout:
- Left: Sentiment over time line chart
- Centre: Recent responses with sentiment tag (Positive / Neutral / Negative)
- Right: Per-engine sentiment breakdown bar chart

Response cards show:
- Engine logo + name
- Prompt text
- Full AI response (truncated, expand button)
- Sentiment badge
- Date/time + visibility score

---

### 6. Competitors Page

Add competitors by: Business name + Website URL + Keywords

Comparison table: Brand | Visibility Score | Mentions | Sentiment | Top Engine

- Competitor logos fetched from Google Favicon API or Clearbit
- Side-by-side bar comparison chart
- "You vs Competitors" visibility gap indicator

---

### 7. Reports Page

- List of generated reports pushed to GitHub Pages
- Date | Prompt | Engines | Overall Score | Link to HTML report
- Reports hosted at: `https://moylal09.github.io/ai-visibility-reports/`
- Each report = standalone HTML file, viewable and shareable

---

### 8. Settings Page

Tabs:
- **Account** — name, email, timezone (Australia/Melbourne)
- **Prompts** — default categories, prompt templates
- **Engines** — enable/disable engines, API key management
- **Credits** — balance, purchase more, usage history
- **Notifications** — email alerts when score drops >10%, weekly digest
- **Integrations** — Google Sheets, GitHub Pages URL

---

## Credit System

```
Rule: 1 prompt x 1 engine = 1 credit

Example:
3 prompts x 5 engines = 15 credits per run
With 100 credits = ~6 full scans

Manual Run = same credit cost, triggered instantly
Auto Run (daily 6am AEST) = deducts credits automatically

Credit top-up tiers:
Starter: 100 credits — $9
Pro: 500 credits — $29
Agency: 2000 credits — $79
```

---

## n8n Workflow Architecture (Already Built)

```
Triggers: [Run Manually] OR [Daily 6am AEST]
    ↓
Read Prompts (Google Sheets)
    ↓
Limit 1 Prompt (for testing)
    ↓
Set Run Context (date, run_id)
    ↓
Parallel branches:
├── ChatGPT (GPT-5 Mini) — Responses API
├── Claude (Haiku 4.5) — Messages API
├── Gemini (3 Flash) — generateContent API
├── Perplexity (Sonar) — chat/completions API
├── Grok (Fast) — chat/completions API
├── NVIDIA (Nemotron) — chat/completions API
└── ValueSERP (Google) — search API + AI Overview
    ↓
Merge All Engines
    ↓
Normalize & Score
    ↓
Generate HTML Reports
    ↓
Push Reports to GitHub → moylal09/ai-visibility-reports
    ↓
Prepare Sheet Data
    ↓
Append to Runs (Google Sheets)
    ↓
Generate Index Page
    ↓
Push Index to GitHub
```

---

## Google Sheets Structure

**Sheet 1: Prompts**
| prompt_id | prompt_text | category | active |
|---|---|---|---|
| P001 | Best painting services in Melbourne | Brand | TRUE |

**Sheet 2: Runs** (auto-populated by n8n)
| run_id | run_date | prompt_id | engine | mentioned | position | sentiment | score | response_snippet | html_url |
|---|---|---|---|---|---|---|---|---|---|

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend Framework | Next.js 14 (App Router) |
| Styling | Tailwind CSS |
| Charts | Recharts |
| UI Components | shadcn/ui (dark theme) |
| Auth | Clerk or NextAuth.js |
| Database | Supabase (PostgreSQL) |
| Backend/API | Next.js API routes |
| Hosting | Vercel |
| Report Hosting | GitHub Pages (moylal09.github.io/ai-visibility-reports) |
| Workflow Engine | n8n (modernize.app.n8n.cloud) |

---

## Key Differentiators vs Competitors

1. **Multi-engine simultaneous tracking** — not just ChatGPT or just Google
2. **Google AI Overview included** via ValueSERP — most competitors miss this
3. **Amber/dark premium aesthetic** — stands out vs clinical white dashboards
4. **Credit system** — transparent, pay-as-you-go model
5. **Competitor logos auto-fetched** — visual, not just text
6. **Reports hosted on GitHub Pages** — shareable URLs for clients
7. **Countdown timer** — shows exactly when data refreshes

---

## Instructions for Claude Code

Build this app using:
- Next.js 14 (App Router)
- Tailwind CSS
- shadcn/ui dark theme
- Amber accent colour (#F59E0B)
- Dark navy background (#0A0F1E)

Start with:
1. Sidebar navigation component
2. Dashboard page with 4 KPI cards
3. Next scan countdown timer component
4. Visibility over time line chart

Data source: Google Sheets (via API) + Supabase for app data.
All scores, responses and run history come from the n8n workflow output stored in Google Sheets.
