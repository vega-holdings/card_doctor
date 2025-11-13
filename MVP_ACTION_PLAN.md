# Card Architect MVP - Action Plan

**Objective:** Ship clean, spec-compliant MVP without scope violations

**Estimated Total Time:** 8-12 hours
**Priority:** CRITICAL - Block all other work until MVP is clean

---

## PHASE 1: SCOPE VIOLATION CLEANUP [CRITICAL]
**Time:** 1-2 hours
**Priority:** DO THIS FIRST

### Step 1.1: Create Feature Branch for Deferred Work
```bash
git checkout -b feature/v1.1-deferred
git push -u origin feature/v1.1-deferred
git checkout claude/card-architect-mvp-plan-011CV5rPpz2YQKetYzzQTQpA
```

### Step 1.2: Delete LLM Integration [NOT IN MVP]
**Files to delete:**
- `apps/api/src/routes/llm.ts`
- `apps/api/src/providers/anthropic.ts`
- `apps/web/src/components/LLMAssistSidebar.tsx`

**Files to edit:**
- `apps/api/src/index.ts` - Remove llm route registration
- `apps/web/src/App.tsx` - Remove LLMAssistSidebar import/usage

**Commands:**
```bash
rm apps/api/src/routes/llm.ts
rm -rf apps/api/src/providers/
rm apps/web/src/components/LLMAssistSidebar.tsx
```

### Step 1.3: Delete RAG System [NOT IN SPEC]
**Files to delete:**
- `apps/api/src/routes/rag.ts`

**Files to edit:**
- `apps/api/src/index.ts` - Remove rag route registration

**Commands:**
```bash
rm apps/api/src/routes/rag.ts
```

### Step 1.4: Move Advanced Features to Deferred Branch [v1.1+]
**Files to move (keep in codebase but defer to v1.1):**

Create new directory structure for future features:
```bash
mkdir -p apps/api/src/services/deferred
mkdir -p apps/api/src/routes/deferred
mkdir -p apps/web/src/components/deferred
```

**Move these files:**
- `apps/api/src/services/redundancy-killer.ts` → `apps/api/src/services/deferred/`
- `apps/api/src/services/prompt-simulator.ts` → `apps/api/src/services/deferred/`
- `apps/api/src/services/lore-trigger-tester.ts` → `apps/api/src/services/deferred/`
- `apps/api/src/routes/redundancy.ts` → `apps/api/src/routes/deferred/`
- `apps/api/src/routes/prompt-simulator.ts` → `apps/api/src/routes/deferred/`
- `apps/api/src/routes/lore-trigger.ts` → `apps/api/src/routes/deferred/`
- `apps/web/src/components/RedundancyPanel.tsx` → `apps/web/src/components/deferred/`
- `apps/web/src/components/PromptSimulatorPanel.tsx` → `apps/web/src/components/deferred/`
- `apps/web/src/components/LoreTriggerPanel.tsx` → `apps/web/src/components/deferred/`

**OR** (simpler approach): Comment out route registrations and component usage

**Files to edit:**
- `apps/api/src/index.ts` - Comment out: redundancyRoutes, promptSimulatorRoutes, loreTriggerRoutes
- `apps/web/src/App.tsx` - Remove any tabs/panels for these features

### Step 1.5: Clean Up Root Directory
```bash
mkdir -p test-data
mv main_*.png test-data/
mv main_*.json test-data/
mv savvy.json test-data/
```

### Step 1.6: Update Documentation
**Files to edit:**

1. `README.md`
   - Remove mentions of: LLM assist, RAG, redundancy killer, prompt simulator, lore trigger tester
   - Update "Roadmap" to show these as v1.1+ features
   - Ensure MVP feature list matches spec exactly

2. `IMPLEMENTATION_STATUS.md`
   - Rename to `docs/ROADMAP_V1.1.md`
   - Add header: "This document describes post-MVP features planned for v1.1 and beyond"

3. `LLM_ASSIST_V2_DOCUMENTATION.md`
   - Move to `docs/future/LLM_ASSIST_V2_DOCUMENTATION.md`

**Commands:**
```bash
mkdir -p docs/future
mv IMPLEMENTATION_STATUS.md docs/ROADMAP_V1.1.md
mv LLM_ASSIST_V2_DOCUMENTATION.md docs/future/
```

### Acceptance Criteria - Phase 1:
- [ ] No LLM/RAG code in API routes
- [ ] No LLM/RAG components in web app
- [ ] Advanced features moved to deferred/ or commented out
- [ ] Root directory clean (only essential files)
- [ ] README describes only MVP features
- [ ] `npm run lint` passes (or only shows expected warnings)

---

## PHASE 2: BUILD FIXES [BLOCKER]
**Time:** 1 hour
**Priority:** HIGH

### Step 2.1: Fix TypeScript Errors

**Issue 1: Unused variable in charx package**
- File: `packages/charx/src/index.ts`
- Fix: Remove or use the `buffer` variable

**Issue 2: Missing @card-architect/schemas import in plugins**
- File: `packages/plugins/src/index.ts`
- Fix: Either remove the import or ensure schemas package builds first

**Issue 3: Remove unused imports**
- File: `apps/api/src/index.ts` (line 15: unused `join`)
- Fix: Remove the import

### Step 2.2: Run Full Build
```bash
npm run build
```

Expected result: Exit code 0, no errors

### Step 2.3: Fix Security Issues
```bash
npm audit fix
```

**If breaking changes required:**
- Review carefully
- Test after fixing

### Acceptance Criteria - Phase 2:
- [ ] `npm run build` exits with code 0
- [ ] No TypeScript compilation errors
- [ ] No blocking security vulnerabilities
- [ ] All workspaces build successfully

---

## PHASE 3: TOKENIZER COMPLETION [MVP REQUIREMENT]
**Time:** 2-3 hours
**Priority:** HIGH

### Step 3.1: Check Current Tokenizer Implementation

**File to inspect:**
- `packages/tokenizers/src/index.ts`

**Required functionality:**
1. `TokenizerAdapter` interface with `estimate(text: string): number`
2. At least one working implementation (GPT-2 BPE or similar)
3. `TokenizerRegistry` or similar to manage multiple tokenizers

### Step 3.2: Implement Basic BPE Tokenizer

**Minimal MVP approach (if not already done):**

Use a simple word-splitting approximation:
- GPT-2 average: ~4 chars per token
- LLaMA average: ~4 chars per token

**OR** (better): Use a lightweight library like `gpt-tokenizer` or `js-tiktoken`

**Files to create/edit:**
- `packages/tokenizers/src/adapters/gpt2.ts` - GPT-2 BPE implementation
- `packages/tokenizers/src/adapters/llama.ts` - LLaMA BPE implementation
- `packages/tokenizers/src/registry.ts` - Registry for tokenizers
- `packages/tokenizers/src/index.ts` - Public API

### Step 3.3: Test Tokenizer

**Create test:**
```typescript
// Quick test
import { TokenizerRegistry } from '@card-architect/tokenizers';

const tokenizer = TokenizerRegistry.get('gpt2-bpe');
const text = "Hello, world! This is a test.";
const tokens = tokenizer.estimate(text);
console.log(`Estimated tokens: ${tokens}`); // Should be ~7-9 tokens
```

**Test with API:**
```bash
# Start dev server
npm run dev:api

# In another terminal
curl -X POST http://localhost:3456/tokenize \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt2-bpe",
    "payload": {
      "name": "Test Character",
      "description": "A test character for token counting."
    }
  }'
```

Expected response:
```json
{
  "model": "gpt2-bpe",
  "fields": {
    "name": 3,
    "description": 8
  },
  "total": 11
}
```

### Step 3.4: Verify Frontend Integration

**Test in browser:**
1. Start dev servers: `npm run dev`
2. Open http://localhost:5173
3. Create or load a card
4. Verify token chips appear next to each field
5. Verify global token count in header
6. Type in a field → token count updates ≤50ms

### Acceptance Criteria - Phase 3:
- [ ] Tokenizer package exports working adapters
- [ ] `/tokenize` endpoint returns accurate counts
- [ ] Frontend shows per-field token chips
- [ ] Global token count displayed
- [ ] Token updates happen ≤50ms after typing

---

## PHASE 4: DOCKER BUILD/TEST [MVP REQUIREMENT]
**Time:** 1 hour
**Priority:** HIGH

### Step 4.1: Build Docker Images
```bash
docker-compose build
```

Expected result: Both `api` and `web` images build successfully

**Common issues:**
- Missing build context files
- Incorrect Dockerfile paths
- Node version mismatches

### Step 4.2: Start Services
```bash
docker-compose up -d
```

### Step 4.3: Test Health Checks
```bash
# API health
curl http://localhost:3456/health

# Web health
curl http://localhost:8765/

# View logs
docker-compose logs api
docker-compose logs web
```

### Step 4.4: Test Basic Flow

**In browser:**
1. Navigate to http://localhost:8765
2. Create a new card
3. Fill in fields
4. Save card
5. Export as JSON
6. Export as PNG
7. Re-import the PNG
8. Verify data matches

### Step 4.5: Test Persistence
```bash
# Stop services
docker-compose down

# Restart
docker-compose up -d

# Check if cards still exist
curl http://localhost:3456/cards
```

### Acceptance Criteria - Phase 4:
- [ ] `docker-compose build` succeeds
- [ ] `docker-compose up` starts both services
- [ ] Health checks pass
- [ ] Web UI loads at port 8765
- [ ] API responds at port 3456
- [ ] Data persists across restarts
- [ ] Volumes mounted correctly

---

## PHASE 5: MVP ACCEPTANCE TESTS [CRITICAL]
**Time:** 2-3 hours
**Priority:** HIGH

### Test 1: Token Count Performance
**Requirement:** Typing updates token chip + global total ≤50ms

**Test:**
1. Load a card
2. Open browser DevTools → Performance tab
3. Start recording
4. Type a long paragraph in description field
5. Stop recording
6. Measure time from keypress to token count update

**Pass criteria:** ≤50ms latency

---

### Test 2: PNG Round-Trip (Canonical)
**Requirement:** Import JSON with character_book (selective AND/NOT, probability) → export JSON+PNG → re-import → byte-stable

**Test:**
1. Create CCv3 card with complex character_book:
   ```json
   {
     "spec": "chara_card_v3",
     "spec_version": "3.0",
     "data": {
       "name": "Test",
       "description": "Test character",
       "personality": "Friendly",
       "scenario": "Test scenario",
       "first_mes": "Hello!",
       "mes_example": "<START>",
       "creator": "Tester",
       "character_version": "1.0",
       "system_prompt": "",
       "post_history_instructions": "",
       "alternate_greetings": [],
       "tags": [],
       "character_book": {
         "entries": [
           {
             "id": 0,
             "keys": ["test"],
             "secondary_keys": ["verify"],
             "content": "This is a test entry",
             "enabled": true,
             "insertion_order": 100,
             "case_sensitive": false,
             "priority": 10,
             "selective": true,
             "selectiveLogic": "AND",
             "constant": false,
             "probability": 80,
             "position": "before_char"
           }
         ]
       }
     }
   }
   ```

2. Import this JSON
3. Export as JSON → save as `export1.json`
4. Export as PNG → save as `export1.png`
5. Re-import `export1.png`
6. Export as JSON → save as `export2.json`
7. Run: `diff export1.json export2.json`

**Pass criteria:** Files are identical (byte-for-byte) OR differ only in whitespace/key order (if canonical serialization is applied)

---

### Test 3: Third-Party PNG Compatibility
**Requirement:** PNG exported by app is readable by third-party tools; importer extracts same JSON

**Test:**
1. Export a card as PNG
2. Use external PNG reader to verify tEXt chunks exist:
   ```bash
   # Using exiftool or similar
   exiftool export1.png | grep -i text
   ```
3. Use Python/Node script to manually extract tEXt chunk
4. Compare extracted JSON to original

**Pass criteria:** Third-party tools can read the PNG and extract the JSON

---

### Test 4: Autosave + Crash Recovery
**Requirement:** Autosave survives tab crash (≤1 keystroke loss); restore from versions works

**Test:**
1. Open app
2. Create new card
3. Type a paragraph in description
4. **Before clicking Save**, force-close the tab (Cmd+W / Alt+F4)
5. Re-open app
6. Check if card data was recovered from IndexedDB

**Pass criteria:** ≤1 keystroke of data loss

---

### Test 5: Version History + Restore
**Requirement:** Manual snapshots work, diff shows changes, restore works

**Test:**
1. Create card with initial data
2. Save card
3. Click "Create Snapshot" → add message "Initial version"
4. Edit description field significantly
5. Click "Create Snapshot" → add message "Updated description"
6. Go to Diff tab
7. View version history
8. Click "Restore" on first version

**Pass criteria:**
- Versions listed with timestamps
- Diff shows field changes
- Restore reverts card to earlier state

---

### Test 6: Markdown Preview + Sanitization
**Requirement:** Preview renders `<img>` + links; DOMPurify blocks scripts

**Test:**
1. Create card
2. In first_mes, add:
   ```markdown
   Hello! [Click here](https://example.com)
   ![Test Image](https://via.placeholder.com/150)
   <script>alert('XSS')</script>
   ```
3. Go to Preview tab

**Pass criteria:**
- Link is clickable
- Image renders
- Script tag does NOT execute
- No alert appears

---

### Test 7: Size Guardrails
**Requirement:** Warnings trigger at configured thresholds; export can be forced after warning

**Test:**
1. Create card with >2MB JSON data (e.g., giant lorebook)
2. Try to save
3. Verify warning appears
4. Create card with large image (>2MB PNG)
5. Export as PNG
6. Verify warning appears but export completes

**Pass criteria:** Warnings shown at 2MB, hard caps work (or can be overridden)

---

### Test 8: v2 ↔ v3 Conversion
**Requirement:** Convert v2 to v3 and vice versa without data loss

**Test:**
1. Import a v2 card
2. Convert to v3 via `/convert` endpoint
3. Verify all fields mapped correctly
4. Convert back to v2
5. Compare to original

**Pass criteria:** No data loss in round-trip conversion

---

### Acceptance Criteria - Phase 5:
- [ ] All 8 tests pass
- [ ] Document test results
- [ ] Fix any failures before proceeding

---

## PHASE 6: DOCUMENTATION CLEANUP [POLISH]
**Time:** 1 hour
**Priority:** MEDIUM

### Step 6.1: Update README.md

**Remove:**
- All mentions of LLM integration
- All mentions of RAG
- All mentions of redundancy killer
- All mentions of prompt simulator
- All mentions of lore trigger tester

**Ensure MVP section includes ONLY:**
- CCv2/CCv3 support
- Token counting (HF tokenizers)
- Lorebook editor
- Autosave + versions
- Import/Export (JSON + PNG)
- Asset management
- Validation
- Dark mode
- Self-hostable

**Add Roadmap section:**
```markdown
## Roadmap

### v1.1 (Planned)
- CHARX pack/unpack
- Voxta export
- CCv2 → CCv3 migration wizard
- Batch normalization tools

### v1.2 (Future)
- Plugin SDK for custom validators
- LLM-assisted field generation
- Advanced analysis tools
- Prompt simulator
```

### Step 6.2: Create CONTRIBUTING.md (if not exists)

**Include:**
- How to build locally
- How to run tests
- Code style guidelines
- PR process

### Step 6.3: Update package.json descriptions

**Ensure each package has:**
- Correct name
- Correct version (0.1.0 for MVP)
- Correct description matching spec

### Acceptance Criteria - Phase 6:
- [ ] README accurately describes MVP (no future features)
- [ ] Roadmap clearly separates MVP from v1.1+
- [ ] CONTRIBUTING.md exists
- [ ] All package.json files accurate

---

## FINAL CHECKLIST (Before MVP Ship)

### Code Quality
- [ ] `npm run build` succeeds
- [ ] `npm run lint` passes (or acceptable warnings only)
- [ ] `npm run type-check` passes
- [ ] No console errors in dev mode

### Functionality
- [ ] Import JSON (v2 and v3) works
- [ ] Import PNG (v2 and v3) works
- [ ] Export JSON works
- [ ] Export PNG works
- [ ] PNG round-trip is canonical
- [ ] Token counting works (per-field + global)
- [ ] Autosave to IndexedDB works
- [ ] Save to API/SQLite works
- [ ] Version snapshots work
- [ ] Version restore works
- [ ] Diff view shows changes
- [ ] Lorebook editor works (all CCv3 fields)
- [ ] Asset upload/crop/resize works
- [ ] Validation errors show correctly

### Docker
- [ ] `docker-compose build` succeeds
- [ ] `docker-compose up` starts services
- [ ] Health checks pass
- [ ] Web UI loads
- [ ] API responds
- [ ] Data persists

### Documentation
- [ ] README matches actual MVP
- [ ] No mentions of unimplemented features
- [ ] Roadmap shows future plans
- [ ] License file present (MIT)
- [ ] CONTRIBUTING.md present

### Scope Compliance
- [ ] No LLM integration
- [ ] No RAG system
- [ ] No redundancy killer (deferred to v1.1)
- [ ] No prompt simulator (deferred to v1.1)
- [ ] No lore trigger tester (deferred to v1.1)

---

## GIT WORKFLOW

### Commit Strategy
```bash
# After Phase 1
git add -A
git commit -m "refactor: remove scope violations (LLM, RAG, advanced features)

- Delete LLM integration (routes, providers, components)
- Delete RAG system
- Move redundancy/prompt-sim/lore-trigger to deferred/
- Clean up root directory (move test files)
- Update documentation to reflect MVP scope"

# After Phase 2
git add -A
git commit -m "fix: resolve TypeScript build errors

- Fix unused variables in charx package
- Remove unused imports in API
- Fix missing type dependencies
- Run npm audit fix"

# After Phase 3
git add -A
git commit -m "feat: complete tokenizer implementation

- Implement GPT-2 BPE tokenizer adapter
- Add tokenizer registry
- Verify per-field token counting
- Test frontend integration"

# After Phase 4
git add -A
git commit -m "chore: verify Docker build and deployment

- Test docker-compose build
- Verify health checks
- Test basic card import/export flow
- Document deployment process"

# After Phase 5
git add -A
git commit -m "test: complete MVP acceptance tests

- Add test suite for all MVP requirements
- Document test results
- Fix any failing tests"

# After Phase 6
git add -A
git commit -m "docs: update documentation for MVP release

- Update README to match actual features
- Add roadmap for v1.1+
- Create CONTRIBUTING guide
- Clean up obsolete documentation"

# Final push
git push -u origin claude/card-architect-mvp-plan-011CV5rPpz2YQKetYzzQTQpA
```

---

## POST-MVP: v1.1 Feature Branch

**After MVP ships**, create feature branch for advanced tools:
```bash
git checkout -b feature/v1.1-advanced-tools
```

**Cherry-pick deferred features:**
- Redundancy killer
- Prompt simulator
- Lore trigger tester

**Implement v1.1 features:**
- CHARX support
- Voxta export
- Batch tools

---

## ESTIMATED TIMELINE

| Phase | Time | Can Start After |
|-------|------|-----------------|
| Phase 1: Scope Cleanup | 1-2 hrs | Now |
| Phase 2: Build Fixes | 1 hr | Phase 1 |
| Phase 3: Tokenizers | 2-3 hrs | Phase 2 |
| Phase 4: Docker Test | 1 hr | Phase 2 |
| Phase 5: Acceptance Tests | 2-3 hrs | Phase 3 + 4 |
| Phase 6: Documentation | 1 hr | Phase 5 |

**Total: 8-12 hours**

**Parallelization possible:**
- Phase 3 and Phase 4 can run in parallel after Phase 2
- Phase 6 can be done while waiting for test results in Phase 5

**Critical path:** Phase 1 → Phase 2 → Phase 3 → Phase 5 → Ship

---

## BLOCKERS & RISKS

| Risk | Impact | Mitigation |
|------|--------|------------|
| Tokenizer implementation complex | HIGH | Use simple approximation for MVP, note limitation |
| Docker build fails | MEDIUM | Fix incrementally, document issues |
| Test failures reveal bugs | HIGH | Allocate buffer time for fixes |
| PNG round-trip not canonical | HIGH | Implement stable JSON serialization |
| Frontend integration broken after cleanup | HIGH | Test thoroughly after Phase 1 |

---

## SUCCESS CRITERIA

**MVP is ready to ship when:**
1. All acceptance tests pass
2. Docker deployment works
3. README accurately describes the product
4. No scope violations remain
5. Build is clean (no errors)
6. Token counting works correctly
7. PNG import/export is reliable

**Definition of Done:**
- [ ] All phases complete
- [ ] All checklists checked
- [ ] All commits pushed
- [ ] Ready for PR/review/deploy
