# TCS RecruitBot · Master Implementation Prompt

Use this standalone prompt to reproduce, execute, or prompt an AI developer to implement the complete TCS RecruitBot Advanced Retrieval and Corporate Branding suite.

---

```markdown
### SYSTEM IMPLEMENTATION PROMPT: TCS RecruitBot Advanced Retrieval & End-to-End Flow

Please implement the TCS RecruitBot Advanced Retrieval features phase-by-phase based on the architecture document `Frontend/TCS_RECRUITBOT_ADVANCED_ARCHITECTURE.md`.

Execution Rules:
1. Complete one phase at a time.
2. Provide the exact Request Body and Expected Response for Postman verification after each phase.
3. Pause and wait for user approval before moving to the next phase.
4. Strictly do not disturb existing Vector, BM25, Hybrid, or Ingestion flows.
5. Apply the official TCS corporate palette (#001B44, #0076CE, #00A3E0, #040D1A, #08162B) and branding emblem across the interface.

Roadmap:
- Phase 1: Implement Backend Deduplication Endpoint (`POST /v1/search/deduplicate`) with Smart Candidate-Identity Deduplication (cross-index + candidate identity via verified email and normalized name).
- Phase 2: Upgrade Frontend UI to TCS Corporate Theme (Official SVG Shield Logo, Enterprise Dark Palette, Header, Look & Feel).
- Phase 3: Expand Search Modes in Sidebar and Topbar (Vector, BM25, Hybrid, Deduplication, LLM Re-Ranking, Fit Summarization, End-to-End Flow).
- Phase 4: Implement Deduplication Frontend Flow with Source Provenance Badges (`BM25 + Vector (Deduped)`) and dual score breakdown.
- Phase 5: Implement LLM Re-Ranking Frontend Flow with Groq AI Match Reasoning Callouts and rank badges.
- Phase 6: Implement Fit Summarization Flow with Recruiter Assessment Dossier card.
- Phase 7: Implement Complete End-to-End Pipeline Integration (`POST /v1/search` with auto-summarization and multi-stage telemetry timings).
```
