# Exploit Research Memory System

A comprehensive memory system for autonomous exploit research that provides persistence, retrieval, deduplication, and feedback routing for DeFi security analysis.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the System](#running-the-system)
- [Quick Start](#quick-start)
- [Core Concepts](#core-concepts)
- [API Reference](#api-reference)
- [Modules](#modules)
- [Usage Examples](#usage-examples)
- [Best Practices](#best-practices)

---

## Overview

The Exploit Research Memory System solves critical problems in autonomous exploit research:

- **Context overflow resets** - Survives restarts with persistent storage
- **Lost findings** - All discoveries are stored and retrievable
- **Repeated hypotheses** - Deduplication prevents redundant work
- **No feedback routing** - Execution results update upstream entities
- **Volatile caches** - Replace in-memory with persistent storage

### Key Features

| Feature | Description |
|---------|-------------|
| **Multi-tenant isolation** | Hard protocol_id + season_id boundaries |
| **Semantic retrieval** | Embedding-based similarity search |
| **Content reduction** | Reducers shrink outputs before LLM consumption |
| **Dedup & novelty** | Prevent repeated hypotheses/tests |
| **Feedback routing** | Test results update linked entities |
| **Secret redaction** | Automatic removal of sensitive data |
| **Metrics tracking** | Monitor efficiency and effectiveness |

---

## Installation

### Prerequisites

- **Node.js** >= 20.0.0
- **Bun** >= 1.2.17 (recommended) or npm/yarn
- **Git** (for cloning)

### 1. Clone the Repository

```bash
git clone https://github.com/supermemoryai/supermemory.git
cd supermemory
```

### 2. Install Dependencies

```bash
# Using Bun (recommended)
bun install

# Or using npm
npm install
```

### 3. Build the Packages

```bash
# Build all packages
bun run build

# Or build only the tools package
cd packages/tools && bun run build
```

### 4. Environment Setup

Create a `.env` file in your project root:

```bash
# OpenRouter API (Required for AI features)
OPENROUTER_API_KEY=your-openrouter-api-key

# Optional - Supermemory API for persistence
SUPERMEMORY_API_KEY=your-supermemory-api-key
SUPERMEMORY_BASE_URL=https://api.supermemory.com
```

### AI Models (via OpenRouter)

| Model Type | Model ID | Dimensions |
|------------|----------|------------|
| **Embeddings** | `qwen/qwen3-embedding-8b` | 4096 |
| **LLM** | `openai/gpt-5.1` | - |

### 5. Import the Module

```typescript
// From the tools package (internal usage)
import {
  // OpenRouter AI Client
  createOpenRouterClient,
  OpenRouterClient,
  OPENROUTER_CONFIG,

  // Reducers
  reduceByKind,
  reduceSourceCode,
  reduceABI,
  reduceForgeLogs,

  // Dedup
  checkDuplicate,
  calculateNoveltyScore,
  filterDuplicates,
  cosineSimilarity,

  // Middleware
  createExploitResearchMiddleware,

  // Security
  redactSecrets,
  processArtifactForSecurity,
  sanitizeContent,

  // Metrics
  createMetricsCollector,
  MetricsCollector,
} from "@supermemory/tools/exploit-research"

// Validation schemas (from validation package)
import {
  ProtocolSchema,
  SeasonSchema,
  InvariantSchema,
  PathSchema,
  PhaseEnum,
  ArtifactKindEnum,
} from "@supermemory/validation/exploit-research-schemas"

import {
  LogEventRequestSchema,
  RetrieveRequestSchema,
  StoreArtifactRequestSchema,
} from "@supermemory/validation/exploit-research-api"
```

---

## Configuration

### OpenRouter Configuration (AI Client)

```typescript
import { createOpenRouterClient } from "@supermemory/tools/exploit-research"

const openrouter = createOpenRouterClient({
  // Required: Your OpenRouter API key
  apiKey: process.env.OPENROUTER_API_KEY!,

  // Optional: Override models (defaults shown)
  embeddingModel: "qwen/qwen3-embedding-8b",  // 4096 dimensions
  llmModel: "openai/gpt-5.1",

  // Optional: Site info for OpenRouter dashboard
  siteUrl: "https://your-site.com",
  siteName: "Your App Name",

  // Optional: Enable verbose logging
  verbose: true,
})

// Generate embeddings
const embedding = await openrouter.embed("smart contract code here")
console.log(`Embedding dimensions: ${embedding.length}`)  // 4096

// Chat with LLM
const response = await openrouter.chat("Analyze this contract for vulnerabilities", {
  systemPrompt: "You are a smart contract security expert",
  maxTokens: 4096,
  temperature: 0.3,
})

// Analyze content
const analysis = await openrouter.analyze(contractCode, "vulnerability")

// Generate exploit test
const testCode = await openrouter.generateExploitTest({
  targetContract: "Pool",
  vulnerability: "Reentrancy in withdraw function",
  attackPath: ["Deposit ETH", "Call withdraw", "Reenter in fallback"],
  expectedProfit: 100000,
})

// Check similarity (dedup)
const { isDuplicate, similarity } = await openrouter.checkSimilarity(
  newHypothesis,
  existingEmbeddings,
  0.85  // threshold
)
```

### Middleware Configuration

```typescript
import { createExploitResearchMiddleware } from "@supermemory/tools/exploit-research"

const middleware = createExploitResearchMiddleware({
  // Required: Your Supermemory API key
  apiKey: process.env.SUPERMEMORY_API_KEY!,

  // Optional: Override the API base URL (default: https://api.supermemory.com)
  baseUrl: process.env.SUPERMEMORY_BASE_URL,

  // Optional: Enable verbose logging (default: false)
  verbose: true,

  // Optional: Automatically generate embeddings (default: true)
  autoEmbed: true,
})
```

### Dedup Configuration

```typescript
import { checkDuplicate, calculateNoveltyScore } from "@supermemory/tools/exploit-research"

// Configure dedup thresholds
const dedupConfig = {
  // Similarity threshold for exact duplicates (default: 0.92)
  duplicateThreshold: 0.92,

  // Similarity threshold for "similar" content (default: 0.85)
  similarityThreshold: 0.85,

  // Novelty penalties for overused attack patterns
  noveltyPenalties: {
    first_depositor: 0.3,    // 30% penalty
    simple_mev: 0.25,
    basic_flash_loan: 0.2,
    oracle_manipulation: 0.15,
    reentrancy: 0.1,
  },
}

// Use with dedup functions
const result = checkDuplicate(newEmbedding, existingEmbeddings, dedupConfig)
```

### Sanitization Configuration

```typescript
import { sanitizeContent } from "@supermemory/tools/exploit-research"

const sanitizeOptions = {
  // Redact API keys, private keys, etc. (default: true)
  redactSecrets: true,

  // Remove internal file paths like /home/user/* (default: true)
  removeInternalPaths: true,

  // Truncate content to max length (default: undefined = no limit)
  maxLength: 10000,
}

const result = sanitizeContent(content, sanitizeOptions)
```

---

## Running the System

### Run Tests

```bash
# Run all exploit-research tests
bun test packages/tools/src/exploit-research/__tests__/

# Run specific test suites
bun test packages/tools/src/exploit-research/__tests__/reducers.test.ts
bun test packages/tools/src/exploit-research/__tests__/dedup.test.ts
bun test packages/tools/src/exploit-research/__tests__/security.test.ts
bun test packages/tools/src/exploit-research/__tests__/metrics.test.ts
bun test packages/tools/src/exploit-research/__tests__/middleware.test.ts

# Run schema validation tests
bun test packages/validation/__tests__/exploit-research-schemas.test.ts
bun test packages/validation/__tests__/exploit-research-api.test.ts

# Run with watch mode
bun test --watch packages/tools/src/exploit-research/__tests__/

# Run with verbose output
bun test --reporter=verbose packages/tools/src/exploit-research/__tests__/
```

### Type Checking

```bash
# Check types for the tools package
cd packages/tools && bun run check-types

# Or from root
bun run check-types
```

### Development Mode

```bash
# Watch and rebuild on changes
cd packages/tools && bun run dev
```

---

## Quick Start

### 1. Create a Season

```typescript
import { CreateSeasonRequestSchema } from "@supermemory/validation/exploit-research-api"

const createRequest = CreateSeasonRequestSchema.parse({
  protocolId: "uniswap-v3",
  metadata: { purpose: "mainnet audit" },
})

// Response: { id: "uniswap-v3:season-20241205-abc123", status: "active", ... }
```

### 2. Set Up Middleware

```typescript
import { createExploitResearchMiddleware } from "@supermemory/tools/exploit-research"

const middleware = createExploitResearchMiddleware({
  apiKey: process.env.SUPERMEMORY_API_KEY,
  baseUrl: "https://api.supermemory.com",
  verbose: true,
})

const context = {
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc123",
  phase: "invariant" as const,
}
```

### 3. Wrap Tool Executors

```typescript
// Wrap any tool to automatically store outputs
const wrappedFetchSource = middleware.wrapToolExecutor(
  "etherscan_fetch_source",
  "source",
  async (input: { address: string }) => {
    // Your tool implementation
    return await etherscan.getSource(input.address)
  },
  {
    extractTarget: (input) => ({ contract: input.address }),
    shouldReduce: true,
  }
)

// Use the wrapped tool
const source = await wrappedFetchSource({ address: "0x..." }, context)
```

### 4. Check for Duplicates

```typescript
const { isDuplicate, similarity } = await middleware.checkDuplicate(
  "hypothesis",
  "Flash loan attack on swap function to drain reserves",
  context,
  0.85 // threshold
)

if (isDuplicate) {
  console.log(`Similar hypothesis exists (${similarity * 100}% match)`)
}
```

### 5. Retrieve Context

```typescript
const results = await middleware.retrieveContext(
  "price manipulation oracle attack",
  context,
  {
    kind: "invariant",
    limit: 10,
    threshold: 0.6,
  }
)
```

---

## Core Concepts

### Identity & Isolation

Every entity requires `protocol_id` and `season_id`:

```
protocol_id: uniswap-v3
season_id: uniswap-v3:season-20241205-a1b2c3d4
graph_id: uniswap-v3:season-20241205-a1b2c3d4:graph-sha256abc
invariant_id: uniswap-v3:season-20241205-a1b2c3d4:inv-arithmetic-sha256xyz
```

### Phase Flow

```
intake → graphing → invariant → violation → poc_generation → execution → refinement
```

### Entity Relationships

```
Protocol
  └── Season
        ├── Contract → Graph
        │                └── Invariant
        │                      └── Path
        │                            └── Scenario
        │                                  └── Test
        │                                        └── Execution
        └── Artifacts, ToolEvents, Questions, Feedback
```

---

## API Reference

### 1. log_event

Store tool call events with metadata.

```typescript
import { LogEventRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = LogEventRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  phase: "invariant",
  toolName: "etherscan_fetch_source",
  toolCallId: "call-123",
  targetContract: "0x1234...",
  success: true,
  duration: 1500,
  inputTokens: 100,
  outputTokens: 500,
  rawContent: "contract Pool { ... }",
  summaryContent: "## Contract Outline\n...",
})

// Response: { id, rawRef, summaryRef, embeddingId }
```

### 2. store_artifact

Store any artifact with optional embedding.

```typescript
import { StoreArtifactRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = StoreArtifactRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  kind: "invariant",
  raw: JSON.stringify({ category: "arithmetic", formula: "x * y >= k" }),
  summary: "Constant product invariant for AMM",
  metadata: { severity: "critical", confidence: 0.95 },
  retainDays: 30,
})

// Response: { id, embeddingId, rawHash }
```

### 3. retrieve

Semantic search with filters.

```typescript
import { RetrieveRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = RetrieveRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  query: "flash loan vulnerability swap function",
  filters: {
    phase: "violation",
    kind: "path",
    contract: "0x1234...",
    createdAfter: new Date("2024-12-01"),
  },
  limit: 10,
  threshold: 0.6,
  includeSummary: true,
  includeRaw: false,
})

// Response: { results: [...], total, timing }
```

### 4. similar

Check for duplicates before creating new entities.

```typescript
import { SimilarRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = SimilarRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  kind: "hypothesis",
  text: "Flash loan attack on swap to drain liquidity",
  threshold: 0.85,
  limit: 5,
})

// Response: { results, isDuplicate, maxSimilarity }
```

### 5. link

Create relationships between entities.

```typescript
import { LinkRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = LinkRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  parentId: "inv-arithmetic-001",
  childId: "path-flash-001",
  relation: "violates",
  confidence: 0.9,
  notes: "Path exploits invariant via price manipulation",
})

// Response: { id, success }
```

### 6. record_feedback

Route test results to update linked entities.

```typescript
import { RecordFeedbackRequestSchema } from "@supermemory/validation/exploit-research-api"

const request = RecordFeedbackRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  testId: "test-001",
  status: "pass",
  profit: 50000,
  linkedPathId: "path-001",
  linkedScenarioId: "scenario-001",
  linkedInvariantIds: ["inv-001", "inv-002"],
  forgeLogsContent: "[PASS] testExploit (2.1s)\nProfit: 50000 USDC",
})

// Response: { id, updatedEntities: [...] }
```

### 7. snapshot / restore

Persist and restore season state.

```typescript
import { SnapshotRequestSchema, RestoreRequestSchema } from "@supermemory/validation/exploit-research-api"

// Create snapshot
const snapshotRequest = SnapshotRequestSchema.parse({
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  metadata: { reason: "pre-refinement" },
})
// Response: { id, seasonId, createdAt }

// Restore from snapshot
const restoreRequest = RestoreRequestSchema.parse({
  protocolId: "uniswap-v3",
  snapshotId: "snapshot-001",
})
// Response: { success, restoredSeasonId, restoredEntities }
```

---

## Modules

### Reducers

Transform verbose outputs into compact summaries.

```typescript
import {
  reduceSourceCode,
  reduceABI,
  reduceGraph,
  reduceInvariants,
  reducePath,
  reduceForgeLogs,
  reduceAddressList,
  reduceGeneric,
  reduceByKind,
} from "@supermemory/tools/exploit-research"

// Auto-dispatch by kind
const result = reduceByKind("source", solidityCode)
// Returns: { summary, shouldStoreRaw, extractedMetadata }

// Or use specific reducers
const sourceReduced = reduceSourceCode(solidityCode)
// Extracts: functions, state vars, modifiers, requires, events, errors

const abiReduced = reduceABI(abiJson)
// Extracts: write/read functions, events, errors

const logsReduced = reduceForgeLogs(forgeOutput)
// Extracts: test results, reverts, profits, errors
```

### Dedup & Novelty

Prevent repeated hypotheses and tests.

```typescript
import {
  cosineSimilarity,
  checkDuplicate,
  calculateNoveltyScore,
  filterDuplicates,
  findMostSimilar,
  calculateDiversity,
} from "@supermemory/tools/exploit-research"

// Check if embedding is a duplicate
const result = checkDuplicate(newEmbedding, existingEmbeddings, {
  duplicateThreshold: 0.92,
  similarityThreshold: 0.85,
})
// Returns: { isDuplicate, maxSimilarity, duplicateOf, noveltyScore }

// Calculate novelty with pattern penalties
const novelty = calculateNoveltyScore(
  embedding,
  existingEmbeddings,
  "first_depositor", // Penalized pattern
  { noveltyPenalties: { first_depositor: 0.3 } }
)

// Filter batch for novel items only
const novel = filterDuplicates(candidates, existing)
// Returns only non-duplicate candidates with novelty scores

// Find most similar to a query
const similar = findMostSimilar(queryEmbedding, existing, 5, 0.5)
// Returns top 5 with similarity >= 0.5

// Calculate diversity of a set
const diversity = calculateDiversity(embeddings)
// Higher = more diverse
```

### Security

Redact secrets and mark sensitive content.

```typescript
import {
  redactSecrets,
  containsSecrets,
  checkSensitivity,
  sanitizeContent,
  processArtifactForSecurity,
} from "@supermemory/tools/exploit-research"

// Redact all secret types
const redacted = redactSecrets(content)
// Returns: { redacted, redactedCount, redactedTypes }

// Supported patterns:
// - Infura/Alchemy RPC URLs
// - Private keys (hex)
// - Bearer tokens / JWTs
// - AWS credentials
// - Etherscan API keys
// - Generic secrets (PASSWORD, SECRET, etc.)

// Check if content contains secrets
if (containsSecrets(content)) {
  // Handle sensitive content
}

// Check sensitivity type
const sensitivity = checkSensitivity(content)
// Returns: { isSensitive, types, reasons }
// Types: credentials, private_key, financial_data, internal_paths, database_credentials

// Full sanitization
const sanitized = sanitizeContent(content, {
  redactSecrets: true,
  removeInternalPaths: true,
  maxLength: 10000,
})
// Returns: { content, wasModified, modifications, sensitivity }

// Process for artifact storage
const { content: safe, securityMetadata } = processArtifactForSecurity(content)
// securityMetadata: { isSensitive, sensitivityTypes, wasRedacted, redactedTypes, sanitizedAt }
```

### Metrics

Track efficiency and effectiveness.

```typescript
import { createMetricsCollector } from "@supermemory/tools/exploit-research"

const collector = createMetricsCollector("season-id", "protocol-id")

// Track token usage
collector.recordTokenUsage({
  inputTokens: 1000,
  outputTokens: 500,
  rawLength: 10000,
  reducedLength: 2000,
  costUsd: 0.05,
})

// Track retrieval performance
collector.recordRetrievalQuery({
  resultCount: 5,
  latencyMs: 150,
  success: true,
})

// Track dedup effectiveness
collector.recordDedupCheck({
  isDuplicate: false,
  noveltyScore: 0.85,
})

// Track phase progress
collector.startPhase("invariant")
collector.updatePhaseProgress("invariant", {
  coverage: 50,
  artifactsGenerated: 20,
  tokensUsed: 5000,
})
collector.completePhase("invariant", 100)

// Track test execution
collector.recordExecution({
  status: "pass",
  profit: 50000,
})

// Get metrics
const tokenMetrics = collector.getTokenMetrics()
const retrievalMetrics = collector.getRetrievalMetrics()
const dedupMetrics = collector.getDedupMetrics()
const executionMetrics = collector.getExecutionMetrics()
const phaseCoverage = collector.getPhaseCoverage()

// Generate summary report
const report = collector.getSummaryReport()
console.log(report)

// Serialize for persistence
const json = collector.toJSON()
// Later...
collector.fromJSON(json)
```

### Middleware

Intercept tool calls and build prompts from memory.

```typescript
import { createExploitResearchMiddleware } from "@supermemory/tools/exploit-research"

const middleware = createExploitResearchMiddleware({
  apiKey: "your-api-key",
  baseUrl: "https://api.supermemory.com",
  verbose: true,
  autoEmbed: true,
})

// Wrap tool executors
const wrappedTool = middleware.wrapToolExecutor(
  "tool_name",
  "artifact_kind",
  executor,
  {
    extractTarget: (input) => ({ contract: input.address }),
    shouldReduce: true,
  }
)

// Build prompt from memory (not chat history)
const prompt = await middleware.buildPromptFromMemory(context, {
  includeInvariants: 10,
  includePaths: 5,
  includeTests: 3,
  contractFocus: "Pool",
  query: "flash loan vulnerability",
})

// Record test feedback
await middleware.recordTestFeedback(
  "test-id",
  { status: "pass", profit: 50000 },
  { pathId: "path-id", invariantIds: ["inv-1"] },
  context
)

// Check for duplicates
const { isDuplicate, similarity } = await middleware.checkDuplicate(
  "hypothesis",
  "Flash loan attack description",
  context,
  0.85
)

// Store artifact directly
const response = await middleware.storeArtifact(
  "path",
  JSON.stringify(pathData),
  context,
  { summary: "Attack path summary" }
)

// Retrieve relevant context
const results = await middleware.retrieveContext(
  "oracle manipulation",
  context,
  { kind: "invariant", limit: 5 }
)
```

---

## Usage Examples

### Complete Exploit Research Workflow

```typescript
import {
  createExploitResearchMiddleware,
  reduceByKind,
  createMetricsCollector,
  processArtifactForSecurity,
} from "@supermemory/tools/exploit-research"

// Initialize
const middleware = createExploitResearchMiddleware({
  apiKey: process.env.SUPERMEMORY_API_KEY!,
})

const metrics = createMetricsCollector(
  "uniswap-v3:season-20241205-abc",
  "uniswap-v3"
)

const context = {
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  phase: "intake" as const,
}

// Phase 1: Intake
metrics.startPhase("intake")

// Wrap source fetching
const fetchSource = middleware.wrapToolExecutor(
  "etherscan_fetch",
  "source",
  async (input: { address: string }) => {
    const source = await etherscan.getSource(input.address)
    return source
  }
)

const source = await fetchSource({ address: "0x..." }, context)
metrics.updatePhaseProgress("intake", { artifactsGenerated: 1 })
metrics.completePhase("intake", 100)

// Phase 2: Invariant derivation
context.phase = "invariant"
metrics.startPhase("invariant")

// Build prompt from stored knowledge
const invariantPrompt = await middleware.buildPromptFromMemory(context, {
  includeInvariants: 0, // No existing invariants yet
  contractFocus: "Pool",
})

// Generate invariants (your LLM call here)
const invariants = await generateInvariants(invariantPrompt)

// Check each for novelty before storing
for (const inv of invariants) {
  const { isDuplicate } = await middleware.checkDuplicate(
    "invariant",
    inv.formula,
    context,
    0.85
  )

  if (!isDuplicate) {
    await middleware.storeArtifact(
      "invariant",
      JSON.stringify(inv),
      context
    )
    metrics.updatePhaseProgress("invariant", { artifactsGenerated: 1 })
  } else {
    metrics.recordDedupCheck({ isDuplicate: true, noveltyScore: 0.1 })
  }
}

metrics.completePhase("invariant", 100)

// Phase 3: Execution
context.phase = "execution"
metrics.startPhase("execution")

// Run test and record feedback
const testResult = await runForgeTest(testCode)

await middleware.recordTestFeedback(
  "test-001",
  {
    status: testResult.passed ? "pass" : "fail",
    profit: testResult.profit,
    forgeLogsContent: testResult.logs,
  },
  {
    pathId: "path-001",
    invariantIds: ["inv-001"],
  },
  context
)

metrics.recordExecution({
  status: testResult.passed ? "pass" : "fail",
  profit: testResult.profit,
})

metrics.completePhase("execution", 100)

// Generate final report
console.log(metrics.getSummaryReport())
```

### Reset Recovery

```typescript
// On context reset, rebuild from memory instead of chat history
async function recoverFromReset(context: ToolCallContext) {
  const prompt = await middleware.buildPromptFromMemory(context, {
    includeInvariants: 10,
    includePaths: 5,
    includeTests: 3,
    query: "current research focus",
  })

  // The prompt now contains:
  // - Season status and coverage
  // - Relevant invariants
  // - Relevant paths
  // - Recent test results

  return prompt
}
```

---

## Best Practices

### 1. Always Use Isolation

```typescript
// GOOD: Include both IDs
const request = {
  protocolId: "uniswap-v3",
  seasonId: "uniswap-v3:season-20241205-abc",
  // ...
}

// BAD: Missing season isolation
const request = {
  protocolId: "uniswap-v3",
  // Missing seasonId!
}
```

### 2. Reduce Before Storing

```typescript
// GOOD: Reduce content first
const reduced = reduceByKind("source", rawSource)
await storeArtifact({
  raw: rawSource,
  summary: reduced.summary,
  metadata: reduced.extractedMetadata,
})

// BAD: Store raw without reduction
await storeArtifact({ raw: rawSource })
```

### 3. Check Duplicates Before Creating

```typescript
// GOOD: Check first
const { isDuplicate } = await middleware.checkDuplicate("hypothesis", text, ctx)
if (!isDuplicate) {
  await middleware.storeArtifact("hypothesis", text, ctx)
}

// BAD: Store without checking
await middleware.storeArtifact("hypothesis", text, ctx)
```

### 4. Sanitize Sensitive Content

```typescript
// GOOD: Sanitize before storage
const { content, securityMetadata } = processArtifactForSecurity(rawContent)
await storeArtifact({
  raw: content,
  isSensitive: securityMetadata.isSensitive,
  wasRedacted: securityMetadata.wasRedacted,
})

// BAD: Store with secrets
await storeArtifact({ raw: contentWithSecrets })
```

### 5. Track Metrics

```typescript
// GOOD: Track everything
collector.recordTokenUsage({ ... })
collector.recordRetrievalQuery({ ... })
collector.recordDedupCheck({ ... })
collector.recordExecution({ ... })

// Review periodically
if (collector.getDedupMetrics().rejectionRate > 0.5) {
  console.log("High duplicate rate - adjust approach")
}
```

### 6. Link Related Entities

```typescript
// GOOD: Create explicit relationships
await link({
  parentId: "inv-001",
  childId: "path-001",
  relation: "violates",
})

await link({
  parentId: "path-001",
  childId: "test-001",
  relation: "tested_by",
})

// BAD: No relationships tracked
```

### 7. Use Snapshots Before Major Operations

```typescript
// GOOD: Snapshot before refinement
await snapshot({ seasonId, protocolId })
// ... refinement phase ...
// If something goes wrong:
await restore({ snapshotId, protocolId })

// BAD: No backup before destructive operations
```

---

## Test Coverage

The system includes comprehensive tests:

| Module | Tests | Coverage |
|--------|-------|----------|
| Schemas | 44 | All entity validations |
| API Schemas | 43 | All request/response formats |
| Reducers | 33 | All reducer types |
| Dedup | 32 | Similarity, novelty, filtering |
| Middleware | 19 | Tool wrapping, context building |
| Integration | 15 | Full workflows |
| Security | 27 | Secret redaction, sensitivity |
| Metrics | 31 | All tracking functions |
| Verification | 26 | End-to-end flows |

**Total: 270 tests, 258 passing, 12 skipped (live API)**

Run tests:
```bash
bun test packages/tools/src/exploit-research/__tests__/
bun test packages/validation/__tests__/exploit-research*
```

---

## License

MIT
