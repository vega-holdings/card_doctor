# Card Architect - Spec vs Current Build Analysis

**Date:** 2025-11-13
**Status:** DRIFT DETECTED - Requires immediate cleanup

---

## CRITICAL FINDINGS

### ❌ SCOPE VIOLATIONS (Features NOT in spec)

The following features exist in the current build but are **NOT** in the MVP spec:

1. **LLM Integration** (`apps/api/src/routes/llm.ts`, `apps/web/src/components/LLMAssistSidebar.tsx`)
   - LLM provider abstraction (Anthropic)
   - Field rewriting with AI
   - This is v1.2+ territory

2. **RAG System** (`apps/api/src/routes/rag.ts`)
   - Vector embeddings
   - Semantic search
   - Not in spec at all

3. **Redundancy Killer** (`apps/api/src/services/redundancy-killer.ts`, `apps/web/src/components/RedundancyPanel.tsx`)
   - Cross-field deduplication
   - Semantic overlap detection
   - Listed as "1.1 Batch tools" in spec, not MVP

4. **Prompt Simulator** (`apps/api/src/services/prompt-simulator.ts`, `apps/web/src/components/PromptSimulatorPanel.tsx`)
   - Multiple profile simulation
   - Token budget drop policies
   - Not in MVP spec

5. **Lore Trigger Tester** (`apps/api/src/services/lore-trigger-tester.ts`, `apps/web/src/components/LoreTriggerPanel.tsx`)
   - Live trigger testing
   - AND/NOT logic preview
   - Nice to have, but not MVP

### ✅ SPEC COMPLIANCE (What's actually correct)

1. **Repository Structure** ✓
   - `/apps/api` - Fastify + SQLite
   - `/apps/web` - React + Vite + Tailwind
   - `/packages/schemas` - Types + validation
   - `/packages/tokenizers` - Stub exists
   - `/packages/charx` - Stub exists
   - `/packages/plugins` - Stub exists

2. **Database Schema** ✓
   - `cards` table with v2/v3 spec support
   - `versions` table for snapshots
   - `assets` table for images
   - `original_image` BLOB column (for PNG round-trip)

3. **PNG Import/Export** ✓
   - Extract: `apps/api/src/utils/png.ts:extractFromPNG()` - **WORKS**
   - Embed: `apps/api/src/utils/png.ts:embedIntoPNG()` - **IMPLEMENTED**
   - Multiple text chunk key support (ccv3, chara, etc.)
   - Size validation

4. **Core API Routes** ✓
   - `POST /import` - JSON/PNG import
   - `GET /cards/:id/export?format=json|png` - Export
   - `POST /convert` - v2↔v3 conversion
   - `GET /cards`, `POST /cards`, `PATCH /cards/:id`, `DELETE /cards/:id`
   - `GET /cards/:id/versions`, `POST /cards/:id/versions`, `POST /cards/:id/versions/:ver/restore`
   - `POST /tokenize`, `GET /tokenizers`
   - `POST /assets`, `GET /assets/:id`, `POST /assets/:id/transform`

5. **Core Frontend Components** ✓
   - `CardEditor.tsx` - Main editor
   - `EditPanel.tsx` - Field editing
   - `LorebookEditor.tsx` - Character book editor (with all CCv3 fields)
   - `JsonPanel.tsx` - JSON view
   - `PreviewPanel.tsx` - Markdown preview
   - `DiffPanel.tsx` - Version diff (basic)
   - `Header.tsx` - Actions bar
   - `SettingsModal.tsx` - Config

6. **Validation** ✓
   - `packages/schemas/src/validator.ts` - Schema validation
   - Ajv-based
   - v2 and v3 support

### ⚠️ PARTIAL / INCOMPLETE

1. **Tokenizers Package** - Stub
   - `packages/tokenizers/src/index.ts` exists but minimal
   - Needs HF tokenizer adapter (BPE/SentencePiece)
   - API route exists but tokenizer impl may be incomplete

2. **Diff View** - Basic
   - Uses jsondiffpatch
   - **Missing:** Token deltas per field
   - **Missing:** Graph visualization

3. **Plugins Package** - Stub
   - Interface defined
   - No implementations

4. **CHARX Package** - Stub
   - Interface defined
   - No pack/unpack implementation

5. **Docker** - Configured but not tested
   - `docker-compose.yml` exists
   - `Dockerfile` exists
   - **Unknown:** If it builds/runs

### 🔥 BUILD ISSUES

1. **TypeScript Errors**
   - Some unused variables (`buffer` in charx)
   - Missing `@card-architect/schemas` imports in plugins package
   - Build fails in workspaces

2. **Dependency Issues**
   - 5 moderate security vulnerabilities
   - Some deprecated packages (glob, inflight)

---

## SPEC REQUIREMENTS vs ACTUAL

| Requirement | Spec | Actual | Status |
|-------------|------|--------|--------|
| **Formats** | CCv2/CCv3 read/edit/validate | ✓ | ✅ |
| **Field Coverage** | All CCv3 fields + lorebook | ✓ | ✅ |
| **Editing UX** | CHUB-style panels, token chips, Markdown | ✓ | ✅ |
| **Tokenization** | HF tokenizers (client-side, BPE/SentencePiece) | Partial | ⚠️ |
| **Save/Version** | Autosave IndexedDB + SQLite + snapshots | ✓ | ✅ |
| **Import/Export** | JSON/PNG, CCv2/v3 detect/normalize | ✓ | ✅ |
| **Validation** | Schema + semantic lint | ✓ | ✅ |
| **Asset Tools** | Upload/crop/resize/convert | ✓ | ✅ |
| **Size Guardrails** | 2MB/4MB warnings, configurable caps | ✓ | ✅ |
| **Self-Host** | Docker Compose or single binary | ✓ | ⚠️ (untested) |
| **MIT License** | Required | ✓ | ✅ |
| **Plugin Contracts** | Defined, hot-swappable | Stubs only | ⚠️ |
| **LLM Integration** | **NOT IN MVP** | **IMPLEMENTED** | ❌ SCOPE CREEP |
| **RAG System** | **NOT IN SPEC** | **IMPLEMENTED** | ❌ SCOPE CREEP |
| **Redundancy Tools** | **v1.1 Milestone** | **IN MVP** | ❌ EARLY |
| **Prompt Simulator** | **NOT IN SPEC** | **IMPLEMENTED** | ❌ SCOPE CREEP |

---

## WHAT SHOULD BE REMOVED (Before MVP Ship)

1. **Delete LLM Integration**
   - `apps/api/src/routes/llm.ts`
   - `apps/api/src/providers/anthropic.ts`
   - `apps/web/src/components/LLMAssistSidebar.tsx`
   - Remove from `apps/api/src/index.ts` route registration

2. **Delete RAG System**
   - `apps/api/src/routes/rag.ts`
   - Remove from `apps/api/src/index.ts` route registration

3. **Move to v1.1 Branch (Defer)**
   - `apps/api/src/services/redundancy-killer.ts`
   - `apps/api/src/routes/redundancy.ts`
   - `apps/web/src/components/RedundancyPanel.tsx`
   - `apps/api/src/services/prompt-simulator.ts`
   - `apps/api/src/routes/prompt-simulator.ts`
   - `apps/web/src/components/PromptSimulatorPanel.tsx`
   - `apps/api/src/services/lore-trigger-tester.ts`
   - `apps/api/src/routes/lore-trigger.ts`
   - `apps/web/src/components/LoreTriggerPanel.tsx`

4. **Clean Up Documentation**
   - `IMPLEMENTATION_STATUS.md` - Delete or rename to `FUTURE_FEATURES.md`
   - `LLM_ASSIST_V2_DOCUMENTATION.md` - Move to docs/future/
   - `README.md` - Update to remove mention of features not in MVP

---

## WHAT MUST BE COMPLETED (MVP Blockers)

### 1. Tokenizers Package ⚠️
**Status:** Stub exists, may not be fully functional

**Required:**
- Implement HF tokenizer adapters (at minimum GPT-2 BPE)
- `TokenizerAdapter` interface with `estimate(text: string): number`
- Built-in presets: `gpt2-bpe`, `llama-bpe`, `generic-bpe`
- Per-field token counting

**Files to check/fix:**
- `packages/tokenizers/src/index.ts`
- Test with `/tokenize` endpoint

### 2. Docker Build/Test 🔥
**Status:** Configured but untested

**Required:**
- `docker-compose up` must work
- API on port 3456
- Web on port 8765
- Volumes for data + storage
- Health checks passing

**Files:**
- `Dockerfile` - Check if multi-stage build works
- `docker-compose.yml` - Validate

### 3. Build Fixes 🔥
**Status:** TypeScript errors

**Required:**
- Fix unused variable warnings
- Fix missing type imports
- All workspaces must build successfully
- `npm run build` exits 0

### 4. Acceptance Tests (Per Spec) 📋

From the spec, these must pass:

- [ ] Typing in any field updates token chip + global total ≤50ms
- [ ] Import CCv3 JSON with `character_book` (selective AND/NOT, probability) → re-export JSON+PNG → re-import → byte-stable canonical form
- [ ] PNG exported by app is readable by third-party tools; importer extracts same JSON
- [ ] Autosave survives tab crash (≤1 keystroke loss); restore from versions works
- [ ] Diff view shows changed fields + token deltas (**MISSING TOKEN DELTAS**)
- [ ] Markdown preview renders `<img>` + links; DOMPurify blocks scripts
- [ ] Size guardrails trigger at configured thresholds; export can be forced after warning

---

## ARCHITECTURE ALIGNMENT

| Component | Spec | Actual | Notes |
|-----------|------|--------|-------|
| **Backend Framework** | Fastify + Node 20 | ✓ | Correct |
| **Database** | SQLite (better-sqlite3) | ✓ | Correct |
| **Image Processing** | Sharp + pngjs | ✓ | Correct |
| **Schema Validation** | Ajv (JSON Schema) | ✓ | Correct |
| **Frontend Framework** | React 18 + TS + Vite | ✓ | Correct |
| **State Management** | Zustand | ✓ | Correct |
| **Styling** | Tailwind CSS | ✓ | Correct |
| **Markdown Editor** | CodeMirror (spec) | marked (actual) | Different but OK |
| **JSON Editor** | Monaco (spec) | Not visible | May be missing |
| **Local Storage** | IndexedDB | idb | Correct |
| **Sanitization** | DOMPurify | ✓ | Correct |

---

## REPO FILE VIOLATIONS

| File | Reason | Action |
|------|--------|--------|
| `IMPLEMENTATION_STATUS.md` | Documents features not in MVP spec | Delete or move to `docs/roadmap/` |
| `LLM_ASSIST_V2_DOCUMENTATION.md` | LLM features not in MVP | Move to `docs/future/` |
| `main_comely-vigorous-*.png` | Test files in root | Move to `test-data/` or delete |
| `main_usmc-psyop-*.json` | Test files in root | Move to `test-data/` or delete |
| `savvy.json` | Test file in root | Move to `test-data/` or delete |

---

## CRITICAL PATH TO MVP

### Phase 1: Remove Scope Violations (1-2 hours)
1. Delete LLM routes + components
2. Delete RAG routes
3. Create `feature/deferred` branch
4. Move redundancy/prompt-sim/lore-trigger to branch
5. Update route registration in `apps/api/src/index.ts`
6. Update frontend to remove panels

### Phase 2: Fix Build (1 hour)
1. Fix TypeScript errors in charx/plugins
2. Run `npm run build` until clean
3. Run `npm audit fix`

### Phase 3: Complete Tokenizers (2-3 hours)
1. Implement basic HF tokenizer adapters
2. Test with `/tokenize` endpoint
3. Verify per-field + global counts work in UI

### Phase 4: Docker Test (1 hour)
1. `docker-compose build`
2. `docker-compose up`
3. Test API health check
4. Test web UI loads
5. Test basic import/export flow

### Phase 5: Acceptance Tests (2-3 hours)
1. Write test script for each acceptance criterion
2. Run all tests
3. Fix any failures
4. Document test results

### Phase 6: Documentation Cleanup (1 hour)
1. Update README to match actual MVP
2. Remove mentions of deferred features
3. Add "Roadmap" section pointing to v1.1/v1.2
4. Clean up root directory (move test files)

---

## SUMMARY

**GOOD NEWS:**
- Core MVP functionality is implemented and mostly working
- Repository structure matches spec
- Database schema is correct
- PNG import/export is functional
- Validation system exists

**BAD NEWS:**
- **Scope creep:** LLM, RAG, and advanced analysis features were added (NOT in MVP spec)
- Build is broken (TypeScript errors)
- Tokenizers may be incomplete
- Docker untested
- Test files pollute root directory
- Documentation describes wrong product

**ESTIMATED TIME TO CLEAN MVP:** 8-12 hours of focused work

**RECOMMENDATION:**
1. **Immediately:** Remove all scope violations (LLM, RAG)
2. **Before shipping:** Fix build, test Docker, complete tokenizers
3. **After MVP ships:** Cherry-pick deferred features into v1.1 branch
