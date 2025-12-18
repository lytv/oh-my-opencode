# Cơ chế Background Agents - Chi tiết kỹ thuật 🔬

> Tài liệu này giải thích chi tiết cách Oh My OpenCode thực hiện parallel execution với background agents để đạt hiệu suất 2-4x so với sequential execution truyền thống.

---

## 📋 Mục lục

1. [Kiến trúc tổng quan](#kiến-trúc-tổng-quan)
2. [Chi tiết từng component](#chi-tiết-từng-component)
3. [Workflow Example](#workflow-example---full-stack-app)
4. [Technical Implementation](#technical-implementation-details)
5. [Advanced Scenarios](#advanced-scenarios)
6. [Performance Characteristics](#performance-characteristics)

---

## 🏗️ Kiến trúc tổng quan

### Traditional Sequential Model (Cách cũ)
```
User Request
    ↓
[Main Agent] ──→ Task 1 (5 min) ──→ Done
    ↓
[Main Agent] ──→ Task 2 (10 min) ──→ Done
    ↓
[Main Agent] ──→ Task 3 (8 min) ──→ Done
    ↓
Total Time: 23 minutes
CPU Idle: 66% (chờ đợi API responses)
```

**Vấn đề:**
- ❌ Một task phải chờ task trước hoàn thành
- ❌ CPU/Network idle phần lớn thời gian
- ❌ Không tận dụng được specialized agents
- ❌ User phải chờ rất lâu mới thấy kết quả

### Oh My OpenCode Parallel Model (Cách mới)
```
User Request
    ↓
[OmO Orchestrator]
    ├──→ [Agent 1: Background] ──→ Task 1 (5 min)
    ├──→ [Agent 2: Background] ──→ Task 2 (10 min)
    └──→ [Agent 3: Main Thread] ──→ Task 3 (8 min)
         ↓
    [Wait for all] ──→ Sync results
         ↓
    [Integration] ──→ Ship
         ↓
Total Time: 10 minutes (longest task)
Efficiency: 2.3x faster
```

**Ưu điểm:**
- ✅ Multiple tasks chạy song song
- ✅ Tận dụng tối đa resources
- ✅ Specialized agents cho từng task type
- ✅ Incremental delivery - user thấy progress liên tục

---

## 🔍 Chi tiết từng component

### 1. OmO Orchestrator - Bộ não điều phối

**Specifications:**
```yaml
Model: Claude Opus 4.5
Role: Orchestrator + Main executor
Capabilities:
  - Extended thinking (32k token budget)
  - Task delegation intelligence
  - Parallel execution coordination
  - Todo-driven workflow
  - Background task monitoring
```

**Nhiệm vụ của OmO:**

#### 1.1 Phân tích Request (Analysis)
```typescript
// OmO receives user request
input: "Build a todo app with React frontend and Node.js backend"

// OmO analyzes
analysis: {
  complexity: "medium",
  components: ["frontend", "backend", "database"],
  estimatedTime: "30-40 minutes",
  parallelizationOpportunity: "high"
}
```

#### 1.2 Task Decomposition (Chia nhỏ)
```typescript
// OmO breaks down into subtasks
tasks: [
  {
    id: "define-api-contract",
    type: "design",
    dependencies: [],
    estimatedTime: "30 seconds",
    blocking: true  // Must complete before parallel tasks
  },
  {
    id: "implement-backend",
    type: "implementation",
    dependencies: ["define-api-contract"],
    estimatedTime: "15 minutes",
    agent: "OmO",
    mode: "main-thread"
  },
  {
    id: "implement-frontend",
    type: "implementation",
    dependencies: ["define-api-contract"],
    estimatedTime: "15 minutes",
    agent: "frontend-ui-ux-engineer",
    mode: "background"
  },
  {
    id: "integration",
    type: "integration",
    dependencies: ["implement-backend", "implement-frontend"],
    estimatedTime: "5 minutes",
    blocking: true
  }
]
```

#### 1.3 Agent Selection (Chọn specialist)
```typescript
// OmO selects best agent for each task
agentSelection: {
  "implement-backend": {
    agent: "OmO",
    reason: "Claude Opus 4.5 - excellent for logical, structured backend code",
    model: "anthropic/claude-opus-4-5"
  },
  "implement-frontend": {
    agent: "frontend-ui-ux-engineer",
    reason: "Gemini 3 Pro - creative, beautiful UI generation",
    model: "google/gemini-3-pro-preview"
  }
}
```

#### 1.4 Execution Strategy (Chiến lược thực thi)
```typescript
// OmO decides execution order
executionPlan: {
  phase1: {
    type: "sequential",
    tasks: ["define-api-contract"],
    reason: "Must establish contract before parallel work"
  },
  phase2: {
    type: "parallel",
    tasks: ["implement-backend", "implement-frontend"],
    reason: "Independent tasks with shared contract"
  },
  phase3: {
    type: "sequential",
    tasks: ["integration"],
    reason: "Requires both implementations complete"
  }
}
```

#### 1.5 Monitor & Integrate (Theo dõi & tổng hợp)
```typescript
// OmO monitors background tasks
monitoring: {
  "implement-frontend": {
    status: "running",
    progress: "60%",
    estimatedCompletion: "5 minutes remaining",
    notifications: [
      "✅ TodoList component complete",
      "✅ TodoItem component complete",
      "🔄 AddTodo form in progress..."
    ]
  }
}

// When all complete, OmO integrates
integration: {
  action: "merge-results",
  verification: "run-integration-tests",
  finalReview: "check-quality-standards"
}
```

---

### 2. Background Task System - Cơ chế chạy ngầm

#### 2.1 Core Architecture

```typescript
// Pseudo-code minh họa cơ chế background tasks
class BackgroundTaskManager {
  // Pool chứa các background tasks
  private tasks: Map<string, BackgroundTask> = new Map();

  // Event emitter cho notifications
  private events: EventEmitter = new EventEmitter();

  /**
   * Spawn background agent
   * Returns immediately (non-blocking)
   */
  spawnBackground(agent: Agent, task: Task): TaskHandle {
    const handle: TaskHandle = {
      id: generateId(),
      agent: agent,
      task: task,
      status: 'running',
      startTime: Date.now(),
      progress: 0
    };

    // Store task
    this.tasks.set(handle.id, handle);

    // Execute asynchronously (không block main thread)
    this.executeAsync(handle);

    // Return handle ngay lập tức
    return handle;
  }

  /**
   * Execute task asynchronously
   * Chạy trong background thread/process
   */
  private async executeAsync(handle: TaskHandle): Promise<void> {
    try {
      // Agent chạy độc lập
      const result = await handle.agent.execute(handle.task);

      // Update status
      handle.status = 'completed';
      handle.result = result;
      handle.completionTime = Date.now();

      // Emit completion event
      this.events.emit('task-completed', handle);

      // Notify orchestrator
      this.notifyCompletion(handle);

    } catch (error) {
      handle.status = 'failed';
      handle.error = error;

      // Emit error event
      this.events.emit('task-failed', handle);

      // Notify orchestrator
      this.notifyError(handle);
    }
  }

  /**
   * Wait for specific tasks to complete
   * Orchestrator calls this to sync
   */
  async waitFor(handles: TaskHandle[]): Promise<Result[]> {
    return Promise.all(
      handles.map(h => this.getResult(h))
    );
  }

  /**
   * Get result (blocks until ready)
   */
  private async getResult(handle: TaskHandle): Promise<Result> {
    if (handle.status === 'completed') {
      return handle.result;
    }

    if (handle.status === 'failed') {
      throw handle.error;
    }

    // Wait for completion
    return new Promise((resolve, reject) => {
      this.events.once(`complete-${handle.id}`, resolve);
      this.events.once(`error-${handle.id}`, reject);
    });
  }

  /**
   * Check progress of background task
   */
  getProgress(handle: TaskHandle): TaskProgress {
    return {
      status: handle.status,
      progress: handle.progress,
      currentStep: handle.currentStep,
      estimatedTimeRemaining: this.estimateTimeRemaining(handle)
    };
  }
}
```

#### 2.2 Task Handle & State Management

```typescript
interface TaskHandle {
  id: string;
  agent: Agent;
  task: Task;
  status: 'queued' | 'running' | 'completed' | 'failed';
  priority: number;

  // Timing
  startTime: number;
  completionTime?: number;

  // Progress tracking
  progress: number;  // 0-100
  currentStep?: string;

  // Results
  result?: any;
  error?: Error;

  // Metadata
  metadata: {
    tokensUsed?: number;
    apiCalls?: number;
    filesModified?: string[];
  };
}

// Task state machine
enum TaskStatus {
  QUEUED = 'queued',      // Task created, waiting to start
  RUNNING = 'running',     // Task executing
  COMPLETED = 'completed', // Task finished successfully
  FAILED = 'failed'        // Task failed with error
}
```

---

### 3. Specialized Agents - Team Members

#### 3.1 Agent Roster

```yaml
OmO:
  model: anthropic/claude-opus-4-5
  role: Orchestrator + Backend specialist
  strengths:
    - Complex reasoning (32k thinking budget)
    - Logical architecture
    - Reliable code generation
    - Task coordination
  use_cases:
    - Main orchestration
    - Backend implementation
    - Integration work
    - Complex logic

oracle:
  model: openai/gpt-5.2
  role: Architecture & code review
  strengths:
    - Stellar logical reasoning
    - Deep analysis
    - Design patterns
    - Debugging expertise
  use_cases:
    - Architecture review
    - Complex debugging
    - Strategic decisions
    - Code review

librarian:
  model: anthropic/claude-sonnet-4-5
  role: Research & documentation
  strengths:
    - Multi-repo analysis
    - Documentation lookup
    - GitHub research
    - Evidence-based answers
  use_cases:
    - Find implementations
    - Research best practices
    - Documentation search
    - Pattern discovery

explore:
  model: opencode/grok-code
  role: Fast codebase exploration
  strengths:
    - Blazing fast search
    - Pattern matching
    - File traversal
  use_cases:
    - Quick codebase search
    - File location
    - Pattern finding
    - Structure exploration

frontend-ui-ux-engineer:
  model: google/gemini-3-pro-preview
  role: UI/UX specialist
  strengths:
    - Creative UI design
    - Beautiful styling
    - Component architecture
    - Responsive design
  use_cases:
    - React/Vue components
    - UI implementation
    - Design systems
    - Frontend features

document-writer:
  model: google/gemini-3-pro-preview
  role: Technical writing
  strengths:
    - Clear prose
    - Documentation structure
    - Explanation clarity
  use_cases:
    - README files
    - API documentation
    - Technical guides
    - Code comments

multimodal-looker:
  model: google/gemini-2.5-flash
  role: Visual analysis
  strengths:
    - Image analysis
    - PDF parsing
    - Diagram interpretation
  use_cases:
    - Design mockup analysis
    - Architecture diagram reading
    - Screenshot interpretation
    - Visual documentation
```

---

## 🎯 Workflow Example - Full-Stack App

Hãy đi qua một ví dụ cụ thể step-by-step:

### User Request
```bash
User: "Build a todo app with React frontend and Node.js backend"
```

### Phase 1: OmO Analysis (10 seconds)

```typescript
// OmO's internal reasoning
OmO.think({
  request: "Build todo app with React + Node.js",

  analysis: {
    components: [
      "Database schema",
      "REST API (Node.js)",
      "React components",
      "Integration"
    ],

    dependencies: {
      "API implementation": ["API contract"],
      "Frontend implementation": ["API contract"],
      "Integration": ["API implementation", "Frontend implementation"]
    },

    parallelization: {
      opportunity: "high",
      reason: "Frontend and Backend can work in parallel after contract defined"
    }
  },

  strategy: {
    phase1: {
      type: "sequential",
      tasks: ["Define API contract"],
      duration: "30 seconds",
      reason: "Contract needed for both parallel tasks"
    },

    phase2: {
      type: "parallel",
      tasks: [
        {
          name: "Backend implementation",
          agent: "OmO",
          thread: "main",
          duration: "15 minutes"
        },
        {
          name: "Frontend implementation",
          agent: "frontend-ui-ux-engineer",
          thread: "background",
          duration: "15 minutes"
        }
      ],
      reason: "Independent implementations with shared contract"
    },

    phase3: {
      type: "sequential",
      tasks: ["Integration", "Testing"],
      duration: "5 minutes",
      reason: "Requires both implementations complete"
    }
  },

  estimatedTotal: "20.5 minutes",
  sequentialEquivalent: "35.5 minutes",
  timeSavings: "42%"
});
```

### Phase 2: Contract Definition (30 seconds)

```typescript
// OmO executes on main thread
[Main Thread] OmO: "Defining API contract..."

// Contract definition
const apiContract = {
  endpoints: [
    {
      method: "POST",
      path: "/todos",
      body: { title: "string", description?: "string" },
      response: { id: "string", title: "string", completed: "boolean", createdAt: "Date" }
    },
    {
      method: "GET",
      path: "/todos",
      response: "Todo[]"
    },
    {
      method: "PUT",
      path: "/todos/:id",
      body: { title?: "string", completed?: "boolean" },
      response: "Todo"
    },
    {
      method: "DELETE",
      path: "/todos/:id",
      response: { success: "boolean" }
    }
  ],

  schema: {
    Todo: {
      id: "string (UUID)",
      title: "string",
      description: "string | null",
      completed: "boolean",
      createdAt: "Date",
      updatedAt: "Date"
    }
  }
};

// Write contract to shared context
sharedContext.set('api-contract', apiContract);

// Write OpenAPI spec file
fs.writeFileSync('openapi.yaml', generateOpenAPISpec(apiContract));

console.log("✅ API contract defined and saved to openapi.yaml");
```

### Phase 3: Parallel Execution (15 minutes)

```typescript
// OmO spawns background task
const frontendHandle = backgroundTaskManager.spawnBackground(
  agent: agents.get('frontend-ui-ux-engineer'),
  task: {
    description: "Build React todo app using the API contract",
    context: {
      apiContract: sharedContext.get('api-contract'),
      requirements: [
        "Use React with TypeScript",
        "Beautiful Tailwind CSS styling",
        "Responsive design",
        "Component-based architecture",
        "Use shadcn/ui for better UX"
      ],
      outputPath: "src/frontend/"
    },
    expectedDeliverables: [
      "src/App.tsx",
      "src/components/TodoList.tsx",
      "src/components/TodoItem.tsx",
      "src/components/AddTodo.tsx",
      "src/types/todo.ts",
      "src/api/todos.ts"
    ]
  }
);

// Main thread continues with backend implementation
console.log("🔄 Frontend agent working in background...");
console.log("🔄 Starting backend implementation on main thread...");

// Backend implementation (Main thread)
[Main Thread] OmO executes:
```

**Backend Implementation Steps:**

```typescript
// Step 1: Setup Express server
OmO.implement({
  step: "1/5 - Express Server Setup",

  files: {
    "src/server.ts": `
      import express from 'express';
      import cors from 'cors';
      import todoRoutes from './routes/todos';

      const app = express();
      const PORT = process.env.PORT || 3000;

      app.use(cors());
      app.use(express.json());
      app.use('/api/todos', todoRoutes);

      app.listen(PORT, () => {
        console.log(\`Server running on port \${PORT}\`);
      });
    `
  },

  progress: "20%"
});

// Step 2: Database models
OmO.implement({
  step: "2/5 - Database Models",

  files: {
    "src/models/Todo.ts": `
      export interface Todo {
        id: string;
        title: string;
        description: string | null;
        completed: boolean;
        createdAt: Date;
        updatedAt: Date;
      }

      // In-memory store (for demo)
      class TodoStore {
        private todos: Map<string, Todo> = new Map();

        create(data: { title: string; description?: string }): Todo {
          const todo: Todo = {
            id: crypto.randomUUID(),
            title: data.title,
            description: data.description || null,
            completed: false,
            createdAt: new Date(),
            updatedAt: new Date()
          };
          this.todos.set(todo.id, todo);
          return todo;
        }

        findAll(): Todo[] {
          return Array.from(this.todos.values());
        }

        findById(id: string): Todo | undefined {
          return this.todos.get(id);
        }

        update(id: string, data: Partial<Todo>): Todo | undefined {
          const todo = this.todos.get(id);
          if (!todo) return undefined;

          const updated = { ...todo, ...data, updatedAt: new Date() };
          this.todos.set(id, updated);
          return updated;
        }

        delete(id: string): boolean {
          return this.todos.delete(id);
        }
      }

      export const todoStore = new TodoStore();
    `
  },

  progress: "40%"
});

// Step 3: CRUD endpoints
OmO.implement({
  step: "3/5 - CRUD Endpoints",

  files: {
    "src/routes/todos.ts": `
      import { Router } from 'express';
      import { todoStore } from '../models/Todo';
      import { validateTodo } from '../middleware/validation';

      const router = Router();

      // GET /api/todos
      router.get('/', (req, res) => {
        const todos = todoStore.findAll();
        res.json(todos);
      });

      // POST /api/todos
      router.post('/', validateTodo, (req, res) => {
        const { title, description } = req.body;
        const todo = todoStore.create({ title, description });
        res.status(201).json(todo);
      });

      // PUT /api/todos/:id
      router.put('/:id', (req, res) => {
        const { id } = req.params;
        const updated = todoStore.update(id, req.body);

        if (!updated) {
          return res.status(404).json({ error: 'Todo not found' });
        }

        res.json(updated);
      });

      // DELETE /api/todos/:id
      router.delete('/:id', (req, res) => {
        const { id } = req.params;
        const deleted = todoStore.delete(id);

        if (!deleted) {
          return res.status(404).json({ error: 'Todo not found' });
        }

        res.json({ success: true });
      });

      export default router;
    `
  },

  progress: "60%"
});

// Step 4: Validation middleware
OmO.implement({
  step: "4/5 - Validation Middleware",

  files: {
    "src/middleware/validation.ts": `
      import { Request, Response, NextFunction } from 'express';

      export function validateTodo(req: Request, res: Response, next: NextFunction) {
        const { title } = req.body;

        if (!title || typeof title !== 'string' || title.trim().length === 0) {
          return res.status(400).json({ error: 'Title is required' });
        }

        if (title.length > 200) {
          return res.status(400).json({ error: 'Title must be less than 200 characters' });
        }

        next();
      }
    `
  },

  progress: "80%"
});

// Step 5: Tests
OmO.implement({
  step: "5/5 - Tests",

  files: {
    "src/__tests__/todos.test.ts": `
      import request from 'supertest';
      import app from '../server';

      describe('Todo API', () => {
        it('should create a new todo', async () => {
          const response = await request(app)
            .post('/api/todos')
            .send({ title: 'Test todo' });

          expect(response.status).toBe(201);
          expect(response.body).toHaveProperty('id');
          expect(response.body.title).toBe('Test todo');
        });

        it('should get all todos', async () => {
          const response = await request(app).get('/api/todos');
          expect(response.status).toBe(200);
          expect(Array.isArray(response.body)).toBe(true);
        });

        it('should update a todo', async () => {
          const created = await request(app)
            .post('/api/todos')
            .send({ title: 'Original' });

          const response = await request(app)
            .put(\`/api/todos/\${created.body.id}\`)
            .send({ completed: true });

          expect(response.status).toBe(200);
          expect(response.body.completed).toBe(true);
        });

        it('should delete a todo', async () => {
          const created = await request(app)
            .post('/api/todos')
            .send({ title: 'To delete' });

          const response = await request(app)
            .delete(\`/api/todos/\${created.body.id}\`);

          expect(response.status).toBe(200);
          expect(response.body.success).toBe(true);
        });
      });
    `
  },

  progress: "100%"
});

console.log("✅ Backend implementation complete!");
```

**Meanwhile, in background thread:**

```typescript
// Frontend agent (Gemini 3 Pro) working in parallel
[Background] @frontend-ui-ux-engineer:

// Progress updates emitted during execution
t=2min:  "✅ Created project structure"
t=4min:  "✅ TodoList component complete with beautiful Tailwind styling"
t=7min:  "✅ TodoItem component with animations and interactions"
t=10min: "✅ AddTodo form with validation and UX polish"
t=12min: "✅ API client with TypeScript types"
t=15min: "✅ Integration complete - app fully functional!"

// Files created:
{
  "src/App.tsx": "Main app component with state management",
  "src/components/TodoList.tsx": "List view with filtering",
  "src/components/TodoItem.tsx": "Individual todo with edit/delete",
  "src/components/AddTodo.tsx": "Form with validation",
  "src/types/todo.ts": "TypeScript interfaces matching API",
  "src/api/todos.ts": "API client methods",
  "src/styles/globals.css": "Tailwind configuration",
  "tailwind.config.js": "Tailwind setup with custom theme"
}
```

**Timeline comparison:**

```
Time    | Main Thread (OmO)      | Background (Frontend Agent)
--------|------------------------|----------------------------
T+0     | Start backend          | Start frontend
T+3     | Express setup ✅       | Project structure ✅
T+6     | Database models ✅     | TodoList component ✅
T+9     | CRUD endpoints ✅      | TodoItem component ✅
T+12    | Validation ✅          | AddTodo form ✅
T+15    | Tests ✅ DONE!         | Integration ✅ DONE!
```

**Notification when frontend completes:**

```typescript
// Background task completion notification
[T+15min] 🎉 Background Notification:

"✅ @frontend-ui-ux-engineer completed frontend implementation!

Deliverables:
  ✅ Beautiful React app with Tailwind CSS
  ✅ TodoList component with filtering
  ✅ TodoItem with smooth animations
  ✅ AddTodo form with validation
  ✅ Full TypeScript support
  ✅ Responsive design (mobile + desktop)
  ✅ shadcn/ui components integrated

Files created: 8
Lines of code: ~450
Design quality: ⭐⭐⭐⭐⭐

Ready for integration!"

// OmO receives notification
OmO.onBackgroundComplete(frontendHandle);
```

### Phase 4: Integration (5 minutes)

```typescript
// Both backend and frontend are done
// OmO performs integration
[Main Thread] OmO: "Integrating frontend and backend..."

// Step 1: Review outputs
OmO.review({
  backend: {
    status: "✅ Complete",
    endpoints: [
      "POST /api/todos - ✅ Working",
      "GET /api/todos - ✅ Working",
      "PUT /api/todos/:id - ✅ Working",
      "DELETE /api/todos/:id - ✅ Working"
    ],
    tests: "✅ All passing (4/4)"
  },

  frontend: {
    status: "✅ Complete",
    components: "✅ All implemented",
    styling: "✅ Beautiful Tailwind design",
    apiClient: "✅ Ready for connection"
  }
});

// Step 2: Configure connection
OmO.integrate({
  action: "Connect frontend to backend",

  changes: {
    "src/api/todos.ts": {
      before: "const API_URL = 'http://localhost:3000';",
      after: "const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:3000';"
    },

    ".env.example": {
      new: "REACT_APP_API_URL=http://localhost:3000/api"
    },

    "README.md": {
      new: `
# Todo App

Full-stack todo application with React frontend and Node.js backend.

## Setup

1. Install dependencies: \`npm install\`
2. Start backend: \`npm run server\`
3. Start frontend: \`npm run dev\`

## API Endpoints

- POST /api/todos - Create todo
- GET /api/todos - List todos
- PUT /api/todos/:id - Update todo
- DELETE /api/todos/:id - Delete todo
      `
    }
  }
});

// Step 3: Integration tests
OmO.test({
  type: "integration",

  tests: [
    {
      name: "Create todo from UI",
      steps: [
        "Start backend server",
        "Start frontend dev server",
        "Open browser to localhost:5173",
        "Click 'Add Todo' button",
        "Enter title 'Integration test'",
        "Submit form"
      ],
      expected: "Todo appears in list",
      result: "✅ PASS"
    },
    {
      name: "Mark todo complete",
      steps: [
        "Click checkbox on todo item",
        "Verify UI updates immediately",
        "Refresh page",
        "Verify state persisted"
      ],
      expected: "Todo stays completed after refresh",
      result: "✅ PASS"
    },
    {
      name: "Delete todo",
      steps: [
        "Click delete button on todo",
        "Confirm deletion",
        "Verify todo removed from UI"
      ],
      expected: "Todo removed from list",
      result: "✅ PASS"
    }
  ],

  summary: "✅ All integration tests passing (3/3)"
});

console.log(`
✅ Integration complete!

📊 Final Summary:
   Backend: ✅ Node.js + Express + TypeScript
   Frontend: ✅ React + Tailwind + shadcn/ui
   Tests: ✅ 4 unit tests + 3 integration tests

⏱️  Total time: 20.5 minutes
💰 Token usage: ~45k tokens
🎯 Quality: Production-ready

🚀 Ready to ship!
`);
```

### Time Breakdown - Sequential vs Parallel

```
Sequential Execution (Traditional):
├─ Define API contract: 0.5 min
├─ Implement backend: 15 min
├─ Implement frontend: 15 min
└─ Integration: 5 min
────────────────────────────────
Total: 35.5 minutes

Parallel Execution (Oh My OpenCode):
├─ Define API contract: 0.5 min
├─ Backend + Frontend (parallel): 15 min ⚡
└─ Integration: 5 min
────────────────────────────────
Total: 20.5 minutes

Time saved: 15 minutes (42% faster!)
```

---

## 🔧 Technical Implementation Details

### 1. Inter-Agent Communication

#### 1.1 Message Protocol

```typescript
// Message types for agent communication
interface AgentMessage {
  type: 'delegate' | 'progress' | 'completed' | 'error' | 'query';
  from: AgentId;
  to: AgentId;
  timestamp: number;
  payload: any;
}

// Delegation message (Orchestrator → Agent)
interface DelegationMessage extends AgentMessage {
  type: 'delegate';
  payload: {
    task: {
      id: string;
      description: string;
      context: SharedContext;
      requirements: string[];
      expectedOutput: string[];
      deadline?: number;
    };
    mode: 'background' | 'blocking';
    priority: number;
  };
}

// Progress update (Agent → Orchestrator)
interface ProgressMessage extends AgentMessage {
  type: 'progress';
  payload: {
    taskId: string;
    progress: number;  // 0-100
    currentStep: string;
    estimatedTimeRemaining: number;
    intermediateResults?: any;
  };
}

// Completion message (Agent → Orchestrator)
interface CompletionMessage extends AgentMessage {
  type: 'completed';
  payload: {
    taskId: string;
    result: {
      files: string[];
      summary: string;
      metrics: {
        duration: number;
        tokensUsed: number;
        linesOfCode: number;
      };
      notes?: string[];
    };
  };
}

// Error message (Agent → Orchestrator)
interface ErrorMessage extends AgentMessage {
  type: 'error';
  payload: {
    taskId: string;
    error: {
      message: string;
      stack?: string;
      recoverable: boolean;
      suggestedAction?: string;
    };
  };
}

// Query message (Agent ↔ Agent)
interface QueryMessage extends AgentMessage {
  type: 'query';
  payload: {
    question: string;
    context?: any;
    urgency: 'low' | 'medium' | 'high';
  };
}
```

#### 1.2 Communication Channel

```typescript
// Message bus for agent communication
class AgentMessageBus {
  private channels: Map<AgentId, MessageQueue> = new Map();
  private subscriptions: Map<string, Callback[]> = new Map();

  // Send message to specific agent
  send(message: AgentMessage): void {
    const queue = this.channels.get(message.to);
    if (!queue) {
      throw new Error(`Agent ${message.to} not found`);
    }

    queue.enqueue(message);
    this.notifySubscribers(message);
  }

  // Broadcast to all agents
  broadcast(message: AgentMessage): void {
    for (const [agentId, queue] of this.channels) {
      if (agentId !== message.from) {
        queue.enqueue({ ...message, to: agentId });
      }
    }
  }

  // Subscribe to message types
  subscribe(pattern: string, callback: Callback): void {
    if (!this.subscriptions.has(pattern)) {
      this.subscriptions.set(pattern, []);
    }
    this.subscriptions.get(pattern)!.push(callback);
  }

  // Notify subscribers
  private notifySubscribers(message: AgentMessage): void {
    const pattern = `${message.type}:${message.to}`;
    const callbacks = this.subscriptions.get(pattern) || [];

    for (const callback of callbacks) {
      callback(message);
    }
  }
}

// Usage example
const messageBus = new AgentMessageBus();

// Orchestrator subscribes to completion events
messageBus.subscribe('completed:OmO', (message: CompletionMessage) => {
  console.log(`Task ${message.payload.taskId} completed!`);
  orchestrator.handleCompletion(message);
});

// Agent sends completion message
messageBus.send({
  type: 'completed',
  from: 'frontend-ui-ux-engineer',
  to: 'OmO',
  timestamp: Date.now(),
  payload: {
    taskId: 'implement-frontend',
    result: {
      files: ['src/App.tsx', 'src/components/*.tsx'],
      summary: 'Frontend implementation complete',
      metrics: {
        duration: 900000,  // 15 minutes
        tokensUsed: 25000,
        linesOfCode: 450
      }
    }
  }
});
```

---

### 2. Context Sharing Mechanism

#### 2.1 Shared Context Store

```typescript
// Shared context accessible by all agents
class SharedContext {
  // Global project context (read-only for agents)
  readonly projectContext: {
    name: string;
    structure: ProjectStructure;
    dependencies: Dependencies;
    conventions: CodingStandards;
    gitInfo: GitInfo;
  };

  // Agent outputs (read/write)
  private outputs: Map<AgentId, AgentOutput> = new Map();

  // Shared artifacts (API contracts, schemas, etc.)
  private artifacts: Map<string, Artifact> = new Map();

  // Dependencies between tasks
  private dependencies: Map<TaskId, TaskId[]> = new Map();

  // Watchers for reactive updates
  private watchers: Map<string, Callback[]> = new Map();

  /**
   * Get output from another agent
   */
  getOutput(agentId: AgentId): AgentOutput | undefined {
    return this.outputs.get(agentId);
  }

  /**
   * Set output from current agent
   */
  setOutput(agentId: AgentId, output: AgentOutput): void {
    this.outputs.set(agentId, output);

    // Notify watchers
    this.notifyWatchers(`output:${agentId}`, output);

    // Notify dependent tasks
    this.notifyDependents(agentId);
  }

  /**
   * Store shared artifact
   */
  setArtifact(key: string, artifact: Artifact): void {
    this.artifacts.set(key, artifact);
    this.notifyWatchers(`artifact:${key}`, artifact);
  }

  /**
   * Get shared artifact
   */
  getArtifact(key: string): Artifact | undefined {
    return this.artifacts.get(key);
  }

  /**
   * Watch for changes
   */
  watch(pattern: string, callback: Callback): void {
    if (!this.watchers.has(pattern)) {
      this.watchers.set(pattern, []);
    }
    this.watchers.get(pattern)!.push(callback);
  }

  /**
   * Notify dependent tasks when output available
   */
  private notifyDependents(agentId: AgentId): void {
    for (const [taskId, deps] of this.dependencies) {
      if (deps.includes(agentId)) {
        this.notifyWatchers(`dependency:${taskId}`, agentId);
      }
    }
  }

  /**
   * Register task dependency
   */
  registerDependency(taskId: TaskId, dependsOn: AgentId[]): void {
    this.dependencies.set(taskId, dependsOn);
  }
}

// Usage example
const sharedContext = new SharedContext();

// Backend agent sets API contract
sharedContext.setArtifact('api-contract', {
  type: 'openapi',
  version: '3.0.0',
  spec: openApiSpec
});

// Frontend agent watches for contract
sharedContext.watch('artifact:api-contract', (contract) => {
  console.log('API contract available, starting frontend implementation');
  frontendAgent.useContract(contract);
});

// Backend agent sets output when done
sharedContext.setOutput('backend-agent', {
  endpoints: [
    { method: 'POST', path: '/todos', status: 'implemented' },
    { method: 'GET', path: '/todos', status: 'implemented' }
  ],
  tests: { passed: 4, failed: 0 }
});

// Integration task depends on both agents
sharedContext.registerDependency('integration-task', [
  'backend-agent',
  'frontend-agent'
]);

// Watch for both dependencies
sharedContext.watch('dependency:integration-task', (completedAgent) => {
  const backendDone = sharedContext.getOutput('backend-agent');
  const frontendDone = sharedContext.getOutput('frontend-agent');

  if (backendDone && frontendDone) {
    console.log('Both backend and frontend ready, starting integration');
    integrationTask.start();
  }
});
```

#### 2.2 Context Optimization

```typescript
// Smart context to reduce token usage
class SmartContext extends SharedContext {
  // Cache frequently accessed data
  private cache: LRUCache<string, any> = new LRUCache(100);

  // Compress large contexts
  private compressor: ContextCompressor = new ContextCompressor();

  /**
   * Get context with optimization
   */
  getOptimizedContext(agentId: AgentId, taskType: string): any {
    // Check cache first
    const cacheKey = `${agentId}:${taskType}`;
    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    // Build minimal context for task type
    const context = this.buildMinimalContext(agentId, taskType);

    // Compress if needed
    const optimized = this.compressor.compress(context);

    // Cache result
    this.cache.set(cacheKey, optimized);

    return optimized;
  }

  /**
   * Build minimal context (only what's needed)
   */
  private buildMinimalContext(agentId: AgentId, taskType: string): any {
    switch (taskType) {
      case 'frontend':
        return {
          apiContract: this.getArtifact('api-contract'),
          styleGuide: this.projectContext.conventions.ui,
          componentLibrary: this.projectContext.dependencies.ui
        };

      case 'backend':
        return {
          databaseSchema: this.getArtifact('db-schema'),
          apiContract: this.getArtifact('api-contract'),
          securityRules: this.projectContext.conventions.security
        };

      case 'integration':
        return {
          backendOutput: this.getOutput('backend-agent'),
          frontendOutput: this.getOutput('frontend-agent'),
          testStrategy: this.projectContext.conventions.testing
        };

      default:
        return this.projectContext;
    }
  }
}
```

---

### 3. Dependency Management

#### 3.1 Task Dependency Graph

```typescript
// Task dependency graph for execution planning
class TaskGraph {
  private nodes: Map<TaskId, TaskNode> = new Map();
  private edges: Map<TaskId, TaskId[]> = new Map();

  /**
   * Add task to graph
   */
  addTask(task: Task, dependencies: TaskId[] = []): void {
    this.nodes.set(task.id, {
      task,
      status: 'pending',
      dependencies
    });

    this.edges.set(task.id, dependencies);
  }

  /**
   * Build execution plan using topological sort
   */
  buildExecutionPlan(): ExecutionPlan {
    const plan: ExecutionPlan = {
      phases: []
    };

    // Topological sort to find execution order
    const sorted = this.topologicalSort();

    // Group tasks by phase (parallel groups)
    let currentPhase: Task[] = [];
    const completed = new Set<TaskId>();

    for (const taskId of sorted) {
      const node = this.nodes.get(taskId)!;
      const deps = this.edges.get(taskId) || [];

      // Check if all dependencies completed
      const allDepsComplete = deps.every(dep => completed.has(dep));

      if (allDepsComplete) {
        // Can run in current phase
        currentPhase.push(node.task);
      } else {
        // Must wait for next phase
        if (currentPhase.length > 0) {
          plan.phases.push({
            type: currentPhase.length === 1 ? 'sequential' : 'parallel',
            tasks: currentPhase
          });
          currentPhase = [];
        }
        currentPhase.push(node.task);
      }

      completed.add(taskId);
    }

    // Add final phase
    if (currentPhase.length > 0) {
      plan.phases.push({
        type: currentPhase.length === 1 ? 'sequential' : 'parallel',
        tasks: currentPhase
      });
    }

    return plan;
  }

  /**
   * Topological sort (Kahn's algorithm)
   */
  private topologicalSort(): TaskId[] {
    const result: TaskId[] = [];
    const inDegree = new Map<TaskId, number>();

    // Calculate in-degree for each node
    for (const [taskId] of this.nodes) {
      inDegree.set(taskId, 0);
    }

    for (const [, deps] of this.edges) {
      for (const dep of deps) {
        inDegree.set(dep, (inDegree.get(dep) || 0) + 1);
      }
    }

    // Queue nodes with no dependencies
    const queue: TaskId[] = [];
    for (const [taskId, degree] of inDegree) {
      if (degree === 0) {
        queue.push(taskId);
      }
    }

    // Process queue
    while (queue.length > 0) {
      const taskId = queue.shift()!;
      result.push(taskId);

      // Reduce in-degree for dependent tasks
      const deps = this.edges.get(taskId) || [];
      for (const dep of deps) {
        const newDegree = (inDegree.get(dep) || 0) - 1;
        inDegree.set(dep, newDegree);

        if (newDegree === 0) {
          queue.push(dep);
        }
      }
    }

    // Check for cycles
    if (result.length !== this.nodes.size) {
      throw new Error('Circular dependency detected in task graph');
    }

    return result;
  }

  /**
   * Visualize dependency graph
   */
  visualize(): string {
    let graph = 'Task Dependency Graph:\n\n';

    for (const [taskId, deps] of this.edges) {
      const task = this.nodes.get(taskId)!.task;
      graph += `[${taskId}] ${task.description}\n`;

      if (deps.length > 0) {
        graph += `  Dependencies:\n`;
        for (const dep of deps) {
          const depTask = this.nodes.get(dep)!.task;
          graph += `    ← [${dep}] ${depTask.description}\n`;
        }
      } else {
        graph += `  No dependencies (can start immediately)\n`;
      }

      graph += '\n';
    }

    return graph;
  }
}

// Usage example
const taskGraph = new TaskGraph();

// Add tasks with dependencies
taskGraph.addTask(
  { id: 'define-contract', description: 'Define API contract' },
  []  // No dependencies
);

taskGraph.addTask(
  { id: 'implement-backend', description: 'Implement backend' },
  ['define-contract']  // Depends on contract
);

taskGraph.addTask(
  { id: 'implement-frontend', description: 'Implement frontend' },
  ['define-contract']  // Depends on contract
);

taskGraph.addTask(
  { id: 'integration', description: 'Integrate and test' },
  ['implement-backend', 'implement-frontend']  // Depends on both
);

// Build execution plan
const plan = taskGraph.buildExecutionPlan();

console.log(plan);
/*
{
  phases: [
    {
      type: 'sequential',
      tasks: [{ id: 'define-contract', ... }]
    },
    {
      type: 'parallel',  // Backend and frontend can run together!
      tasks: [
        { id: 'implement-backend', ... },
        { id: 'implement-frontend', ... }
      ]
    },
    {
      type: 'sequential',
      tasks: [{ id: 'integration', ... }]
    }
  ]
}
*/

// Visualize
console.log(taskGraph.visualize());
/*
Task Dependency Graph:

[define-contract] Define API contract
  No dependencies (can start immediately)

[implement-backend] Implement backend
  Dependencies:
    ← [define-contract] Define API contract

[implement-frontend] Implement frontend
  Dependencies:
    ← [define-contract] Define API contract

[integration] Integrate and test
  Dependencies:
    ← [implement-backend] Implement backend
    ← [implement-frontend] Implement frontend
*/
```

---

## 🎯 Advanced Scenarios

### Scenario 1: Massive Parallel Search

**Use Case:** "Find all authentication implementations in our codebase and compare with popular open-source projects"

```typescript
// OmO orchestrates massive parallel search
async function massiveParallelSearch() {
  console.log("🔍 Starting massive parallel search for authentication patterns...");

  // Spawn multiple background searches
  const searches = [
    // Search 1: Local codebase
    backgroundTaskManager.spawnBackground(
      agent: agents.get('explore'),
      task: {
        description: "Search local codebase for auth patterns",
        tools: ['ast-grep', 'lsp'],
        patterns: [
          'authentication',
          'login',
          'jwt',
          'session',
          'oauth',
          'password'
        ],
        scope: 'entire codebase'
      }
    ),

    // Search 2: GitHub public repos
    backgroundTaskManager.spawnBackground(
      agent: agents.get('librarian'),
      task: {
        description: "Search GitHub for auth implementations",
        mcp: 'grep_app',
        queries: [
          'express authentication middleware',
          'react auth context',
          'jwt implementation',
          'oauth2 flow'
        ],
        filters: {
          stars: '>1000',
          language: ['typescript', 'javascript']
        }
      }
    ),

    // Search 3: Official documentation
    backgroundTaskManager.spawnBackground(
      agent: agents.get('librarian'),
      task: {
        description: "Fetch auth documentation",
        mcp: 'context7',
        libraries: [
          'passport',
          'next-auth',
          'express-session',
          'jsonwebtoken'
        ]
      }
    ),

    // Search 4: Security best practices
    backgroundTaskManager.spawnBackground(
      agent: agents.get('oracle'),
      task: {
        description: "Analyze auth security patterns",
        focus: 'security',
        aspects: [
          'OWASP auth guidelines',
          'JWT security',
          'Session management',
          'Password hashing'
        ]
      }
    )
  ];

  console.log(`🚀 Spawned ${searches.length} parallel searches`);
  console.log("⏳ Waiting for results...");

  // Wait for all searches to complete
  const results = await backgroundTaskManager.waitFor(searches);

  // Aggregate results
  const aggregated = {
    local: results[0],      // Local codebase patterns
    github: results[1],     // Open-source implementations
    docs: results[2],       // Official documentation
    security: results[3]    // Security analysis
  };

  // Synthesize findings
  console.log("\n📊 Search Results:\n");
  console.log(`Local implementations: ${aggregated.local.count}`);
  console.log(`GitHub examples: ${aggregated.github.count}`);
  console.log(`Documentation sources: ${aggregated.docs.count}`);
  console.log(`Security recommendations: ${aggregated.security.recommendations.length}`);

  // OmO analyzes and provides recommendations
  const recommendations = await analyzeAuthPatterns(aggregated);

  return recommendations;
}

// Results
{
  timeline: {
    sequential: "12 minutes",  // If done one by one
    parallel: "3 minutes",     // With background agents
    speedup: "4x faster"
  },

  findings: {
    local: {
      implementations: 15,
      patterns: ['JWT', 'Session', 'OAuth2'],
      issues: ['Weak password hashing in UserService.ts']
    },

    github: {
      examples: 50,
      topRepos: [
        'nextauthjs/next-auth',
        'jaredhanson/passport',
        'auth0/node-jsonwebtoken'
      ],
      commonPatterns: ['Middleware-based', 'Context providers', 'Hook-based']
    },

    docs: {
      libraries: 4,
      bestPractices: [
        'Use httpOnly cookies for tokens',
        'Implement CSRF protection',
        'Rate limit login attempts'
      ]
    },

    security: {
      vulnerabilities: [
        'No rate limiting on /login endpoint',
        'JWT secrets in environment variables (should use secrets manager)'
      ],
      recommendations: [
        'Implement refresh token rotation',
        'Add 2FA support',
        'Use bcrypt with salt rounds >= 12'
      ]
    }
  },

  recommendation: `
Based on analysis of 15 local implementations, 50 GitHub examples,
and security best practices:

1. Standardize on JWT + refresh tokens
2. Implement rate limiting immediately (security risk)
3. Move secrets to AWS Secrets Manager
4. Add 2FA using speakeasy library
5. Refactor UserService.ts password hashing

Estimated implementation time: 4 hours
Priority: HIGH (security vulnerabilities found)
  `
}
```

---

### Scenario 2: Multi-Approach Debugging

**Use Case:** "Why is login failing intermittently? It works sometimes, fails other times."

```typescript
// OmO uses multiple agents with different debugging approaches
async function multiApproachDebugging() {
  console.log("🐛 Investigating intermittent login failures...");

  // Spawn multiple debugging approaches in parallel
  const investigations = [
    // Approach 1: Pattern search (fast)
    backgroundTaskManager.spawnBackground(
      agent: agents.get('explore'),
      task: {
        description: "Quick pattern search for auth errors",
        actions: [
          'Search logs for auth errors',
          'Find recent auth-related changes in git',
          'Check for timeout patterns'
        ],
        timeLimit: '1 minute'
      }
    ),

    // Approach 2: Known issues research
    backgroundTaskManager.spawnBackground(
      agent: agents.get('librarian'),
      task: {
        description: "Research known auth issues",
        sources: [
          'GitHub issues for passport library',
          'Stack Overflow questions on intermittent auth',
          'Check if JWT library has version bugs'
        ],
        timeLimit: '2 minutes'
      }
    ),

    // Approach 3: Deep logical analysis
    backgroundTaskManager.spawnBackground(
      agent: agents.get('oracle'),
      task: {
        description: "Analyze auth flow for race conditions",
        focus: [
          'Session store implementation',
          'JWT token generation timing',
          'Database connection pooling',
          'Race conditions in middleware'
        ],
        thinkingBudget: '16k tokens',
        timeLimit: '5 minutes'
      }
    )
  ];

  console.log("⏳ Running 3 parallel investigations...");

  // Collect results as they complete
  const results = await backgroundTaskManager.waitFor(investigations);

  // Synthesize findings
  console.log("\n🔍 Investigation Results:\n");

  console.log("Approach 1 (Pattern Search):");
  console.log(results[0].findings);
  /*
    - Found timeout errors in auth.log
    - Last change to auth: JWT library upgrade to v2.3.0
    - Pattern: Failures increase during high traffic
  */

  console.log("\nApproach 2 (Known Issues):");
  console.log(results[1].findings);
  /*
    - GitHub Issue #456 in JWT library v2.3.0: "Intermittent verification failures"
    - Stack Overflow: Multiple reports of session store race conditions
    - Known bug in JWT v2.3.0, fixed in v2.3.1
  */

  console.log("\nApproach 3 (Deep Analysis):");
  console.log(results[2].findings);
  /*
    - Session store (Redis) not using connection pooling correctly
    - Race condition identified: Session write vs token verification
    - Token verification happens before session fully persisted
    - Probability of failure increases with concurrent logins
  */

  // OmO synthesizes all findings
  const rootCause = {
    primary: "JWT library v2.3.0 has known bug causing intermittent verification failures",
    secondary: "Session store race condition - token verified before session persisted",

    confidence: "HIGH",
    evidence: [
      "3 different approaches identified same components (JWT + Session)",
      "GitHub issue confirms JWT v2.3.0 bug",
      "Log pattern matches timing issue",
      "Oracle identified race condition"
    ],

    solution: [
      "Immediate: Downgrade JWT to v2.2.0 or upgrade to v2.3.1",
      "Short-term: Add retry logic with exponential backoff",
      "Long-term: Fix session store to use proper locking"
    ],

    timeline: {
      investigation: "5 minutes (vs 15+ minutes sequential)",
      fix: "30 minutes",
      testing: "1 hour"
    }
  };

  return rootCause;
}

/*
Timeline comparison:

Sequential debugging:
├─ Try pattern search: 5 min → Find some clues
├─ Research issues: 5 min → Find GitHub issue
├─ Deep analysis: 8 min → Confirm race condition
└─ Total: 18 minutes, might miss correlations

Parallel debugging (Oh My OpenCode):
├─ All 3 approaches: 5 min (longest one)
├─ Synthesis: 1 min
└─ Total: 6 minutes
   - 3x faster
   - Higher confidence (multiple confirmations)
   - More comprehensive (3 perspectives)
*/
```

---

### Scenario 3: Incremental Delivery

**Use Case:** "Refactor entire auth system to use JWT instead of sessions"

This scenario shows how background agents enable incremental delivery and early user feedback.

```typescript
// Traditional approach: Big bang delivery
async function traditionalRefactoring() {
  console.log("Starting auth refactoring...");

  // User waits for everything
  const research = await doResearch();         // 10 min
  const plan = await createPlan(research);     // 15 min
  const implementation = await implement(plan); // 60 min
  const testing = await runTests(implementation); // 20 min

  console.log("✅ Done! Here's everything.");

  return {
    totalTime: "105 minutes",
    userFeedback: "Can't give feedback until end",
    riskOfWastedWork: "HIGH - user might want different approach"
  };
}

// Oh My OpenCode approach: Incremental delivery
async function incrementalRefactoring() {
  console.log("🚀 Starting incremental auth refactoring...");

  // Phase 1: Research (background)
  const researchHandle = backgroundTaskManager.spawnBackground(
    agent: agents.get('librarian'),
    task: {
      description: "Research JWT best practices and migration strategies",
      deliverables: [
        'JWT vs Session comparison',
        'Migration strategies',
        'Security considerations',
        'Code examples'
      ]
    }
  );

  console.log("📚 Research started in background...");
  console.log("⏳ I'll notify you when research complete (~10 min)");

  // Phase 2: Architecture (background, depends on research)
  researchHandle.onComplete(async (research) => {
    console.log("\n✅ Research complete! Here's the summary:");
    console.log(research.summary);
    console.log("\n📋 Full report available at: ./docs/jwt-research.md");

    // User can review and give feedback now!
    const userApproval = await askUser("Does this approach look good?");

    if (userApproval) {
      // Start architecture design
      const designHandle = backgroundTaskManager.spawnBackground(
        agent: agents.get('oracle'),
        task: {
          description: "Design new JWT-based auth architecture",
          context: research,
          deliverables: [
            'Architecture diagram',
            'Migration plan',
            'Risk assessment',
            'Implementation phases'
          ]
        }
      );

      console.log("🏗️  Architecture design started...");
      console.log("⏳ Will notify when complete (~15 min)");

      // Phase 3: Implementation (background, depends on design)
      designHandle.onComplete(async (design) => {
        console.log("\n✅ Architecture design complete!");
        console.log(design.summary);
        console.log("\n📐 Full design at: ./docs/jwt-architecture.md");

        // User can review architecture before implementation
        const designApproval = await askUser("Approve this architecture?");

        if (designApproval) {
          // Start implementation with progress updates
          const implHandle = backgroundTaskManager.spawnBackground(
            agent: agents.get('OmO'),
            task: {
              description: "Implement JWT auth system",
              context: { research, design },
              phases: [
                'JWT token service',
                'Auth middleware refactor',
                'Migration script',
                'Update tests'
              ],
              progressUpdates: true  // Enable incremental updates
            }
          );

          // Progress updates during implementation
          implHandle.onProgress((progress) => {
            console.log(`\n📊 Progress: ${progress.percentage}%`);
            console.log(`   Current: ${progress.currentPhase}`);
            console.log(`   Completed: ${progress.completed.join(', ')}`);

            // User sees intermediate results!
            if (progress.completed.length > 0) {
              console.log(`   ✅ You can review completed parts now!`);
            }
          });

          implHandle.onComplete(async (implementation) => {
            console.log("\n✅ Implementation complete!");
            console.log(`   Files changed: ${implementation.filesChanged}`);
            console.log(`   Tests: ${implementation.tests.passed}/${implementation.tests.total} passing`);
            console.log("\n🚀 Ready for final review and deployment!");
          });
        } else {
          console.log("📝 Let's revise the architecture...");
        }
      });
    } else {
      console.log("📝 Let's adjust the research direction...");
    }
  });

  // Meanwhile, OmO can work on other tasks!
  console.log("\n💡 While research runs, I can help with other tasks.");
  console.log("   Just ask if you need anything!");
}

/*
Timeline and User Experience Comparison:

Traditional (Big Bang):
T=0:    Start work
T=105:  "Here's everything, hope you like it!"
        └─ User sees nothing for 105 minutes
        └─ If approach wrong, wasted 105 minutes

Incremental (Oh My OpenCode):
T=0:    Start research (background)
T=10:   ✅ "Research done! Review?"
        └─ User reviews, approves/adjusts
        └─ Early course correction possible!

T=10:   Start architecture (background)
T=25:   ✅ "Architecture done! Review?"
        └─ User reviews, approves/adjusts
        └─ Catch design issues early!

T=25:   Start implementation (background)
T=40:   📊 "30% done - JWT service ready"
        └─ User can test JWT service now!

T=55:   📊 "60% done - Middleware refactored"
        └─ User can test auth flow now!

T=70:   📊 "90% done - Migration script ready"
        └─ User can test migration now!

T=85:   ✅ "100% done - All tests passing"
        └─ Final review, already tested incrementally

Total time: 85 minutes (20 min faster)
User experience: Much better!
  - Sees progress continuously
  - Can course-correct early
  - Tests incrementally
  - Lower risk of wasted work
  - Feels more involved in process
*/
```

---

## 🚀 Performance Characteristics

### Resource Usage Comparison

```yaml
Sequential Model (Traditional):
  CPU Utilization: 20-30% average
    Reason: Waiting for API responses, idle most of time

  Memory Usage: ~500MB
    Reason: Single agent context in memory

  Network: Bursty pattern
    ├─ API call → Wait for response → Idle
    └─ Repeat
    Result: Inefficient use of available bandwidth

  Latency: High
    Reason: Serial API calls, each waits for previous

  Throughput: Low
    Formula: 1 task / avg_task_time
    Example: 1 task / 15min = 4 tasks/hour

Parallel Model (Oh My OpenCode):
  CPU Utilization: 60-80% average
    Reason: Multiple agents actively working

  Memory Usage: ~2-3GB
    Reason: 3-4 agent contexts in memory
    Note: Still acceptable for modern machines

  Network: Steady pattern
    ├─ Multiple concurrent API calls
    └─ Continuous utilization
    Result: Efficient use of available bandwidth

  Latency: Lower
    Reason: Parallel API calls, overlapping wait times

  Throughput: High
    Formula: n tasks / max(task_times)
    Example: 3 tasks / 15min = 12 tasks/hour
    Improvement: 3x better throughput

Efficiency Gain:
  Time: 2-4x faster (depending on parallelizability)
  Cost: Similar (same total token usage)
  Quality: Better (multiple perspectives)
  UX: Much better (incremental delivery)
```

### Token Consumption Analysis

```typescript
// Intelligent context sharing reduces total tokens
interface TokenUsage {
  sequential: {
    agent1: {
      context: 8000,    // Full project context
      task: 2000,       // Task-specific
      total: 10000
    },
    agent2: {
      context: 8000,    // Duplicate full context!
      task: 2000,
      total: 10000
    },
    agent3: {
      context: 8000,    // Duplicate again!
      task: 2000,
      total: 10000
    },
    totalTokens: 30000,
    waste: 16000        // Wasted on duplicate contexts
  },

  parallel: {
    sharedContext: {
      project: 8000,    // Loaded once, shared by all
      cached: true
    },
    agent1: {
      shared: 0,        // Reuses cached context
      task: 2000,
      total: 2000
    },
    agent2: {
      shared: 0,
      task: 2000,
      total: 2000
    },
    agent3: {
      shared: 0,
      task: 2000,
      total: 2000
    },
    totalTokens: 14000,  // 8000 shared + 6000 tasks
    savings: 16000,      // 53% less tokens!

    bonus: {
      betterCoordination: "Agents see each other's outputs",
      consistency: "All agents use same context version",
      efficiency: "Faster execution + less tokens = win-win"
    }
  }
}
```

### Scalability Characteristics

```typescript
// How performance scales with number of tasks
interface ScalabilityMetrics {
  tasks_1: {
    sequential: "10 min",
    parallel: "10 min",
    speedup: "1x (no difference)"
  },

  tasks_2: {
    sequential: "20 min",
    parallel: "10 min",    // If independent
    speedup: "2x"
  },

  tasks_3: {
    sequential: "30 min",
    parallel: "10 min",    // If all independent
    speedup: "3x"
  },

  tasks_10: {
    sequential: "100 min",
    parallel: "25 min",    // Limited by dependencies
    speedup: "4x"
  },

  limitingFactors: {
    apiRateLimits: "Max ~5-10 concurrent requests per provider",
    memory: "Each agent uses ~500MB, limit ~4-6 agents",
    dependencies: "Can't parallelize dependent tasks",
    coordination: "More agents = more coordination overhead"
  },

  sweetSpot: {
    tasks: "3-5 parallel tasks",
    agents: "3-4 concurrent agents",
    speedup: "2.5-3x average",
    efficiency: "Optimal cost/performance ratio"
  }
}
```

---

## 🎓 Kết luận

### Cơ chế Background Agents của Oh My OpenCode

Đây là một hệ thống phức tạp và tinh vi, hoạt động như một **development team thực sự**:

#### 1. **OmO Orchestrator = Tech Lead**
```yaml
Responsibilities:
  - Analyze requirements và break down tasks
  - Select right specialist cho từng task
  - Decide execution strategy (parallel vs sequential)
  - Monitor progress và coordinate
  - Integrate results và ensure quality

Skills:
  - Extended thinking (32k token budget)
  - Multi-agent coordination
  - Context management
  - Quality assurance
```

#### 2. **Specialized Agents = Team Members**
```yaml
Characteristics:
  - Each có expertise riêng biệt
  - Work independently nhưng coordinated
  - Communicate via shared context
  - Deliver incrementally với progress updates

Examples:
  - oracle: Architecture & debugging expert
  - librarian: Research & documentation specialist
  - frontend-ui-ux-engineer: Creative UI designer
  - explore: Fast codebase navigation
```

#### 3. **Background Task System = Async Workflow**
```yaml
Features:
  - Non-blocking execution
  - Parallel task processing
  - Event-driven notifications
  - Progress tracking
  - Graceful error handling

Benefits:
  - Maximize resource utilization
  - Reduce wall-clock time
  - Enable incremental delivery
  - Better user experience
```

### Kết quả đạt được

```yaml
Performance:
  Speed: 2-4x faster through parallelization
  Efficiency: 10-15% token savings through context sharing
  Quality: Better through multiple specialized perspectives

User Experience:
  Transparency: Real-time progress updates
  Control: Early feedback and course correction
  Confidence: Incremental testing and validation
  Engagement: Continuous visibility into work

Development Process:
  Flexibility: Easy to adjust mid-execution
  Scalability: Handle complex multi-domain tasks
  Reliability: Error recovery and retry mechanisms
  Maintainability: Clear separation of concerns
```

### Đây chính là tương lai của AI-assisted development! 🚀

**Traditional:** 1 agent, sequential, slow, frustrating
**Oh My OpenCode:** Team of specialists, parallel, fast, delightful

---

## 📚 Related Documentation

- [README.md](../README.md) - Project overview
- [Agent Configuration](./agent-configuration.md) - How to configure agents
- [MCP Integration](./mcp-integration.md) - Using MCP servers
- [Performance Tuning](./performance-tuning.md) - Optimization tips

---

**Tác giả:** Oh My OpenCode Team
**Cập nhật:** 2025-01-18
**Phiên bản:** 1.0.0
