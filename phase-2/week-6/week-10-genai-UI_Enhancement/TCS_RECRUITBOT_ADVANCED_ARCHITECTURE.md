# TCS RecruitBot — Advanced Retrieval & End-to-End Flow Architecture

## 1. Executive Summary & Objective

This architecture document extends the **TCS RecruitBot RAG Platform** by introducing 4 advanced retrieval capabilities into the existing frontend and backend without disrupting any existing flows:

1. **Deduplication (Phase A)**: Merge and deduplicate candidates across BM25 and Vector retrieval paths, preserving provenance tags (`bm25`, `vector`, `bm25 + vector`).
2. **Groq LLM Re-Ranking (Phase B)**: Re-score and re-order candidate pools using the Groq LLaMA model (`meta-llama/llama-4-scout-17b-16e-instruct`), explaining match reasoning.
3. **Candidate Fit Summarization (Phase C)**: Generate customized recruiter summaries (`short` or `detailed`) highlighting candidate strengths and skill gaps against the job query.
4. **Complete End-to-End Flow (Phase D)**: Unified one-click pipeline executing Vector + BM25 ➔ Deduplication ➔ LLM Re-Ranking ➔ Fit Summarization with comprehensive telemetry.

In addition, the entire user interface is upgraded to the **Tata Consultancy Services (TCS) corporate identity**, featuring the official TCS color palette, typography, organization logo emblem, and high-contrast enterprise dark styling.

---

## 2. System Architecture & End-to-End Pipeline

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│              TCS RecruitBot Web Application (React 18 + TS + Tailwind)          │
│                      [TCS Brand Theme: #001B44 | #0076CE | #00A3E0]             │
└────────────────────────────────────────┬────────────────────────────────────────┘
                                         │
 ┌───────────────────────────────────────┴───────────────────────────────────────┐
 │                               Search Mode Tabs                                │
 │  [Vector] [BM25] [Hybrid] │ [Deduplication] [Re-Rank] [Summarize] [End-to-End]│
 └───────┬──────────────┬───────────────┬─────────────┬───────────┬──────────────┘
         │              │               │             │           │
         ▼              ▼               ▼             ▼           ▼
┌──────────────────┐ ┌────────────────────┐ ┌───────────────────┐ ┌──────────────┐
│ POST             │ │ POST               │ │ POST              │ │ POST         │
│ /v1/search/vector│ │ /v1/search/bm25    │ │ /v1/search/dedup  │ │ /v1/search/  │
│                  │ │                    │ │ (NEW ENDPOINT)    │ │ rerank       │
└────────┬─────────┘ └──────────┬─────────┘ └─────────┬─────────┘ └──────┬───────┘
         │                      │                     │                  │
         └──────────────┬───────┴─────────────────────┘                  │
                        ▼                                                ▼
         ┌─────────────────────────────┐                  ┌──────────────────────┐
         │   Parallel BM25 + Vector    │                  │ Groq LLM Re-Ranking  │
         │   Candidate Retrieval       │                  │ (LLaMA-4 Scout 17B)  │
         └──────────────┬──────────────┘                  └──────────────┬───────┘
                        │                                                │
                        ▼                                                ▼
         ┌─────────────────────────────┐                  ┌──────────────────────┐
         │  Deduplication & Provenance │                  │ Candidate Fit        │
         │  mergeCandidates(bm25, vec) │                  │ Summarization Engine │
         └──────────────┬──────────────┘                  └──────────────┬───────┘
                        │                                                │
                        └───────────────────────┬────────────────────────┘
                                                ▼
                               ┌──────────────────────────────────┐
                               │ POST /v1/search (Full Pipeline)  │
                               │ End-to-End Candidate Dossier     │
                               └──────────────────────────────────┘
```

---

## 3. TCS Brand Identity & Design System

The application styling is aligned with the official **Tata Consultancy Services (TCS)** corporate design guidelines:

### Color Palette (TCS Corporate Dark Theme)
| Token | Hex Code | Usage |
|---|---|---|
| `--tcs-navy` | `#001B44` | Header, sidebar background, primary brand tone |
| `--tcs-blue` | `#0076CE` | Primary action buttons, active tabs, selected states |
| `--tcs-cyan` | `#00A3E0` | Brand secondary, highlights, pulsing indicators |
| `--tcs-bg-base` | `#040D1A` | Root viewport background |
| `--tcs-bg-surface`| `#08162B` | Sidebars, modal containers, top bars |
| `--tcs-bg-card` | `#0D223F` | Result cards, message bubbles, input panels |
| `--tcs-border` | `rgba(0, 163, 224, 0.15)` | Card and panel structural outlines |
| `--tcs-text` | `#F8FAFC` | Headings, candidate names, primary copy |
| `--tcs-muted` | `#94A3B8` | Subtitles, metadata labels, duration badges |
| `--tcs-accent` | `#68D2DF` | Special highlights, LLM reasoning badges |

### TCS Emblem & Logo Component
- Hexagonal / shield badge with official **TCS** typography and cyan-to-blue gradient ring.
- Organization header tag: `"TATA CONSULTANCY SERVICES · TalentAI RecruitBot"`.

---

## 4. Functionality Specifications & Backend Contracts

---

### Functionality 1: Deduplication (Backend & Frontend)

#### Objective
Ensure that candidates matching both keyword and vector criteria are merged into a single entry without duplicate candidate cards, while clearly exposing source provenance (`BM25 + Vector`, `Vector only`, `BM25 only`).

#### Backend Implementation
- **File**: `resume-rag-backend/src/modules/retrieval/controllers/retrievalController.ts`
- **Route**: `POST /v1/search/deduplicate` (registered in `retrievalRoutes.ts`)
- **Logic**:
  1. Executes `searchService.bm25Search()` and `searchService.vectorSearch()` in parallel.
  2. Passes both result arrays to `mergeCandidates(bm25, vector)` in `deduplicate.ts`.
  3. Returns unified candidates with dual scores (`bm25Score`, `vectorScore`) and `sources`.

#### API Contract (Postman)
- **Method**: `POST`
- **URL**: `http://localhost:3000/v1/search/deduplicate`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "query": "Selenium QA automation engineer 3 years",
    "topK": 10,
    "filters": {
      "minYearsExperience": 2
    }
  }
  ```
- **Expected Response (200 OK)**:
  ```json
  {
    "mode": "deduplicated",
    "query": "Selenium QA automation engineer 3 years",
    "count": 5,
    "results": [
      {
        "resumeId": "6a954f7bad2ba05e4d1ff0e6",
        "name": "Sharmila NKS",
        "role": "Senior Test Engineer",
        "company": "Tata Consultancy Services",
        "totalExperience": 4,
        "skills": ["Selenium", "Java", "TestNG", "Cucumber"],
        "sources": ["bm25", "vector"],
        "bm25Score": 18.4,
        "vectorScore": 0.912
      }
    ],
    "timings": {
      "bm25Ms": 65,
      "vectorMs": 280,
      "dedupMs": 2,
      "totalMs": 347
    }
  }
  ```

#### Frontend Presentation
- **Mode Tab**: `Deduplication` (Icon: `GitMerge` / `Layers`)
- **Card UI**: Provenance chips:
  - `BM25 + Vector`: Dual-matched highlighted in TCS Cyan
  - `Vector only`: Indigo tag
  - `BM25 only`: Pink tag

---

### Functionality 2: Groq LLM Re-Ranking

#### Objective
Apply generative reasoning to re-evaluate the relevance of candidate snippets against complex recruiter requirements, sorting candidates strictly by semantic role fit rather than pure statistical overlap.

#### Backend Implementation
- **Route**: `POST /v1/search/rerank` (already implemented in `retrievalRoutes.ts`)
- **Model**: `meta-llama/llama-4-scout-17b-16e-instruct` via Groq SDK
- **Logic**: Evaluates candidate snippets, scores each candidate from `0.00` to `1.00`, and provides a concise `reason`.

#### API Contract (Postman)
- **Method**: `POST`
- **URL**: `http://localhost:3000/v1/search/rerank`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "query": "Java backend developer with AWS and Microservices experience",
    "candidates": [
      {
        "resumeId": "6a9550a73a581b5166fa1c82",
        "snippet": "Jawahar Rajendran | Java Developer | 4 years | Spring Boot, Microservices, AWS, PostgreSQL"
      }
    ],
    "topK": 5
  }
  ```
- **Expected Response (200 OK)**:
  ```json
  {
    "results": [
      {
        "resumeId": "6a9550a73a581b5166fa1c82",
        "rank": 1,
        "relevanceScore": 0.95,
        "reason": "Strong match for both Java and AWS skills with 4 years of backend microservices experience."
      }
    ],
    "model": "meta-llama/llama-4-scout-17b-16e-instruct"
  }
  ```

#### Frontend Presentation
- **Mode Tab**: `LLM Re-Ranking` (Icon: `Sparkles`)
- **Card UI**:
  - Highlights LLM relevance score (`Re-Rank Score: 95%`)
  - Prominent **AI Reasoning Callout** with Sparkle icon: *"Strong match for both Java and AWS skills..."*

---

### Functionality 3: Candidate Fit Summarization

#### Objective
Provide instant, human-like recruiter summaries explaining exactly how well a candidate fits a role, detailing key strengths and potential gaps without requiring the recruiter to read the entire resume.

#### Backend Implementation
- **Route**: `POST /v1/search/summarize` (already implemented in `retrievalRoutes.ts`)
- **Options**: `style: "short" | "detailed"`, `maxTokens?: number`

#### API Contract (Postman)
- **Method**: `POST`
- **URL**: `http://localhost:3000/v1/search/summarize`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "query": "Java backend developer with AWS cloud",
    "candidate": {
      "resumeId": "6a9550a73a581b5166fa1c82",
      "snippet": "Jawahar Rajendran | Java Developer | 4 years exp | Spring Boot, AWS, Docker"
    },
    "style": "detailed"
  }
  ```
- **Expected Response (200 OK)**:
  ```json
  {
    "resumeId": "6a9550a73a581b5166fa1c82",
    "summary": "The candidate has 4 years of solid hands-on experience in Java and Spring Boot backend architectures. They have practical cloud deployment exposure on AWS and Docker, making them an immediate fit for the Senior Java Cloud developer opening."
  }
  ```

#### Frontend Presentation
- **Mode Tab**: `Summarization` (Icon: `FileText` / `Bot`)
- **Interactive Controls**: Toggle between `Short Bullet Summary` and `Detailed Dossier`.
- **Card UI**: Dedicated **Recruiter Fit Assessment Card** showing structured summary and evaluation points.

---

### Functionality 4: End-to-End Orchestrated Pipeline

#### Objective
Deliver the complete end-to-end recruiter workflow:
1. Recruiter enters a requirement.
2. System triggers parallel **BM25 + Vector Search**.
3. Results are merged and **deduplicated**.
4. Top candidate pool is **re-ranked by Groq LLM**.
5. Candidates receive an instant **AI Fit Summary**.
6. Recruiter clicks any card to view the complete **TCS Candidate Profile Modal**.

#### Backend Implementation
- **Route**: `POST /v1/search` (already implemented in `retrievalRoutes.ts` & `SearchService.ts`)
- **Body Options**: `{ query, filters, options: { finalTopK: 5, summarize: true, summaryStyle: "short" } }`

#### API Contract (Postman)
- **Method**: `POST`
- **URL**: `http://localhost:3000/v1/search`
- **Headers**: `Content-Type: application/json`
- **Request Body**:
  ```json
  {
    "query": "Java developer AWS",
    "options": {
      "finalTopK": 3,
      "summarize": true,
      "summaryStyle": "short"
    }
  }
  ```
- **Expected Response (200 OK)**:
  ```json
  {
    "query": "Java developer AWS",
    "results": [
      {
        "rank": 1,
        "resumeId": "6a97b9b64c03651438d4f433",
        "name": "THANGAMEENA",
        "role": "Associate Projects",
        "company": "Cognizant",
        "totalExperience": 4,
        "skills": ["Java", "Spring Boot", "AWS", "CI/CD"],
        "sources": ["bm25", "vector"],
        "relevanceScore": 0.95,
        "reason": "Strong match for both Java and AWS skills with 4 years of backend experience.",
        "summary": "Candidate possesses 4 years of active Java development experience with extensive AWS microservices exposure."
      }
    ],
    "timings": {
      "embeddingMs": 350,
      "bm25Ms": 65,
      "vectorMs": 280,
      "rerankMs": 910,
      "summarizeMs": 420,
      "totalMs": 2025
    },
    "degraded": false
  }
  ```

---

## 5. Phase-by-Phase Implementation Roadmap

| Phase | Module | Scope | API Verification |
|---|---|---|---|
| **Phase 1** | Backend Deduplication Endpoint | Create `POST /v1/search/deduplicate` in `retrievalController.ts` & `retrievalRoutes.ts`. | `POST /v1/search/deduplicate` |
| **Phase 2** | TCS Design System & UI Overhaul | Update Tailwind tokens with TCS `#001B44` / `#0076CE` palette, add TCS corporate logo emblem, brand header, and corporate styling. | Browser Visual Check |
| **Phase 3** | Navigation & SearchMode Expansion | Add Deduplication, Re-Rank, Summarize, and End-to-End buttons in `SearchModeNav.tsx` and Topbar. | Tab Switching Verification |
| **Phase 4** | Deduplication Frontend Flow | Connect `/v1/search/deduplicate`, render dual scores and source provenance tags (`BM25 + Vector`). | End-to-End Query Test |
| **Phase 5** | LLM Re-Ranking Frontend Flow | Connect `/v1/search/rerank`, render LLM ranking positions, relevance percentages, and AI reasoning pills. | End-to-End Query Test |
| **Phase 6** | Fit Summarization Frontend Flow | Connect `/v1/search/summarize`, add style selector (`short`/`detailed`), render generated fit summaries. | End-to-End Query Test |
| **Phase 7** | Full End-to-End Pipeline & Modal | Connect full orchestrated `/v1/search` with auto-summarization and TCS Candidate Profile Modal. | Full Recruiter Journey Test |

---

## 6. Generated Prompt for Phase-by-Phase Implementation

*(Use the prompt below to execute the implementation sequentially with token-efficient phase gates)*

```markdown
### SYSTEM IMPLEMENTATION PROMPT: TCS RecruitBot Advanced Retrieval & End-to-End Flow

Please implement the TCS RecruitBot Advanced Retrieval features phase-by-phase based on the architecture document `Frontend/TCS_RECRUITBOT_ADVANCED_ARCHITECTURE.md`.

Execution Rules:
1. Complete one phase at a time.
2. Provide the exact Request Body and Expected Response for Postman verification after each phase.
3. Pause and wait for user approval before moving to the next phase.
4. Strictly do not disturb existing Vector, BM25, Hybrid, or Ingestion flows.
5. Apply the official TCS corporate palette (#001B44, #0076CE, #00A3E0) and branding across the interface.

Roadmap:
- Phase 1: Implement Backend Deduplication Endpoint (`POST /v1/search/deduplicate`).
- Phase 2: Upgrade Frontend UI to TCS Corporate Theme (Logo, Colors, Header, Look & Feel).
- Phase 3: Expand Search Modes in Sidebar and Topbar (Deduplication, Re-Rank, Summarize, End-to-End).
- Phase 4: Implement Deduplication Frontend Flow with Source Provenance Badges.
- Phase 5: Implement LLM Re-Ranking Flow with AI Match Reasoning.
- Phase 6: Implement Fit Summarization Flow with Short/Detailed Toggle.
- Phase 7: Implement Complete End-to-End Pipeline with Full Candidate Dossier & Modal.
```
