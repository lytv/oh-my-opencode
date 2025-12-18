---
description: "Comprehensive codebase onboarding - Auto-generates AGENTS.md, rules, docs, and configurations"
---

# Onboard Codebase Command

**Automatically analyze codebase and generate all necessary context files for Oh My OpenCode agents.**

This command performs deep codebase analysis and generates:
- ✅ AGENTS.md (project-wide + per-module)
- ✅ Conditional rules (.claude/rules/)
- ✅ README.md / ARCHITECTURE.md
- ✅ LSP configuration
- ✅ Oh My OpenCode settings

## Usage

```bash
/onboard-codebase [--scope full|basic] [--skip-docs] [--dry-run]
```

## Flags

- `--scope full|basic` - Analysis depth (default: full)
  - `basic`: Quick scan + essential AGENTS.md only (~3 min)
  - `full`: Complete analysis + all documentation (~12 min)

- `--skip-docs` - Skip README/ARCHITECTURE generation (only AGENTS.md + rules)

- `--dry-run` - Preview what will be generated without creating files

- `--force` - Overwrite existing files (default: merge/update)

- `--modules <path1,path2>` - Only analyze specific modules (comma-separated)

## What This Command Does

### 🎯 Execution Workflow (Multi-Phase with Background Agents)

**CRITICAL: You MUST use background agents for parallel execution to achieve 12-minute completion time.**

---

#### **PHASE 1: Quick Project Scan (30 seconds)**

**Agent:** `@explore` (Grok - blazing fast)

**Tasks:**
1. Scan entire file structure with `glob` tool
2. Detect package managers and read dependency files:
   ```
   - package.json / package-lock.json / yarn.lock / pnpm-lock.yaml (Node.js)
   - requirements.txt / Pipfile / pyproject.toml (Python)
   - go.mod / go.sum (Go)
   - Cargo.toml (Rust)
   - composer.json (PHP)
   - pom.xml / build.gradle (Java)
   - Gemfile (Ruby)
   ```
3. Identify config files:
   ```
   - tsconfig.json, jsconfig.json
   - .prettierrc, .eslintrc
   - pytest.ini, jest.config.js
   - docker-compose.yml, Dockerfile
   - .env.example
   - Database configs (prisma/schema.prisma, knexfile.js, etc)
   ```
4. Find entry points:
   ```
   - index.js, main.ts, app.py, main.go
   - src/index.*, src/main.*
   - server.js, server.ts
   ```
5. Count statistics:
   ```
   - Total files
   - Files by extension
   - Lines of code (approximate)
   - Directory depth
   ```

**Output:** `project_metadata.json`
```json
{
  "name": "detected-project-name",
  "type": "monorepo|single-app",
  "packageManager": "npm|yarn|pnpm|pip|go|cargo",
  "entryPoints": ["src/index.ts", "server.ts"],
  "configFiles": [...],
  "statistics": {
    "totalFiles": 1234,
    "byExtension": { "ts": 456, "tsx": 123, ... },
    "estimatedLOC": 45000
  }
}
```

**Time:** 30 seconds
**Status:** ✅ Complete → Proceed to Phase 2

---

#### **PHASE 2: Tech Stack Detection (2 minutes - PARALLEL)**

**⚠️ CRITICAL: Spawn 3 background agents in parallel!**

**Agent 1: `@librarian` (Background) - Dependency Analysis**

Tasks:
1. Parse all dependency files found in Phase 1
2. Identify all frameworks and libraries:
   ```
   Frontend: React, Vue, Angular, Svelte, Next.js, Nuxt
   Backend: Express, Fastify, NestJS, Django, Flask, FastAPI, Gin, Actix
   Database: PostgreSQL, MySQL, MongoDB, Redis, SQLite
   ORM: Prisma, TypeORM, Sequelize, Drizzle, SQLAlchemy, GORM
   Testing: Jest, Vitest, Pytest, Go test, Cargo test
   State: Redux, Zustand, Pinia, Vuex, MobX
   Styling: Tailwind, styled-components, CSS Modules, SASS
   ```
3. Check version compatibility and detect outdated dependencies
4. Identify development vs production dependencies
5. Detect monorepo tools (Turborepo, Nx, Lerna, pnpm workspaces)

Output: `tech_stack.json`

**Agent 2: `@explore` (Background) - Code Pattern Detection**

Tasks:
1. Use `ast-grep` to find import/require patterns:
   ```
   - React imports: import React from 'react'
   - Vue imports: import { createApp } from 'vue'
   - Express patterns: const express = require('express')
   - Database clients: import { PrismaClient } from '@prisma/client'
   ```
2. Detect ORM patterns by searching for schema files
3. Find database connection configs
4. Identify API route patterns (REST, GraphQL, tRPC)
5. Detect authentication patterns (JWT, Session, OAuth)
6. Find testing file patterns (*.test.*, *.spec.*, __tests__/)

Output: `code_patterns.json`

**Agent 3: `@oracle` (Background) - Architecture Analysis**

Tasks:
1. Analyze project structure to determine architecture:
   ```
   Monorepo indicators:
   - packages/, apps/, libs/ directories
   - Workspace configs (pnpm-workspace.yaml, lerna.json)

   Microservices indicators:
   - Multiple server entry points
   - docker-compose with multiple services
   - Separate deployment configs

   Monolith indicators:
   - Single entry point
   - Unified directory structure
   ```
2. Identify design patterns:
   ```
   - MVC (models, views, controllers directories)
   - Repository pattern (repositories/)
   - Service layer (services/)
   - Clean architecture (domain/, application/, infrastructure/)
   ```
3. Analyze code organization quality
4. Detect separation of concerns

Output: `architecture.json`

**Synchronization Point:** Wait for all 3 agents to complete

**Time:** 2 minutes (parallel execution)
**Status:** ✅ All agents complete → Synthesize results → Proceed to Phase 3

---

#### **PHASE 3: Deep Analysis (5 minutes - PARALLEL)**

**⚠️ CRITICAL: Spawn 3 background agents in parallel!**

**Agent 1: `@librarian` (Background) - Documentation Audit**

Tasks:
1. Find and read all existing documentation:
   ```
   - README.md (all levels)
   - CONTRIBUTING.md
   - ARCHITECTURE.md
   - docs/ directory
   - API documentation
   - Inline code comments (JSDoc, docstrings)
   ```
2. Extract existing information:
   ```
   - Setup instructions
   - Environment variables
   - API endpoints documentation
   - Architecture diagrams
   - Known issues
   ```
3. Identify documentation gaps:
   ```
   - Missing README in modules
   - Undocumented API endpoints
   - No setup instructions
   - Missing architecture overview
   ```
4. Use `context7` MCP to find official docs for detected frameworks

Output: `documentation_audit.json`

**Agent 2: `@oracle` (Background) - Code Quality Analysis**

Tasks:
1. Run LSP diagnostics on key files (if LSP available)
2. Analyze code complexity:
   ```
   - Find large files (>500 lines)
   - Detect complex functions (cyclomatic complexity)
   - Identify code smells
   ```
3. Security scan:
   ```
   - Search for potential secrets (.env files committed)
   - Find TODO/FIXME comments
   - Detect deprecated patterns
   - Check for unsafe dependencies
   ```
4. Find technical debt:
   ```
   - Commented out code
   - Console.log statements
   - Duplicate code patterns
   - Missing error handling
   ```

Output: `quality_report.json`

**Agent 3: `@explore` (Background) - Workflow Mapping**

Tasks:
1. Trace main user workflows:
   ```
   - Find route definitions (Express routes, Next.js pages, etc)
   - Map frontend routes (React Router, Vue Router)
   - Identify API endpoints
   ```
2. Map authentication flow:
   ```
   - Login/register endpoints
   - Token validation middleware
   - Session management
   - OAuth providers
   ```
3. Database schema analysis:
   ```
   - Read Prisma schema / TypeORM entities / Mongoose schemas
   - Identify relationships
   - Find migrations
   ```
4. Find background jobs/workers:
   ```
   - Bull queues
   - Cron jobs
   - Event listeners
   ```

Output: `workflows.json`

**Synchronization Point:** Wait for all 3 agents to complete

**Time:** 5 minutes (parallel execution)
**Status:** ✅ All agents complete → Proceed to Phase 4

---

#### **PHASE 4: Content Generation (3 minutes - PARALLEL)**

**⚠️ CRITICAL: Spawn 3 background agents in parallel!**

**Agent 1: `@OmO` (Background) - AGENTS.md Generation**

Tasks:
1. **Generate root AGENTS.md** with synthesized information:
   ```markdown
   # Project Name - Context for AI Agents

   ## Project Overview
   [Generated from Phase 1-3 data]
   - Architecture: [from architecture.json]
   - Tech Stack: [from tech_stack.json]
   - Size: [from project_metadata.json]

   ## Tech Stack
   [Detailed breakdown from tech_stack.json]

   ## Project Structure
   [From project_metadata.json + architecture.json]

   ## Important Conventions
   [Detected from code_patterns.json]

   ## Critical Files
   [Entry points + key configs from Phase 1]

   ## Workflows
   [From workflows.json - authentication, API flows, etc]

   ## Common Tasks
   [Extracted from package.json scripts]

   ## Known Issues / Tech Debt
   [From quality_report.json]

   ## External Services
   [Detected from .env.example and configs]
   ```

2. **Generate per-module AGENTS.md** (if monorepo/modular):
   ```
   For each major module in apps/ or packages/:
   - Create module-specific AGENTS.md
   - Include module purpose, dependencies, structure
   - Document module-specific patterns
   ```

3. **Merge strategy** if AGENTS.md exists:
   ```
   - Preserve user-written content
   - Update detected sections
   - Add new sections for newly discovered patterns
   - Mark auto-generated sections with <!-- AUTO-GENERATED -->
   ```

Output: `AGENTS.md` + per-module `AGENTS.md` files

**Agent 2: `@document-writer` (Background) - Documentation Generation**

Tasks:
1. **Generate/Update README.md** (if missing or incomplete):
   ```markdown
   # Project Name

   ## Description
   [Inferred from package.json or existing README]

   ## Tech Stack
   [Summary from tech_stack.json]

   ## Getting Started

   ### Prerequisites
   [From dependencies]

   ### Installation
   ```bash
   # Detected package manager
   npm install  # or yarn, pnpm, etc
   ```

   ### Environment Variables
   [From .env.example]

   ### Running the Application
   ```bash
   # From package.json scripts
   npm run dev
   ```

   ## Project Structure
   [Overview from architecture.json]

   ## API Documentation
   [Endpoints from workflows.json]

   ## Testing
   ```bash
   npm test
   ```

   ## Deployment
   [From Dockerfile, docker-compose, or deployment configs]
   ```

2. **Generate ARCHITECTURE.md** (if scope=full):
   ```markdown
   # Architecture Overview

   ## System Architecture
   [Diagram in text/mermaid format from architecture.json]

   ## Design Patterns
   [From architecture.json]

   ## Data Flow
   [From workflows.json]

   ## Database Schema
   [From schema analysis]

   ## Authentication & Authorization
   [From auth flow analysis]

   ## External Integrations
   [Detected services]
   ```

3. **Merge strategy** if files exist:
   ```
   - Preserve existing content
   - Update outdated sections
   - Add missing sections
   ```

Output: `README.md`, `ARCHITECTURE.md`

**Agent 3: `@OmO` (Background) - Conditional Rules Generation**

Tasks:
1. **Create language-specific rules** in `.claude/rules/`:

   For TypeScript projects:
   ```markdown
   # .claude/rules/typescript.md
   ---
   globs: ["*.ts", "*.tsx"]
   description: "TypeScript coding standards for this project"
   ---

   # TypeScript Standards

   [Detected from code_patterns.json]:
   - Interface naming: [Detected pattern]
   - Type vs Interface usage: [Detected preference]
   - Null handling: [Detected pattern]
   - Import style: [Relative vs absolute, detected]
   ```

   For React projects:
   ```markdown
   # .claude/rules/react.md
   ---
   globs: ["*.tsx", "*.jsx"]
   description: "React component standards"
   ---

   # React Standards

   [Detected patterns]:
   - Component style: [Functional vs Class, detected]
   - Props typing: [Pattern detected]
   - State management: [Redux, Zustand, Context - detected]
   - Styling approach: [Tailwind, styled-components - detected]
   ```

2. **Create framework-specific rules**:
   ```
   - Express.js: API route patterns, middleware patterns
   - Next.js: Page routing, API routes, data fetching patterns
   - NestJS: Module structure, dependency injection patterns
   - Django: App structure, view patterns
   ```

3. **Create database rules**:
   ```markdown
   # .claude/rules/database.md
   ---
   globs: ["prisma/**/*", "*.prisma", "src/models/**/*"]
   description: "Database schema standards"
   ---

   [Detected from schema]:
   - Naming convention: [snake_case, camelCase - detected]
   - ID strategy: [UUID, auto-increment - detected]
   - Timestamp fields: [created_at, updated_at - detected]
   - Migration strategy: [Detected from migrations/]
   ```

4. **Create testing rules**:
   ```markdown
   # .claude/rules/testing.md
   ---
   globs: ["*.test.*", "*.spec.*", "__tests__/**/*"]
   description: "Testing standards"
   ---

   [Detected]:
   - Test framework: [Jest, Vitest, Pytest]
   - Test organization: [Co-located, separate __tests__/]
   - Naming convention: [Detected pattern]
   - Coverage requirements: [From config if present]
   ```

Output: Multiple `.claude/rules/*.md` files

**Synchronization Point:** Wait for all 3 agents to complete

**Time:** 3 minutes (parallel execution)
**Status:** ✅ All content generated → Proceed to Phase 5

---

#### **PHASE 5: Configuration Setup (1 minute)**

**Agent:** `@OmO` (Main thread)

Tasks:
1. **Configure LSP servers** for detected languages:
   ```json
   // .opencode/oh-my-opencode.json or ~/.config/opencode/oh-my-opencode.json
   {
     "lsp": {
       "typescript-language-server": {
         "command": ["typescript-language-server", "--stdio"],
         "extensions": [".ts", ".tsx"],
         "priority": 10
       },
       // ... other LSP servers based on detected languages
     }
   }
   ```

2. **Recommend MCP servers** based on tech stack:
   ```
   - context7: For framework documentation
   - websearch_exa: For latest best practices
   - grep_app: For finding implementation examples
   ```

3. **Create project-specific Oh My OpenCode config**:
   ```json
   // .opencode/oh-my-opencode.json
   {
     "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",

     "agents": {
       // Customize agents based on project type
       "frontend-ui-ux-engineer": {
         "model": "google/gemini-3-pro-preview",
         // If React project with Tailwind detected
       },
       "oracle": {
         "model": "openai/gpt-5.2",
         // For architecture and debugging
       }
     },

     "lsp": {
       // LSP configs from step 1
     },

     "disabled_hooks": [],
     "disabled_mcps": []
   }
   ```

Output: Configuration files

**Time:** 1 minute
**Status:** ✅ Configuration complete → Proceed to Phase 6

---

#### **PHASE 6: Validation & Reporting (30 seconds)**

**Agent:** `@OmO` (Main thread)

Tasks:
1. **Verify all generated files**:
   ```bash
   - AGENTS.md exists and has content
   - .claude/rules/ has appropriate rule files
   - README.md updated/created
   - ARCHITECTURE.md created (if full scope)
   - Configurations valid JSON
   ```

2. **Quick validation checks**:
   ```
   - AGENTS.md mentions detected tech stack
   - Rules match actual file extensions in project
   - No placeholder text left in templates
   - All paths in configs are valid
   ```

3. **Generate comprehensive onboarding report**:
   ```markdown
   # Codebase Onboarding Complete! 🎉

   ## Summary

   ✅ Project analyzed: [name]
   ✅ Tech stack detected: [summary]
   ✅ Files generated: [count]
   ✅ Agents configured: [list]

   ## Generated Files

   📄 AGENTS.md (project root)
   📄 apps/frontend/AGENTS.md
   📄 apps/backend/AGENTS.md
   📋 .claude/rules/typescript.md
   📋 .claude/rules/react.md
   📋 .claude/rules/database.md
   📋 .claude/rules/testing.md
   📖 README.md (updated)
   📖 ARCHITECTURE.md (created)
   ⚙️  .opencode/oh-my-opencode.json (created)

   ## Tech Stack Detected

   **Frontend:**
   - React 18.2.0
   - TypeScript 5.0
   - Tailwind CSS 3.3
   - Vite 4.3

   **Backend:**
   - Node.js 20.x
   - Express 4.18
   - Prisma ORM 5.0
   - PostgreSQL 15

   **Testing:**
   - Jest 29.5
   - React Testing Library 14.0

   **DevOps:**
   - Docker
   - Docker Compose

   ## Architecture Insights

   - Type: Monorepo (Turborepo)
   - Structure: apps/ + packages/
   - Pattern: Service layer architecture
   - API Style: REST
   - Auth: JWT with refresh tokens

   ## Code Quality

   ✅ TypeScript strict mode enabled
   ✅ Good test coverage (78%)
   ⚠️  Some files >500 lines (consider refactoring)
   ⚠️  3 TODO comments found
   ✅ No security issues detected

   ## Recommendations

   1. **Immediate Actions:**
      - Review generated AGENTS.md for accuracy
      - Add project-specific details to AGENTS.md
      - Review .claude/rules/ and adjust if needed

   2. **Optional Improvements:**
      - Refactor UserService.ts (503 lines)
      - Add rate limiting to auth endpoints
      - Update Express to 4.19 (current: 4.18)
      - Consider adding API documentation (Swagger)

   3. **Next Steps:**
      - Start working: Just type your task!
      - Agents now understand your codebase
      - They'll auto-inject context from AGENTS.md
      - Conditional rules will apply automatically

   ## Example Usage

   ```
   # Agents now understand your project!

   You: "Add rate limiting to auth endpoints"
   → OmO reads AGENTS.md, knows auth flow
   → Implements with proper conventions
   → Follows detected patterns

   You: "Refactor UserService.ts"
   → Oracle analyzes with LSP tools
   → Proposes clean architecture
   → Maintains test compatibility

   You: "Build a new dashboard component"
   → Frontend agent knows React + Tailwind stack
   → Follows component patterns from rules
   → Uses shadcn/ui (detected dependency)
   ```

   ## Files You Should Review

   🔍 **Priority 1 (Must Review):**
   - [ ] `AGENTS.md` - Add project-specific context
   - [ ] `.env.example` - Verify all env vars documented

   🔍 **Priority 2 (Should Review):**
   - [ ] `.claude/rules/*.md` - Adjust coding standards
   - [ ] `ARCHITECTURE.md` - Add architecture decisions

   🔍 **Priority 3 (Nice to Have):**
   - [ ] `README.md` - Add contributing guidelines
   - [ ] Module AGENTS.md - Add module-specific details

   ## Token Usage

   - Total: ~25,000 tokens
   - Time: 12 minutes
   - Agents used: 7 (3 parallel phases)

   ## 🎯 What's Next?

   Your codebase is now fully onboarded! Agents have:
   ✅ Complete understanding of architecture
   ✅ Tech stack knowledge
   ✅ Coding conventions and patterns
   ✅ Access to IDE-grade tools (LSP)
   ✅ Framework-specific rules

   **You can now:**
   - Ask agents to implement features
   - Request refactoring with confidence
   - Debug with full context
   - Generate code following your patterns

   **Agents will automatically:**
   - Read AGENTS.md when working on files
   - Apply conditional rules based on file type
   - Use LSP for safe refactoring
   - Follow detected conventions

   ---

   **Need help?** Just ask! Agents are ready to work. 🚀
   ```

Output: Comprehensive report displayed to user

**Time:** 30 seconds
**Status:** ✅ ONBOARDING COMPLETE!

---

### 📊 Total Timeline

```
Phase 1: Quick Scan          → 30s
Phase 2: Tech Stack (||)     → 2min
Phase 3: Deep Analysis (||)  → 5min
Phase 4: Content Gen (||)    → 3min
Phase 5: Configuration       → 1min
Phase 6: Validation          → 30s
─────────────────────────────────────
Total:                        ~12min

(|| = Parallel execution with background agents)
```

---

## 🎯 Implementation Instructions for AI Agent

### **CRITICAL REQUIREMENTS:**

1. **YOU MUST use background agents for parallel execution**
   ```
   Bad: Sequential execution (30+ minutes)
   Good: Spawn @librarian, @explore, @oracle in parallel (12 minutes)
   ```

2. **YOU MUST use Sequential MCP for complex reasoning**
   ```
   When synthesizing data from multiple sources
   When making architecture decisions
   When generating high-quality AGENTS.md content
   ```

3. **YOU MUST preserve existing user content**
   ```
   - Never overwrite user-written content
   - Merge/update approach
   - Mark auto-generated sections
   ```

4. **YOU MUST validate generated content**
   ```
   - No placeholder text
   - No generic templates
   - Project-specific content only
   - Accurate tech stack detection
   ```

5. **YOU MUST handle errors gracefully**
   ```
   - If background agent fails → Continue with available data
   - If file exists and --force not set → Merge
   - If tech stack unclear → Ask user for clarification
   ```

---

## 🚀 Advanced Features

### **Smart Templates**

Based on detected project type, use specialized templates:

**React + TypeScript + Tailwind:**
```
- Emphasize component patterns
- Include Tailwind conventions
- Add TypeScript best practices
- Note state management approach
```

**Express + Prisma + PostgreSQL:**
```
- Emphasize API structure
- Include database schema patterns
- Add middleware conventions
- Note authentication strategy
```

**Monorepo (Turborepo/Nx):**
```
- Document workspace structure
- Include shared packages
- Add build/deploy strategy
- Note inter-package dependencies
```

### **Incremental Updates**

If onboarding already done:
```
You: "/onboard-codebase --update"

→ Detect what changed since last onboarding
→ Update only changed sections
→ Preserve manual edits
→ Fast update (2-3 min)
```

### **Module-Specific Onboarding**

```
You: "/onboard-codebase --modules apps/frontend,apps/backend"

→ Only analyze specified modules
→ Generate module-specific AGENTS.md
→ Faster for large monorepos
```

---

## 📝 Example Outputs

### Generated AGENTS.md (Sample)

```markdown
# MyApp - Context for AI Agents

<!-- AUTO-GENERATED: 2025-01-18 - Do not edit sections marked AUTO-GENERATED -->

## Project Overview
- **Type**: Monorepo (Turborepo)
- **Architecture**: Microservices with shared packages
- **Tech Stack**: React + Node.js + PostgreSQL + Redis
- **Team Size**: Medium (5-10 developers)
- **Deployment**: Docker + Kubernetes on AWS

## Tech Stack

### Frontend (apps/frontend)
- **Framework**: React 18.2.0 with TypeScript 5.0
- **Bundler**: Vite 4.3
- **Styling**: Tailwind CSS 3.3 + shadcn/ui components
- **State**: Zustand 4.3 for global state
- **Routing**: React Router 6.14
- **Data Fetching**: React Query 4.29
- **Forms**: React Hook Form 7.45 + Zod validation

### Backend (apps/api)
- **Runtime**: Node.js 20.x
- **Framework**: Express.js 4.18
- **Language**: TypeScript 5.0
- **ORM**: Prisma 5.0
- **Database**: PostgreSQL 15
- **Cache**: Redis 7.0
- **Queue**: Bull 4.10 for background jobs
- **Auth**: JWT with refresh tokens (jsonwebtoken 9.0)

### Testing
- **Unit**: Jest 29.5 + React Testing Library 14.0
- **E2E**: Playwright 1.35
- **Coverage**: 78% (target: 80%)

### DevOps
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes
- **CI/CD**: GitHub Actions
- **Hosting**: AWS (ECS + RDS + ElastiCache)

<!-- END AUTO-GENERATED -->

## Project Structure

```
myapp/
├── apps/
│   ├── frontend/          # React SPA
│   │   ├── src/
│   │   │   ├── components/    # Reusable components
│   │   │   ├── pages/         # Page components
│   │   │   ├── hooks/         # Custom hooks
│   │   │   ├── stores/        # Zustand stores
│   │   │   └── api/           # API client
│   │   └── vite.config.ts
│   │
│   ├── api/              # Express REST API
│   │   ├── src/
│   │   │   ├── routes/        # API routes
│   │   │   ├── services/      # Business logic
│   │   │   ├── middleware/    # Express middleware
│   │   │   ├── models/        # Prisma models (types)
│   │   │   └── utils/         # Utilities
│   │   └── server.ts
│   │
│   └── worker/           # Background job processor
│       └── src/
│           ├── jobs/          # Job handlers
│           └── index.ts
│
├── packages/
│   ├── shared/           # Shared utilities
│   │   ├── auth/             # Auth helpers
│   │   ├── db/               # Database client
│   │   └── types/            # Shared TypeScript types
│   │
│   └── ui/               # Shared UI components
│       └── components/       # Component library
│
├── prisma/
│   ├── schema.prisma     # Database schema
│   └── migrations/       # Database migrations
│
└── docker/
    ├── Dockerfile.frontend
    ├── Dockerfile.api
    └── docker-compose.yml
```

<!-- AUTO-GENERATED -->

## Important Conventions

### Code Style
- **Naming**: camelCase for variables/functions, PascalCase for components/classes
- **Imports**: Absolute imports from src/ using tsconfig paths
- **Components**: Functional components only, no class components
- **Hooks**: Custom hooks prefix with `use` (e.g., useAuth)
- **Files**: Co-located tests (*.test.ts next to source)

### TypeScript
- **Strict mode**: Enabled
- **Interface vs Type**: Use `interface` for objects, `type` for unions
- **Props typing**: Interface named `{ComponentName}Props`
- **No `any`**: Use `unknown` if type truly unknown

### React Patterns
- **Components**: Export as named exports (not default)
- **Props**: Always destructure props
- **State**: Zustand for global, useState for local
- **Side effects**: useEffect with proper dependencies
- **Styling**: Tailwind utility classes, shadcn/ui components

### API Conventions
- **Routes**: RESTful (GET /api/users, POST /api/users, etc)
- **Response**: Consistent JSON structure { data, error, meta }
- **Errors**: HTTP status codes + error messages
- **Auth**: JWT in Authorization header (Bearer token)
- **Validation**: Zod schemas in middleware

### Database
- **Naming**: snake_case for tables/columns
- **IDs**: UUID v4 (not auto-increment)
- **Timestamps**: created_at, updated_at on all tables
- **Migrations**: Always use Prisma migrations (never manual SQL)

<!-- END AUTO-GENERATED -->

## Critical Files

### Entry Points
- `apps/frontend/src/main.tsx` - Frontend application entry
- `apps/api/src/server.ts` - API server entry
- `apps/worker/src/index.ts` - Background worker entry

### Configuration
- `turbo.json` - Turborepo configuration
- `prisma/schema.prisma` - Database schema
- `.env.example` - Environment variables template
- `docker-compose.yml` - Local development setup

### Core Services
- `apps/api/src/services/AuthService.ts` - Authentication logic
- `apps/api/src/services/UserService.ts` - User management (⚠️ Complex, 503 lines)
- `apps/api/src/middleware/auth.ts` - JWT validation middleware
- `packages/shared/db/client.ts` - Prisma client singleton

<!-- AUTO-GENERATED -->

## Workflows

### Authentication Flow
1. User submits credentials → POST /api/auth/login
2. API validates → UserService.validateCredentials()
3. Generate JWT + refresh token → AuthService.generateTokens()
4. Store refresh token in Redis with expiry
5. Return tokens → Frontend stores in httpOnly cookie
6. Subsequent requests include Authorization: Bearer {token}
7. Middleware validates JWT → auth.ts middleware
8. Refresh flow → POST /api/auth/refresh with refresh token

### User Registration Flow
1. User submits form → POST /api/auth/register
2. Validate input → Zod schema
3. Hash password → bcrypt (12 rounds)
4. Create user → Prisma User.create()
5. Queue email verification → Bull queue (sendEmailVerification job)
6. Return success → User gets email with verification link
7. Click link → GET /api/auth/verify?token=...
8. Verify token → Mark user as verified

### File Upload Flow
1. User uploads file → POST /api/files/upload
2. Validate file type/size → Multer middleware
3. Queue upload job → Bull queue (uploadToS3 job)
4. Worker picks up job → Upload to S3
5. Generate thumbnail → If image, create thumbnail
6. Update database → Store file metadata
7. Return file URL → Frontend displays file

<!-- END AUTO-GENERATED -->

## Common Tasks

```bash
# Development
npm run dev          # Start all apps in dev mode (Turborepo)
npm run dev:frontend # Start frontend only
npm run dev:api      # Start API only

# Database
npm run db:migrate   # Run Prisma migrations
npm run db:seed      # Seed database with test data
npm run db:studio    # Open Prisma Studio (DB GUI)

# Testing
npm test            # Run all tests
npm run test:watch  # Run tests in watch mode
npm run test:e2e    # Run E2E tests with Playwright

# Build & Deploy
npm run build       # Build all apps for production
npm run docker:up   # Start Docker containers
npm run docker:down # Stop Docker containers

# Code Quality
npm run lint        # Run ESLint
npm run format      # Run Prettier
npm run typecheck   # Run TypeScript compiler check
```

<!-- AUTO-GENERATED -->

## Known Issues / Tech Debt

### High Priority
⚠️ **UserService.ts too complex** (503 lines, cyclomatic complexity: 45)
   - Should be split into: UserAuthService, UserProfileService, UserNotificationService
   - Estimated refactoring: 2-3 hours

⚠️ **Missing rate limiting on auth endpoints**
   - Login/register endpoints vulnerable to brute force
   - Recommended: express-rate-limit
   - Estimated: 1 hour

### Medium Priority
⚠️ **Express outdated** (4.18 → 4.19 available)
   - Update recommended for security patches
   - Breaking changes: None
   - Estimated: 15 minutes

⚠️ **3 TODO comments in codebase**
   - apps/api/src/services/UserService.ts:156 - "TODO: Add pagination"
   - apps/frontend/src/hooks/useAuth.ts:45 - "TODO: Handle refresh token expiry"
   - packages/shared/auth/jwt.ts:23 - "FIXME: Secret should come from env"

### Low Priority
💡 **Consider React Query devtools** for better debugging
💡 **Add Swagger/OpenAPI** for API documentation
💡 **Test coverage below 80%** (current: 78%, target: 80%)

<!-- END AUTO-GENERATED -->

## External Services

### AWS Services
- **S3**: File storage (bucket: myapp-uploads-production)
- **RDS**: PostgreSQL database (db.t3.medium)
- **ElastiCache**: Redis cache (cache.t3.micro)
- **ECS**: Container orchestration (Fargate)

### Third-Party APIs
- **SendGrid**: Email delivery (transactional emails)
- **Stripe**: Payment processing (subscriptions)
- **Sentry**: Error tracking and monitoring
- **Datadog**: Application performance monitoring

### Environment Variables
See `.env.example` for complete list. Critical variables:
```
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
JWT_SECRET=...
AWS_ACCESS_KEY_ID=...
SENDGRID_API_KEY=...
STRIPE_SECRET_KEY=...
```

---

<!-- USER EDITS BELOW - Not auto-generated -->

## Team Notes

*Add team-specific context here that agents should know*

## Custom Patterns

*Document any custom patterns specific to your team*
```

---

## 🎓 Summary

This command provides **comprehensive, intelligent codebase onboarding** that:

✅ Saves 10+ hours of manual documentation work
✅ Generates high-quality, project-specific context
✅ Enables agents to work productively from day one
✅ Maintains consistency with auto-generated sections
✅ Scales from small apps to large monorepos
✅ Uses parallel execution for speed (12 min vs 30+ min)

The generated files become the foundation for all future agent interactions with your codebase.

---

**Next:** After running this command, review generated files and add project-specific details. Then start working - agents will automatically use this context!
