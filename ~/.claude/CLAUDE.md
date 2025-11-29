# Claude Code Guidelines

## 1. CORE PRINCIPLES

### 1.1 Design Philosophy
| Principle | Rule | Anti-Pattern |
|-----------|------|--------------|
| **ETC** | Prioritize changeability | Rigid, tightly-coupled code |
| **SRP** | One reason to change per module | God classes/functions |
| **DRY** | Single authoritative source | Copy-paste code |
| **YAGNI** | Build only what's needed now | Speculative features |
| **SSOT** | One source per knowledge type | Duplicated validation/constants |

### 1.2 SSOT Mapping
| Domain | Single Source | Never Duplicate In |
|--------|---------------|-------------------|
| Schema | DB migrations | Application layers |
| Validation | Schema definitions | Multiple components |
| Constants | Centralized file | Magic strings |
| Config | Environment variables | Hardcoded values |
| Business Logic | Domain/service layer | UI/presentation |

### 1.3 Structural Defects (ALWAYS Prevent)
- ❌ N+1 queries → Use eager loading/joins
- ❌ Sequential independent API calls → Use `Promise.all()`
- ❌ O(n²) when O(n) exists
- ❌ Missing indexes on queried columns
- ❌ Large bundles → Code split heavy features

---

## 2. CODE QUALITY

### 2.1 Naming
| Type | Convention | Example |
|------|------------|---------|
| Functions | verb + noun | `getUserData`, `calculateTotal` |
| Booleans | is/has/should prefix | `isActive`, `hasPermission` |
| Constants | SCREAMING_SNAKE | `MAX_RETRY_COUNT` |

### 2.2 Functions
- **Size**: ~20 lines max
- **Parameters**: 3 max (use object for more)
- **Pattern**: Early return with guard clauses

```typescript
// ✅ Good
function process(user: User | null): Result {
    if (!user) return { error: 'Invalid' };
    if (!user.isActive) return { error: 'Inactive' };
    return { success: true };
}
```

### 2.3 Error Handling
- **Fail fast**: Detect errors early
- **Never silent**: Always log or propagate
- **Typed errors**: Define error types
- **User-friendly**: Separate user messages from logs

---

## 3. DOCUMENTATION (MECE by Artifact)

| Artifact | Documents | Focus |
|----------|-----------|-------|
| **Code** | HOW | Implementation mechanics |
| **Tests** | WHAT | Expected behavior |
| **Commits** | WHY | Reasoning for change |
| **Comments** | WHY NOT | Rejected alternatives |

### 3.1 WHY NOT Comments (MANDATORY)

**Trigger Table - MUST add `// WHY NOT` when:**

| Situation | Explain |
|-----------|---------|
| Sequential API calls | Why not `Promise.all()`? |
| No caching | Why not cache? |
| Custom impl over library | Why not existing library? |
| Sync over async | Why not async? |
| Polling over WebSocket | Why not real-time? |
| Raw SQL over ORM | Why not ORM? |
| Hardcoded value | Why not config/env? |
| Denormalized data | Why not normalized? |
| No input validation | Why trust this input? |
| No retry logic | Why no retry? |
| No error handling | Why propagate? |

**Format:**
```typescript
// WHY NOT [alternative]: [reason].
// WHY NOT Promise.all(): Second API requires user.id from first response.
// WHY NOT Redis: Response <50ms, cache invalidation adds complexity.
```

**Examples:**
```typescript
// ✅ Good
// WHY NOT cache: Real-time balance, changes every request.
const data = await fetchFreshData('/endpoint');

// WHY NOT Promise.all(): fetchSettings requires user.id from fetchUser.
const user = await fetchUser();
const settings = await fetchSettings(user.id);

// ❌ Bad: No explanation for sequential calls
const user = await fetchUser();
const settings = await fetchSettings(user.id);
```

### 3.2 Self-Check Before Commit
1. Would a senior ask "why not [X]?" → Add WHY NOT
2. Did I reject an alternative? → Add WHY NOT
3. Not using the "standard" way? → Add WHY NOT

---

## 4. TESTING

### 4.1 Principles
- Test **behavior**, not implementation
- One assertion per test
- Independent tests (no shared state)
- Target <100ms per unit test

### 4.2 Pattern: Arrange-Act-Assert
```typescript
it('returns user when API succeeds', async () => {
    // Arrange
    const mockData = { id: '1', name: 'Test' };
    mockFetcher.mockResolvedValue(mockData);
    // Act
    const result = await userService.getUser('1');
    // Assert
    expect(result).toEqual(mockData);
});
```

---

## 5. SECURITY

### 5.1 Rules
| Area | Rule |
|------|------|
| Secrets | Never commit; use env vars |
| Input | Validate all external input; whitelist |
| Auth | Server-side validation; least privilege |
| Dependencies | Regular audits; commit lock files |

---

## 6. VERSION CONTROL

### 6.1 Commit Rules (NON-NEGOTIABLE)
| Rule | Guideline |
|------|-----------|
| Size | ~100 lines (max 300) |
| Scope | One logical change |
| Test | "and" in message = split it |
| Format | `type: description` |

**Types:** `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`

### 6.2 Commit Sequence (Layer by Layer)
```bash
# 1. Domain
git commit -m "feat: Add User model and repository interface"
# 2. Infrastructure
git commit -m "feat: Implement User PostgreSQL repository"
# 3. Migration
git commit -m "feat: Add users table migration"
# 4. Application
git commit -m "feat: Add User usecase and DTOs"
# 5. Presentation
git commit -m "feat: Add User REST endpoints"
```

### 6.3 Anti-Patterns
```bash
# ❌ Multiple changes
feat: Add user feature and fix seed data
# ❌ Vague
feat: Update code
# ❌ Too large (1000+ lines)
feat: Add complete authentication system
```

### 6.4 Branch Strategy
- Naming: `feat/name`, `fix/name`, `refactor/name`
- Merge within 2-3 days
- Sync with main daily

---

## 7. CODE REVIEW

### 7.1 PR Guidelines
- Target: ≤300 lines
- Self-review before requesting

### 7.2 Review Checklist
- [ ] Solves stated problem
- [ ] Tests pass + adequate coverage
- [ ] No structural defects
- [ ] No security vulnerabilities
- [ ] WHY NOT comments present for triggers
- [ ] No debug logs/commented code

---

## 8. AI IMPLEMENTATION RULES

### 8.1 Before Implementing
1. ✅ Search codebase for existing solutions
2. ✅ Verify no SSOT violation
3. ✅ Consider simpler alternatives
4. ✅ Identify security/performance risks

### 8.2 Implementation Workflow
1. Clarify requirements if unclear
2. Plan structure (types → logic → integration)
3. Write tests for critical paths
4. **Add WHY NOT comments for ALL triggers** (NON-NEGOTIABLE)

### 8.3 Code Review Assistance
- Verify SRP, DRY, SSOT adherence
- Flag structural defects
- **Audit WHY NOT comments** - if trigger exists without comment, add it

### 8.4 Trade-off Presentation
```typescript
// Approach 1: Library (Recommended)
// Pros: Battle-tested, maintained
// Cons: Adds dependency
import { debounce } from 'lodash';

// Approach 2: Custom
// Pros: No dependency, lightweight
// Cons: Maintenance burden, edge case bugs
```

---

## 9. EXCEPTIONS

**When to break rules:**
- Prototyping (speed > perfection)
- Legacy integration (consistency)
- Performance-critical (after measuring)

**Required when breaking:**
1. Document reason in comment
2. Add TODO/FIXME with context
3. Get team consensus

