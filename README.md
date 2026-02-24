┌─────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL SOURCES                              │
│  Regulation Scraper → Kafka "regulation.new"                        │
│  GitHub Webhooks    → spec.pr_merged / spec.pr_reviewed             │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      ASSURE CODE HUB                                 │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                    KAFKA EVENT BUS                           │    │
│  │  Topics:                                                     │    │
│  │  → regulation.new / regulation.updated                      │    │
│  │  ← spec.created / spec.updated                              │    │
│  │  → spec.pr_requested                                        │    │
│  │  ← spec.pr_created                                          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│          │                           │                               │
│          ▼                           ▼                               │
│  ┌───────────────────┐    ┌─────────────────────────────────┐       │
│  │ Regulation Impact │    │          BullMQ Queues           │       │
│  │    Analyzer       │    │  spec-patch (concurrency: 3)     │       │
│  │                   │    │  github-pr  (concurrency: 2)     │       │
│  │ Which workspaces  │    │  compliance-job (from API)       │       │
│  │ are affected?     │    └──────────────────┬──────────────┘       │
│  └────────┬──────────┘                       │                      │
│           │ Per-spec BullMQ job              │                      │
│           ▼                                  ▼                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  CLAUDE DIFF ENGINE                          │    │
│  │  1. detectAffectedModules() — which of 5 modules?           │    │
│  │  2. generateModuleDiffs()   — before/after per clause       │    │
│  │  3. applyDiffsToSpec()      — minimal patch only            │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              5-MODULE SPEC VERSION (Versioned)               │    │
│  │                                                              │    │
│  │  Module 1: Master Specification   (projectSummary, flows)   │    │
│  │  Module 2: Security Blueprint     (encryption, IAM, audit)  │    │
│  │  Module 3: Cost Analysis          (breakdown, premium)      │    │
│  │  Module 4: Tech Stack Justification (decisions, alternatives│    │
│  │  Module 5: Code Scaffolding       (Dockerfile, CI, prompts) │    │
│  │                                                              │    │
│  │  parent_id chain: v1.0.0 → v1.1.0 → v1.2.0 → ...          │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                              │                                       │
│                              ▼                                       │
└──────────────────────────────┼──────────────────────────────────────┘
                               │
                               ▼ GitHub App (Octokit)
┌─────────────────────────────────────────────────────────────────────┐
│                        GITHUB                                        │
│                                                                      │
│  Branch: compliance/gdpr-article-32-v1.1.0                          │
│  Files committed:                                                    │
│    specs/01-master-specification.md                                  │
│    specs/02-security-blueprint.md       ← amended clauses only      │
│    specs/03-cost-analysis.md                                         │
│    specs/04-tech-stack-justification.md                              │
│    specs/05-code-scaffolding.md                                      │
│    specs/compliance-diff-summary.md     ← before/after diff         │
│    Dockerfile                           ← from Module 5             │
│    docker-compose.yml                   ← from Module 5             │
│    .github/workflows/compliance-ci.yml  ← from Module 5            │
│                                                                      │
│  PR assigned to: Developer (from workspace_members)                  │
│  Labels: compliance, automated, gdpr                                │
└─────────────────────────────────────────────────────────────────────┘
