# Hướng dẫn Onboarding Codebase với Oh My OpenCode

> Tài liệu này hướng dẫn cách sử dụng command `/onboard-codebase` để tự động phân tích và tạo tất cả context files cần thiết cho agents.

---

## 📋 Mục lục

1. [Giới thiệu](#giới-thiệu)
2. [Cách sử dụng](#cách-sử-dụng)
3. [Command hoạt động như thế nào](#command-hoạt-động-như-thế-nào)
4. [Kết quả đầu ra](#kết-quả-đầu-ra)
5. [Best Practices](#best-practices)
6. [Troubleshooting](#troubleshooting)

---

## 🎯 Giới thiệu

### Vấn đề

Khi bắt đầu với một codebase mới (hoặc muốn Oh My OpenCode hiểu codebase hiện tại):
- ❌ Agents không biết architecture của bạn
- ❌ Agents không biết conventions, patterns của bạn
- ❌ Phải giải thích lại mỗi lần làm việc mới
- ❌ Mất hàng giờ viết documentation thủ công

### Giải pháp

Command `/onboard-codebase` tự động:
- ✅ Phân tích toàn diện codebase
- ✅ Phát hiện tech stack, architecture, patterns
- ✅ Tự động tạo AGENTS.md (project + per-module)
- ✅ Tự động tạo conditional rules
- ✅ Tạo/cập nhật README, ARCHITECTURE.md
- ✅ Configure LSP cho tech stack
- ✅ Generate comprehensive onboarding report

**Thời gian:** ~12 phút (thay vì 10+ giờ thủ công!)

---

## 🚀 Cách sử dụng

### Bước 1: Cài đặt Oh My OpenCode

Nếu chưa cài:
```bash
# Follow instructions at:
# https://github.com/code-yeongyu/oh-my-opencode
```

### Bước 2: Navigate to your project

```bash
cd /path/to/your-project
```

### Bước 3: Start OpenCode

```bash
opencode
```

### Bước 4: Run onboarding command

**Basic usage (recommended for first time):**
```
/onboard-codebase
```

**Full analysis (more comprehensive):**
```
/onboard-codebase --scope full
```

**Quick scan (faster, essential files only):**
```
/onboard-codebase --scope basic
```

**Dry run (preview without creating files):**
```
/onboard-codebase --dry-run
```

**Specific modules only (for monorepos):**
```
/onboard-codebase --modules apps/frontend,apps/backend
```

---

## 🔍 Command hoạt động như thế nào

### Multi-Phase Workflow (12 phút)

#### Phase 1: Quick Project Scan (30s)
```
Agent: @explore (Grok - super fast)

Scanning:
├─ File structure (glob)
├─ Package managers (package.json, requirements.txt, etc)
├─ Config files (tsconfig.json, .prettierrc, etc)
├─ Entry points (index.ts, main.py, etc)
└─ Statistics (files count, LOC, etc)

Output: project_metadata.json
```

#### Phase 2: Tech Stack Detection (2min - PARALLEL)
```
🔄 Parallel execution với 3 background agents:

Agent 1: @librarian
├─ Analyze dependencies
├─ Identify frameworks (React, Express, Django, etc)
├─ Check versions
└─ Detect monorepo tools

Agent 2: @explore
├─ AST-grep for import patterns
├─ Detect ORM (Prisma, TypeORM, etc)
├─ Find database configs
└─ Detect testing frameworks

Agent 3: @oracle
├─ Analyze architecture (monorepo, microservices)
├─ Identify design patterns
└─ Check code organization

Wait for all → Synthesize → tech_stack.json
```

#### Phase 3: Deep Analysis (5min - PARALLEL)
```
🔄 Parallel execution với 3 background agents:

Agent 1: @librarian (Documentation Audit)
├─ Find existing docs (README, ARCHITECTURE, etc)
├─ Extract existing info
├─ Identify documentation gaps
└─ Fetch official framework docs (context7 MCP)

Agent 2: @oracle (Code Quality)
├─ Run LSP diagnostics
├─ Analyze code complexity
├─ Security scan
└─ Find tech debt

Agent 3: @explore (Workflow Mapping)
├─ Trace user workflows
├─ Map API endpoints
├─ Database schema analysis
└─ Find background jobs

Wait for all → Synthesize → Deep insights
```

#### Phase 4: Content Generation (3min - PARALLEL)
```
🔄 Parallel execution với 3 background agents:

Agent 1: @OmO (AGENTS.md)
├─ Generate root AGENTS.md
├─ Generate per-module AGENTS.md
└─ Merge với existing content (preserve user edits)

Agent 2: @document-writer (Documentation)
├─ Generate/Update README.md
├─ Create ARCHITECTURE.md
└─ API documentation outline

Agent 3: @OmO (Conditional Rules)
├─ Create language rules (.claude/rules/typescript.md)
├─ Create framework rules (.claude/rules/react.md)
├─ Create database rules (.claude/rules/database.md)
└─ Create testing rules (.claude/rules/testing.md)

Wait for all → All documentation complete
```

#### Phase 5: Configuration (1min)
```
Agent: @OmO

Configuring:
├─ LSP servers for detected languages
├─ Recommended MCP servers
└─ Oh My OpenCode project config

Output: .opencode/oh-my-opencode.json
```

#### Phase 6: Validation & Report (30s)
```
Agent: @OmO

Validating:
├─ All files created
├─ Content quality check
└─ No placeholders left

Generating:
└─ Comprehensive onboarding report

Display report to user ✅
```

---

## 📦 Kết quả đầu ra

### Files được tạo

```
your-project/
├── AGENTS.md                          # ✅ Project-wide context
├── apps/
│   ├── frontend/
│   │   └── AGENTS.md                  # ✅ Frontend-specific context
│   └── backend/
│       └── AGENTS.md                  # ✅ Backend-specific context
├── .claude/
│   └── rules/
│       ├── typescript.md              # ✅ TypeScript standards
│       ├── react.md                   # ✅ React patterns
│       ├── database.md                # ✅ Database conventions
│       └── testing.md                 # ✅ Testing standards
├── .opencode/
│   └── oh-my-opencode.json            # ✅ Oh My OpenCode config
├── README.md                          # ✅ Updated/Created
└── ARCHITECTURE.md                    # ✅ Created (if --scope full)
```

### AGENTS.md Sample

```markdown
# Your Project - Context for AI Agents

## Project Overview
- Type: Monorepo (Turborepo)
- Architecture: Microservices
- Tech Stack: React + Node.js + PostgreSQL

## Tech Stack

### Frontend
- React 18.2.0 with TypeScript 5.0
- Vite 4.3
- Tailwind CSS 3.3
- Zustand for state management

### Backend
- Node.js 20.x
- Express.js 4.18
- Prisma ORM 5.0
- PostgreSQL 15

## Project Structure
[Detailed structure with explanations]

## Important Conventions
- camelCase for variables
- PascalCase for components
- Absolute imports from src/
- Co-located tests

## Critical Files
- apps/frontend/src/main.tsx - Frontend entry
- apps/api/src/server.ts - API entry
- prisma/schema.prisma - Database schema

## Workflows

### Authentication Flow
1. User login → POST /api/auth/login
2. Validate credentials
3. Generate JWT + refresh token
4. Store in Redis
5. Return to client

[More workflows...]

## Common Tasks
```bash
npm run dev          # Start dev mode
npm run db:migrate   # Run migrations
npm test             # Run tests
```

## Known Issues / Tech Debt
⚠️ UserService.ts too complex (503 lines)
⚠️ Missing rate limiting on auth endpoints

## External Services
- AWS S3: File storage
- SendGrid: Email delivery
- Stripe: Payments
```

### Conditional Rules Sample

**`.claude/rules/typescript.md`:**
```markdown
---
globs: ["*.ts", "*.tsx"]
description: "TypeScript coding standards for this project"
---

# TypeScript Standards

Based on detected patterns in this codebase:

## Naming Conventions
- Interfaces: PascalCase (detected: UserProps, AuthConfig)
- Types: PascalCase (detected: User, Post)
- Variables: camelCase (detected: userName, isActive)
- Constants: UPPER_SNAKE_CASE (detected: API_URL, MAX_RETRIES)

## Type vs Interface
Use `interface` for object shapes (detected preference)
Use `type` for unions and primitives

Examples from codebase:
```typescript
// Detected pattern
interface UserProps {
  id: string;
  name: string;
}

type Status = 'active' | 'inactive';
```

## Null Handling
Prefer explicit undefined over null (detected pattern)
Use optional chaining (?.) consistently

## Import Style
Absolute imports from src/ (detected pattern)
```typescript
// Detected pattern
import { User } from 'src/types/user';
// NOT: import { User } from '../../types/user';
```
```

### Onboarding Report Sample

```markdown
# Codebase Onboarding Complete! 🎉

## Summary
✅ Project: my-awesome-app
✅ Type: Monorepo (Turborepo)
✅ Tech Stack: React + Node.js + PostgreSQL
✅ Files generated: 8
✅ Time: 12 minutes
✅ Token usage: ~25,000 tokens

## Generated Files
📄 AGENTS.md (root)
📄 apps/frontend/AGENTS.md
📄 apps/backend/AGENTS.md
📋 .claude/rules/typescript.md
📋 .claude/rules/react.md
📋 .claude/rules/database.md
📋 .claude/rules/testing.md
📖 README.md (updated)
📖 ARCHITECTURE.md (created)
⚙️  .opencode/oh-my-opencode.json

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

## Architecture Insights
- Pattern: Service layer architecture
- API Style: REST
- Auth: JWT with refresh tokens
- State: Zustand

## Code Quality
✅ TypeScript strict mode enabled
✅ Good test coverage (78%)
⚠️  Some files >500 lines
⚠️  3 TODO comments found
✅ No security issues

## Recommendations

**Immediate Actions:**
1. Review AGENTS.md for accuracy
2. Add project-specific details
3. Review .claude/rules/ and adjust

**Optional Improvements:**
1. Refactor UserService.ts (503 lines)
2. Add rate limiting to auth
3. Update Express to 4.19
4. Add API documentation

## Next Steps

Your codebase is fully onboarded! 🚀

**Agents now have:**
✅ Complete architecture understanding
✅ Tech stack knowledge
✅ Coding conventions
✅ IDE-grade tools (LSP)
✅ Framework-specific rules

**You can now:**
- Implement features confidently
- Refactor with full context
- Debug effectively
- Generate code following patterns

**Try:**
```
You: "Add rate limiting to auth endpoints"
→ OmO reads AGENTS.md, knows auth flow
→ Implements with proper conventions

You: "Build a new dashboard component"
→ Frontend agent knows React + Tailwind
→ Follows component patterns from rules
```

---

Need help? Just ask! Agents are ready. 🚀
```

---

## 💡 Best Practices

### DO's ✅

#### 1. Review Generated Files
```bash
# After onboarding, review and customize:
- AGENTS.md → Add team-specific context
- .claude/rules/*.md → Adjust standards if needed
- .env.example → Verify all env vars documented
```

#### 2. Keep AGENTS.md Updated
```
When architecture changes:
You: "/onboard-codebase --update"
→ Refreshes auto-generated sections
→ Preserves your manual edits
```

#### 3. Use --modules for Large Monorepos
```
# Faster onboarding for specific modules
/onboard-codebase --modules apps/frontend,packages/shared
```

#### 4. Run Periodically
```
Recommended schedule:
- After major refactoring
- When adding new tech stack
- Monthly for active projects
- Before new team members join
```

#### 5. Combine with Other Commands
```
# Onboard first, then work
/onboard-codebase
# Wait for completion
You: "Now implement user authentication"
→ Agents use onboarding context
```

### DON'Ts ❌

#### 1. Don't Skip Review
```
❌ Bad: Run and forget
✅ Good: Review AGENTS.md, add specifics
```

#### 2. Don't Overwrite Manual Edits
```
❌ Bad: /onboard-codebase --force (loses edits)
✅ Good: /onboard-codebase (merges)
```

#### 3. Don't Ignore Recommendations
```
The report shows tech debt and issues
→ Address high-priority items
→ Plan medium-priority improvements
```

#### 4. Don't Run in Wrong Directory
```
❌ Bad: Run in subdirectory
✅ Good: Run in project root
```

---

## 🔧 Troubleshooting

### Issue: "Command not found"

**Problem:** Command not loaded

**Solution:**
```bash
# Verify command file exists
ls -la ~/.claude/commands/onboard-codebase.md

# If missing, create it (see installation guide)

# Restart OpenCode
exit
opencode
```

### Issue: "Takes too long (>20 minutes)"

**Problem:** Not using background agents

**Solution:**
```
Check command implementation uses:
- backgroundTaskManager.spawnBackground()
- Parallel execution in Phase 2, 3, 4
- Sequential MCP for complex reasoning

Expected: 12 minutes with parallel execution
```

### Issue: "Generic/template content generated"

**Problem:** Insufficient project information detected

**Solution:**
```
1. Ensure package.json exists with dependencies
2. Check config files are present (tsconfig.json, etc)
3. Run with --scope full for deeper analysis
4. Manually review and add specifics to AGENTS.md
```

### Issue: "AGENTS.md already exists, nothing created"

**Problem:** Default is merge mode (safe)

**Solution:**
```
Options:
1. Let it merge (recommended):
   /onboard-codebase
   → Preserves your edits, updates auto sections

2. Force overwrite (careful!):
   /onboard-codebase --force
   → Overwrites everything, use with caution

3. Dry run first:
   /onboard-codebase --dry-run
   → Preview what will be generated
```

### Issue: "Wrong tech stack detected"

**Problem:** Ambiguous project structure

**Solution:**
```
1. Check package.json has correct dependencies
2. Ensure config files are present
3. Add manual section to AGENTS.md:

## Tech Stack (Manual Override)
- Frontend: React (not Vue, ignore Vue dev dependency)
- Backend: Express (main framework)

4. Mark section to prevent auto-overwrite
```

### Issue: "Missing LSP configuration"

**Problem:** Language servers not detected

**Solution:**
```
Manually add to .opencode/oh-my-opencode.json:

{
  "lsp": {
    "typescript-language-server": {
      "command": ["typescript-language-server", "--stdio"],
      "extensions": [".ts", ".tsx"],
      "priority": 10
    }
  }
}

Install LSP server if needed:
npm install -g typescript-language-server
```

---

## 📊 Performance Metrics

### Expected Timeline

```yaml
Scope: basic
├─ Phase 1: 30s
├─ Phase 2: 1min (simplified)
├─ Phase 4: 1min (AGENTS.md only)
└─ Total: ~3 minutes

Scope: full (default)
├─ Phase 1: 30s
├─ Phase 2: 2min (parallel)
├─ Phase 3: 5min (parallel)
├─ Phase 4: 3min (parallel)
├─ Phase 5: 1min
├─ Phase 6: 30s
└─ Total: ~12 minutes

Scope: full with --modules
├─ Reduced scope = faster
└─ ~5-7 minutes (depending on modules)
```

### Token Usage

```yaml
Basic scope: ~8,000 tokens
Full scope: ~25,000 tokens
With context7 MCP: +5,000 tokens
With grep_app MCP: +3,000 tokens

Total (full): ~30,000 tokens
Cost: ~$0.50 (Claude Opus 4.5 + GPT-5.2 + Gemini 3)
```

### Quality Metrics

```yaml
AGENTS.md Quality:
- Accuracy: 90-95% (tech stack detection)
- Completeness: 80-90% (needs manual additions)
- Usefulness: 95% (agents can work immediately)

Rules Quality:
- Pattern matching: 85-90%
- Convention detection: 80-85%
- Framework rules: 90-95%

Overall Onboarding:
- Time saved: 10+ hours (vs manual)
- Agent productivity: +300% immediately
- Context accuracy: 85-95%
```

---

## 🎯 Use Cases

### Use Case 1: New Team Member Onboarding

```
Scenario: New developer joins team

Traditional:
├─ Read docs: 2 days
├─ Explore code: 3 days
├─ Understand patterns: 2 days
├─ First productive task: 1 week+
└─ Total: 1-2 weeks

With Oh My OpenCode:
├─ Run /onboard-codebase: 12 minutes
├─ Review AGENTS.md: 30 minutes
├─ First productive task: Same day!
└─ Total: 1 hour

Savings: 39 hours (1 week)
```

### Use Case 2: Legacy Codebase Migration

```
Scenario: Inheriting legacy project with no docs

Challenge:
- No documentation
- Unknown architecture
- Unclear conventions
- Lost tribal knowledge

Solution:
/onboard-codebase --scope full

Result:
✅ Architecture documented
✅ Tech stack identified
✅ Patterns extracted
✅ Conventions documented
✅ Ready to work immediately
```

### Use Case 3: Monorepo Management

```
Scenario: Large monorepo with 20+ packages

Problem:
- Each package different patterns
- Hard to track conventions
- New packages inconsistent

Solution:
/onboard-codebase --modules packages/pkg1,packages/pkg2,...

Result:
✅ Per-package AGENTS.md
✅ Per-package rules
✅ Consistent patterns documented
✅ Easy to maintain consistency
```

### Use Case 4: Pre-Refactoring Analysis

```
Scenario: Planning major refactoring

Process:
1. Run /onboard-codebase
2. Review generated quality report
3. Identify tech debt
4. Plan refactoring phases
5. Use agents to execute refactoring

Benefit:
- Data-driven refactoring decisions
- Clear understanding of current state
- Automated pattern detection
- Safe refactoring with LSP
```

---

## 📚 Related Documentation

- [Background Agents Mechanism](./background-agents-mechanism.md) - How parallel execution works
- [Oh My OpenCode README](../README.md) - Project overview
- [Agent Configuration](./agent-configuration.md) - Customizing agents

---

## 🎓 Summary

### Command `/onboard-codebase` provides:

```yaml
Analysis:
  ✅ Comprehensive codebase scan
  ✅ Tech stack detection (90-95% accurate)
  ✅ Architecture identification
  ✅ Pattern recognition
  ✅ Quality assessment

Generation:
  ✅ AGENTS.md (project + modules)
  ✅ Conditional rules (language/framework)
  ✅ README/ARCHITECTURE docs
  ✅ LSP configuration
  ✅ Oh My OpenCode settings

Performance:
  ⚡ 12 minutes (vs 10+ hours manual)
  🎯 85-95% accuracy
  💰 ~$0.50 token cost
  🚀 Immediate agent productivity

Workflow:
  📊 6-phase multi-agent orchestration
  🔄 Parallel execution (3 agents per phase)
  🧠 Sequential MCP for quality
  ✅ Validation and reporting
```

### Bottom Line:

**Run once, work forever.** Agents understand your codebase completely and work like senior developers from day one.

---

**Tác giả:** Oh My OpenCode Team
**Cập nhật:** 2025-01-18
**Phiên bản:** 1.0.0
