# 🔌 Tripzy Intelligence Platform - Plugin Architecture

> **Document Version:** 1.0  
> **Last Updated:** December 25, 2025  
> **Status:** Future Development (Documented for Reference)

---

## 📋 Overview

This document outlines a **modular, plug-and-play architecture** for the Tripzy 3-Layer Intelligence system. The goal is to create reusable components that can be dropped into any project to add AI-powered personalization.

---

## 🎯 Vision

> _"Install one package, get intelligent personalization in any app."_

Instead of rebuilding the 3-layer architecture for each project, we create:

- **Reusable SDK** for signal collection
- **Standalone Agent Service** for AI reasoning
- **Database Schema Module** for instant setup

---

## 📦 The Three Modules

### Module 1: `@tripzy/signal-sdk` (NPM Package)

**Purpose:** Collect user behavior signals in any web application.

**Features:**

- Framework agnostic (React, Vue, Next.js, vanilla JS)
- Auto-tracks: page views, scroll depth, reading time, clicks
- Configurable tracking options
- Event buffering and batching
- Offline support with queue

**Installation:**

```bash
npm install @tripzy/signal-sdk
```

**Usage:**

```typescript
import { TripzySignals } from "@tripzy/signal-sdk";

const tripzy = new TripzySignals({
  projectId: "my-project",
  supabaseUrl: process.env.SUPABASE_URL,
  supabaseKey: process.env.SUPABASE_ANON_KEY,

  tracking: {
    pageViews: true,
    scrollDepth: true,
    readingTime: true,
    clicks: true,
    searches: true,
  },

  bufferSize: 10,
  flushInterval: 5000,
});

tripzy.init();

// Manual tracking
tripzy.track("custom_event", { key: "value" });
```

**Signals Collected:**

| Signal         | Description               | Auto-Tracked |
| -------------- | ------------------------- | ------------ |
| `page_view`    | User visited a page       | ✅ Yes       |
| `scroll_depth` | How far user scrolled (%) | ✅ Yes       |
| `reading_time` | Time spent on page        | ✅ Yes       |
| `click`        | Element clicks            | ✅ Yes       |
| `search`       | Search queries            | ✅ Yes       |
| `custom`       | Any custom event          | Manual       |

---

### Module 2: `tripzy-agents` (Docker Service)

**Purpose:** Process signals and generate AI-powered recommendations.

**Features:**

- Autonomous AI agents for different tasks
- Gemini/OpenAI compatible
- REST API for recommendations
- Cross-domain intelligence transfer
- Explainable recommendations

**Deployment:**

```yaml
# docker-compose.yml
version: "3.8"
services:
  tripzy-agents:
    image: tripzy/agents:latest
    environment:
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_SERVICE_KEY=${SUPABASE_SERVICE_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    ports:
      - "8000:8000"
```

**API Endpoints:**

| Endpoint     | Method | Description                          |
| ------------ | ------ | ------------------------------------ |
| `/recommend` | GET    | Get personalized recommendations     |
| `/analyze`   | POST   | Analyze content, generate embeddings |
| `/profile`   | GET    | Get user preference profile          |
| `/transfer`  | POST   | Cross-domain intelligence transfer   |

**Agents Included:**

| Agent                 | Purpose                                                   |
| --------------------- | --------------------------------------------------------- |
| **Content Agent**     | Analyzes content, generates embeddings, extracts entities |
| **Profile Agent**     | Builds/updates user preference vectors                    |
| **CrossDomain Agent** | Transfers intelligence between projects                   |
| **Reasoning Agent**   | Gemini-powered explainable recommendations                |

---

### Module 3: `@tripzy/schema` (CLI Tool)

**Purpose:** Set up required database tables in any Supabase project.

**Features:**

- One-command schema setup
- pgvector extension configuration
- RLS policies included
- Idempotent migrations

**Installation:**

```bash
npm install -g @tripzy/schema
```

**Usage:**

```bash
# Initialize schema in Supabase project
tripzy-schema init \
  --project-id=your-supabase-project \
  --enable-vectors \
  --enable-signals

# Export as SQL migration
tripzy-schema export > migrations/001_tripzy_intelligence.sql
```

**Tables Created:**

| Table                        | Purpose                        |
| ---------------------------- | ------------------------------ |
| `tripzy.user_signals`        | Stores all tracked events      |
| `tripzy.user_profiles`       | User preference vectors        |
| `tripzy.content_embeddings`  | Content semantic vectors       |
| `tripzy.recommendations_log` | Audit trail of recommendations |

---

## 🏗️ Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ANY PROJECT (Blog, Deals, Mobile, etc.)                  │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                     @tripzy/signal-sdk                              │  │
│   │                                                                      │  │
│   │  • Auto-tracks user behavior                                        │  │
│   │  • Buffers and sends signals                                        │  │
│   │  • Works in any JavaScript environment                              │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                    │                                        │
│                                    ▼                                        │
└────────────────────────────────────┼────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TRIPZY AGENT SERVICE                                │
│                         (Centralized Deployment)                            │
│                                                                             │
│   ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌───────────────────────┐  │
│   │  Content  │  │  Profile  │  │ CrossDom  │  │  Reasoning (Gemini)   │  │
│   │   Agent   │  │   Agent   │  │   Agent   │  │                       │  │
│   └───────────┘  └───────────┘  └───────────┘  └───────────────────────┘  │
│                                                                             │
│   REST API: /recommend, /analyze, /profile, /transfer                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                              SUPABASE                                       │
│                         (Shared Database)                                   │
│                                                                             │
│   ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────┐  │
│   │ user_signals  │  │ user_profiles │  │ content_embeddings (pgvector) │  │
│   └───────────────┘  └───────────────┘  └───────────────────────────────┘  │
│                                                                             │
│   @tripzy/schema - One-command setup                                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📁 Repository Structure (Future)

```
tripzy-intelligence/
├── packages/
│   ├── signal-sdk/              # @tripzy/signal-sdk
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tracker.ts
│   │   │   ├── buffer.ts
│   │   │   └── types.ts
│   │   ├── package.json
│   │   └── README.md
│   │
│   ├── schema/                  # @tripzy/schema
│   │   ├── migrations/
│   │   │   ├── 001_signals.sql
│   │   │   ├── 002_embeddings.sql
│   │   │   └── 003_profiles.sql
│   │   ├── src/cli.ts
│   │   └── package.json
│   │
│   └── react-hooks/             # @tripzy/react (optional)
│       ├── src/
│       │   ├── useSignals.ts
│       │   └── useRecommendations.ts
│       └── package.json
│
├── services/
│   └── agents/                  # tripzy-agents
│       ├── app/
│       │   ├── main.py
│       │   ├── agents/
│       │   │   ├── content_agent.py
│       │   │   ├── profile_agent.py
│       │   │   └── crossdomain_agent.py
│       │   └── routers/
│       │       ├── recommend.py
│       │       └── analyze.py
│       ├── Dockerfile
│       └── requirements.txt
│
├── docs/
│   ├── getting-started.md
│   ├── signal-sdk.md
│   ├── agents-api.md
│   └── schema-setup.md
│
└── examples/
    ├── nextjs-integration/
    ├── react-integration/
    └── vue-integration/
```

---

## 🚀 Implementation Phases

### Phase 1: Signal SDK (2-3 days)

- [ ] Create monorepo structure
- [ ] Build core tracker functionality
- [ ] Implement event buffering
- [ ] Add Supabase integration
- [ ] Publish to npm

### Phase 2: Schema Module (1-2 days)

- [ ] Create migration files
- [ ] Build CLI tool
- [ ] Add pgvector setup
- [ ] Include RLS policies
- [ ] Publish to npm

### Phase 3: Agent Service (1-2 weeks)

- [ ] FastAPI service setup
- [ ] Content Agent implementation
- [ ] Profile Agent implementation
- [ ] CrossDomain Agent implementation
- [ ] Gemini reasoning integration
- [ ] Docker deployment

### Phase 4: Documentation & Examples

- [ ] Complete API documentation
- [ ] Integration examples
- [ ] Best practices guide

---

## 🎯 Benefits Summary

| Benefit                  | Description                               |
| ------------------------ | ----------------------------------------- |
| **Plug & Play**          | Add to any project with one npm install   |
| **Unified Profiles**     | User understanding across all Tripzy apps |
| **Scale Once**           | Single agent service handles all projects |
| **Framework Agnostic**   | Works with React, Vue, Next.js, etc.      |
| **Progressive Adoption** | Start simple, add features as needed      |
| **Self-Hosted Option**   | Run on your own infrastructure            |

---

## 📝 Notes

- This is a **future development** plan
- Current priority is getting the blog functional
- Will implement after core blog features are complete
- Signal SDK should be first module built (provides immediate value)

---

<div align="center">

**Status: Documented for Future Implementation**

_Build once, use everywhere._

</div>
