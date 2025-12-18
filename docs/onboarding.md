---
  🎯 Oh My OpenCode hiểu codebase như thế nào?

  Oh My OpenCode không cần "học" hay "training" trước. Nó hiểu codebase theo 
  thời gian thực khi làm việc, thông qua nhiều cơ chế thông minh:

  ---
  📋 Các bước Onboarding Codebase

  BƯỚC 1: Setup Context Files (Quan trọng nhất!)

  1.1. Tạo AGENTS.md ở root project

  Đây là cách chính thức để "dạy" agents về codebase của bạn:

  cd /path/to/your-project
  touch AGENTS.md

  Nội dung AGENTS.md:

  # Your Project Name - Context for AI Agents

  ## Project Overview
  - **Tech Stack**: React + Node.js + PostgreSQL + Redis
  - **Architecture**: Microservices với event-driven
  - **Deployment**: Docker + Kubernetes on AWS

  ## Key Technologies
  - Frontend: React 18, TypeScript, Tailwind CSS, Zustand
  - Backend: Express.js, Prisma ORM, Bull Queue
  - Database: PostgreSQL 15, Redis 7
  - Testing: Jest, React Testing Library, Supertest

  ## Project Structure
  project/
  ├── apps/
  │   ├── frontend/         # React app
  │   ├── api/             # REST API
  │   └── worker/          # Background jobs
  ├── packages/
  │   ├── shared/          # Shared utilities
  │   └── types/           # TypeScript types
  ├── prisma/              # Database schema
  └── docker/              # Docker configs

  ## Important Conventions
  - Use `camelCase` for variables, `PascalCase` for components
  - All API endpoints follow REST conventions
  - Database migrations in `prisma/migrations/`
  - Tests co-located with source files (*.test.ts)

  ## Critical Files
  - `apps/api/src/server.ts` - API entry point
  - `apps/frontend/src/App.tsx` - Frontend entry
  - `prisma/schema.prisma` - Database schema
  - `.env.example` - Environment variables template

  ## Authentication Flow
  1. User login → POST /api/auth/login
  2. Server validates → JWT token issued
  3. Frontend stores in httpOnly cookie
  4. All requests include Authorization header

  ## Common Tasks
  - Start dev: `npm run dev`
  - Run tests: `npm test`
  - Database migrate: `npm run db:migrate`
  - Build: `npm run build`

  ## Current Issues / Tech Debt
  - Need to refactor UserService.ts (too complex)
  - Auth middleware needs rate limiting
  - Consider switching to React Query for data fetching

  ## External Services
  - AWS S3 for file storage
  - SendGrid for emails
  - Stripe for payments

  Tại sao AGENTS.md quan trọng?
  - ✅ Auto-inject vào context mỗi khi agent đọc file
  - ✅ Agents hiểu ngay architecture, conventions, workflows
  - ✅ Tránh phải giải thích lại mỗi lần làm việc mới

  ---
  1.2. Tạo AGENTS.md cho từng module (Optional nhưng rất hữu ích)

  # Frontend-specific context
  apps/frontend/AGENTS.md

  # Backend-specific context
  apps/api/AGENTS.md

  Ví dụ apps/frontend/AGENTS.md:

  # Frontend Module Context

  ## Component Structure
  - All components in `src/components/`
  - Shared components in `src/components/shared/`
  - Page components in `src/pages/`

  ## State Management
  - Using Zustand for global state
  - Store files in `src/stores/`
  - Example: `useAuthStore`, `useUserStore`

  ## Styling
  - Tailwind CSS with custom config
  - Design tokens in `tailwind.config.js`
  - Use shadcn/ui components when possible

  ## API Integration
  - API client in `src/api/client.ts`
  - All endpoints typed in `src/types/api.ts`
  - Use React Query for data fetching

  ## Important Components
  - `src/components/Layout.tsx` - Main layout wrapper
  - `src/components/Auth/LoginForm.tsx` - Login flow
  - `src/hooks/useAuth.ts` - Auth helper hook

  Nested context hierarchy:
  Reading: apps/frontend/src/components/Auth/LoginForm.tsx

  Auto-injects:
  1. /AGENTS.md (project-wide)
  2. /apps/frontend/AGENTS.md (frontend-specific)
  3. /apps/frontend/src/components/Auth/AGENTS.md (nếu có)

  → Agent hiểu full context từ general → specific!

  ---
  1.3. Setup Conditional Rules (cho các patterns cụ thể)

  mkdir -p .claude/rules

  Ví dụ: TypeScript rules

  .claude/rules/typescript.md:
  ---
  globs: ["*.ts", "*.tsx"]
  description: "TypeScript coding standards"
  ---

  # TypeScript Rules

  - Always use strict mode
  - Prefer `interface` over `type` for objects
  - Use `type` for unions and primitives
  - No `any` type (use `unknown` if needed)
  - All functions must have return type annotations

  Ví dụ: React rules

  .claude/rules/react.md:
  ---
  globs: ["*.tsx", "src/components/**/*.ts"]
  description: "React component standards"
  ---

  # React Component Rules

  - Use functional components only (no class components)
  - Custom hooks start with `use` prefix
  - Props interface named `{ComponentName}Props`
  - Always destructure props
  - Use TypeScript for all props

  Example:
  ```tsx
  interface ButtonProps {
    label: string;
    onClick: () => void;
  }

  export function Button({ label, onClick }: ButtonProps) {
    return <button onClick={onClick}>{label}</button>;
  }

  **Ví dụ: Database rules**

  `.claude/rules/database.md`:
  ```markdown
  ---
  globs: ["prisma/schema.prisma", "prisma/migrations/**/*"]
  description: "Database schema standards"
  alwaysApply: false
  ---

  # Database Rules

  - Use `snake_case` for table and column names
  - All tables must have `id`, `created_at`, `updated_at`
  - Use `@db.Uuid` for IDs (not auto-increment)
  - Add indexes for foreign keys
  - Always write migration comments

  ---
  BƯỚC 2: LSP Configuration (Auto IDE-grade tools)

  Oh My OpenCode tự động setup LSP cho project, nhưng bạn có thể customize:

  ~/.config/opencode/oh-my-opencode.json:

  {
    "lsp": {
      "typescript-language-server": {
        "command": ["typescript-language-server", "--stdio"],
        "extensions": [".ts", ".tsx"],
        "priority": 10
      },
      "prisma-language-server": {
        "command": ["prisma-language-server", "--stdio"],
        "extensions": [".prisma"],
        "priority": 8
      }
    }
  }

  LSP cho agents access to:
  - ✅ Type information (hover)
  - ✅ Go to definition
  - ✅ Find all references
  - ✅ Rename symbol across project
  - ✅ Code actions (quick fixes)
  - ✅ Diagnostics (errors/warnings)

  → Agents "see" code như bạn dùng VSCode!

  ---
  BƯỚC 3: First Session - Let Agents Explore

  Khi bắt đầu làm việc với codebase mới:

  cd /path/to/your-project
  opencode

  Prompt đầu tiên nên là:

  Explore this codebase and give me a comprehensive overview.

  Focus on:
  1. Architecture and tech stack
  2. Key modules and their responsibilities
  3. Entry points and main workflows
  4. Database schema and models
  5. API endpoints and routes
  6. Testing strategy
  7. Build and deployment process

  Oh My OpenCode sẽ:

  // OmO orchestrates exploration
  const exploration = {
    phase1: {
      // @explore (Grok - fast) scans structure
      agent: 'explore',
      task: 'Quick codebase structure scan',
      tools: ['glob', 'ast-grep'],
      time: '30 seconds',
      output: 'File structure, entry points, config files'
    },

    phase2: {
      // @librarian (Claude Sonnet) deep analysis
      agent: 'librarian',
      task: 'Analyze architecture and patterns',
      tools: ['lsp', 'read', 'grep'],
      time: '3 minutes',
      output: 'Architecture diagram, tech stack, workflows'
    },

    phase3: {
      // @oracle (GPT-5.2) identifies patterns
      agent: 'oracle',
      task: 'Review design patterns and quality',
      tools: ['lsp', 'ast-grep'],
      time: '2 minutes',
      output: 'Design patterns, code quality assessment'
    }
  };

  // Total: ~5 minutes for comprehensive understanding

  Output example:

  # Codebase Overview: Your Project

  ## Architecture
  - **Pattern**: Monorepo with microservices
  - **Frontend**: React SPA with Vite
  - **Backend**: Express.js REST API
  - **Database**: PostgreSQL with Prisma ORM
  - **Queue**: Bull with Redis for async jobs

  ## Tech Stack Analysis
  ✅ Modern stack (React 18, Node 20, PostgreSQL 15)
  ⚠️  Some dependencies outdated (express@4.17 → 4.18 available)
  ✅ TypeScript throughout
  ✅ Good test coverage (78%)

  ## Entry Points
  1. Frontend: `apps/frontend/src/main.tsx`
  2. API: `apps/api/src/server.ts`
  3. Worker: `apps/worker/src/index.ts`

  ## Key Modules
  - `packages/shared/auth` - Authentication utilities
  - `packages/shared/db` - Database client
  - `apps/api/src/routes` - API route handlers
  - `apps/api/src/services` - Business logic

  ## Database Schema (12 tables)
  - User, Profile, Post, Comment
  - AuthToken, Session
  - File, Upload
  - Job, JobResult
  - Notification, ActivityLog

  ## API Endpoints (24 routes)
  - Auth: /api/auth/* (login, register, logout)
  - Users: /api/users/* (CRUD)
  - Posts: /api/posts/* (CRUD + comments)
  - Files: /api/files/* (upload, download)

  ## Workflows Identified
  1. **User Registration**
     - POST /api/auth/register
     - Email verification job queued
     - Welcome email sent

  2. **File Upload**
     - POST /api/files/upload
     - File → S3 via worker
     - Thumbnail generation job

  ## Code Quality
  - ✅ Consistent naming conventions
  - ✅ Good TypeScript usage
  - ⚠️  UserService.ts is complex (300+ lines)
  - ⚠️  Missing rate limiting on auth endpoints
  - ✅ Tests co-located with code

  ## Recommendations
  1. Refactor UserService.ts into smaller services
  2. Add rate limiting to auth routes
  3. Update Express to latest version
  4. Consider adding API documentation (Swagger)

  ---
  BƯỚC 4: Cách Agents Tiếp Tục Học

  Oh My OpenCode học liên tục khi làm việc:

  4.1. Progressive Context Building

  // Lần đầu làm việc với authentication
  User: "Fix the login bug"

  OmO: {
    step1: "Read AGENTS.md → Hiểu auth flow",
    step2: "Read apps/api/src/routes/auth.ts → Hiểu implementation",
    step3: "Use LSP → Find all auth-related files",
    step4: "Read test files → Hiểu expected behavior",
    step5: "Search logs → Find error patterns"
  }

  // Context đã build:
  Context.auth = {
    flow: "JWT-based with refresh tokens",
    files: ["auth.ts", "authMiddleware.ts", "AuthService.ts"],
    tests: ["auth.test.ts"],
    endpoints: ["/login", "/logout", "/refresh"],
    knownIssues: ["Intermittent timeout on /login"]
  }

  // Lần sau làm việc với auth → Context sẵn sàng!

  4.2. MCP-Enhanced Learning

  // Agents query external knowledge
  User: "Implement OAuth2 authentication"

  @librarian: {
    mcp: 'context7',
    query: 'passport.js OAuth2 implementation',
    result: 'Official Passport.js documentation + examples'
  }

  @librarian: {
    mcp: 'grep_app',
    query: 'express oauth2 passport implementation',
    result: 'Top 10 GitHub repos with OAuth2 + Express'
  }

  // Agents now understand:
  // - Official OAuth2 flow from docs
  // - Real-world implementations from GitHub
  // - Best practices from popular repos

  4.3. AST-Grep Pattern Learning

  // Agents identify code patterns
  User: "Find all database queries"

  @explore: {
    tool: 'ast-grep',
    pattern: 'prisma.$queryRaw`$_`',
    result: [
      'UserService.ts:45 - prisma.$queryRaw`SELECT * FROM users`',
      'PostService.ts:89 - prisma.$queryRaw`SELECT * FROM posts`',
      // ...
    ]
  }

  // Now agents know:
  // - How database queries are written in this project
  // - Common patterns and conventions
  // - Where to look for similar code

  ---
  🚀 Workflow Thực Tế

  Scenario 1: First Day với Codebase Mới

  # Day 1 - Morning
  cd my-project
  opencode

  Session 1: Exploration
  You: "Explore this codebase, I'm new to this project"

  OmO:
  - Spawns @explore for quick scan (30s)
  - Spawns @librarian for deep analysis (3min)
  - Generates comprehensive overview
  - Creates visual architecture diagram
  - Lists key files and workflows

  Output: Full understanding in 5 minutes ✅

  Session 2: First Task
  You: "Add rate limiting to auth endpoints"

  OmO:
  1. Reads AGENTS.md → Knows auth flow
  2. Uses LSP → Finds auth middleware
  3. Uses grep_app MCP → Finds best rate-limit libraries
  4. Uses context7 MCP → Reads express-rate-limit docs
  5. Spawns @oracle → Design rate limit strategy
  6. Implements with tests
  7. Updates AGENTS.md with new pattern

  Time: 15 minutes ✅
  Quality: Production-ready with tests ✅

  ---
  Scenario 2: Complex Refactoring

  You: "Refactor UserService.ts - it's too complex"

  OmO Workflow:

  Phase 1: Analysis (parallel)
  ├─ @explore: Map all UserService dependencies
  ├─ @librarian: Find similar refactoring examples
  └─ @oracle: Analyze complexity and suggest breakdown

  Phase 2: Planning
  ├─ OmO: Create refactoring plan
  ├─ Split into: UserAuthService, UserProfileService, UserNotificationService
  └─ Generate migration strategy

  Phase 3: Implementation (parallel)
  ├─ @OmO: Implement UserAuthService (main thread)
  ├─ @frontend-ui-ux-engineer: Update frontend imports (background)
  └─ @OmO: Update tests (main thread)

  Phase 4: Validation
  ├─ Run all tests
  ├─ Check no broken references (LSP)
  └─ Update documentation

  Total: 25 minutes (vs 2 hours manual) ✅
  Safety: LSP ensures no broken references ✅

  ---
  🎯 Best Practices

  DO's ✅

  1. Luôn tạo AGENTS.md
    - Đầu tư 30 phút viết AGENTS.md tốt
    - Tiết kiệm hàng giờ giải thích sau này
  2. Update AGENTS.md khi architecture thay đổi
  You: "We just added GraphQL API"

  Then: Update AGENTS.md to include GraphQL info
  3. Use conditional rules cho từng tech stack
    - .claude/rules/graphql.md khi dùng GraphQL
    - .claude/rules/react.md cho React
    - .claude/rules/testing.md cho test conventions
  4. Let agents explore first trước khi làm task lớn
  Bad: "Refactor the entire auth system" (agents confused)

  Good:
  Step 1: "Analyze current auth implementation"
  Step 2: "Propose refactoring strategy"
  Step 3: "Implement refactoring phase 1"
  5. Leverage background agents cho learning
  While implementing:
  - Spawn @librarian to research best practices (background)
  - Spawn @explore to find similar code (background)
  - Continue implementing (main thread)

  ---
  DON'Ts ❌

  1. Đừng assume agents biết mọi thứ ngay
    - First session: Let them explore
    - Give context qua AGENTS.md
  2. Đừng bỏ qua LSP setup
    - LSP = agents có superpowers
    - Refactoring safe, navigation fast
  3. Đừng đặt quá nhiều conventions vào AGENTS.md
    - Keep concise, highlight important only
    - Details → conditional rules
  4. Đừng quên update docs khi code changes
    - AGENTS.md outdated = agents confused
    - Ask agents to update AGENTS.md after major changes

  ---
  📊 Kết Luận

  Oh My OpenCode hiểu codebase qua:

  Static Context:
    - AGENTS.md files (hierarchy)
    - Conditional rules (.claude/rules/)
    - README.md files
    - Package.json, tsconfig.json, etc.

  Dynamic Learning:
    - LSP tools (real-time code understanding)
    - AST-Grep (pattern recognition)
    - File reads (on-demand)
    - MCP servers (external knowledge)

  Knowledge Building:
    - Progressive: Learns as works
    - Cached: Remembers within session
    - Shared: All agents access same context
    - Enhanced: MCP for external research

  Timeline:

  Traditional onboarding:
  ├─ Read docs: 2 hours
  ├─ Explore code: 4 hours
  ├─ Understand patterns: 3 hours
  ├─ First productive task: 2 hours
  └─ Total: 11 hours (or days!)

  Oh My OpenCode onboarding:
  ├─ Create AGENTS.md: 30 minutes
  ├─ Let agents explore: 5 minutes
  ├─ First task: 15 minutes
  └─ Total: 50 minutes! 🚀

  13x faster onboarding!

  ---
  Bạn muốn tôi tạo:
  1. Template AGENTS.md cho project của bạn?
  2. Onboarding checklist step-by-step?
  3. Example conditional rules cho tech stack cụ thể?

  Hoặc bạn có codebase cụ thể muốn tôi giúp setup ngay bây giờ? 😊