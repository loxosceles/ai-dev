---
name: core-principles
description: Universal coding principles that apply to all projects. These preferences take precedence over conflicting advice from third-party skills. Follow when writing or reviewing any code.
type: guideline
---

# Core Principles

**This is a strict guideline.** Follow these rules. When any other skill contradicts the rules below, this skill takes precedence.

Essential guidelines that apply to ALL projects.

**Important**: These are guiding principles, not absolute rules. Project context matters. If you need to deviate, explain your reasoning and trade-offs.

**Note on Examples**: Code examples may use specific languages (JavaScript, Python, etc.) for clarity, but principles apply universally. Adapt patterns to your project's language and context.

---

## 1. Prefer Pure Functions

Helper functions should accept data as parameters rather than reading from globals or environment.

```typescript
// ✅ Preferred: Pure, testable, moveable
function buildResourceName(environment: string, type: string, name: string): string {
  return `${environment}-${type}-${name}`;
}

// ⚠️ Less ideal: Reads from global
const ENV = process.env.ENVIRONMENT;
function buildResourceName(type: string, name: string): string {
  return `${ENV}-${type}-${name}`;
}
```

**When impurity is necessary** (AWS calls, DB access): Keep scope minimal, pass clients as parameters when feasible.

## 2. Avoid Fallbacks for Environment Variables

Prefer explicit failures over silent fallbacks to catch misconfigurations early.

```javascript
// ⚠️ Avoid when possible
const region = process.env.AWS_REGION || 'us-east-1';

// ✅ Preferred
if (!process.env.AWS_REGION) {
  throw new Error('AWS_REGION is required. Set in environment or SSM.');
}
const region = process.env.AWS_REGION;
```

**Exception**: Reasonable defaults for truly optional configuration are acceptable.

## 3. Code Structure and Organization

Aim for consistent file structure and clear separation of concerns.

**Recommended File Structure**:
1. Imports
2. Environment variable validation (fail fast on cold start)
3. Constants
4. AWS clients and expensive initializations
5. Types/Interfaces
6. Pure helper functions
7. Impure functions (AWS calls, I/O)
8. Entry point (handler, main, etc.)

**Separation of Concerns**:
- Prefer separate files for schemas, prompts, and business logic
- Avoid mixing unrelated functionality
- Balance between separation and over-fragmentation

**Function Declaration**:
- **Simple/inline**: Arrow functions `const fn = () => {}`
- **Complex logic**: Function keyword `function fn() {}`

## 4. Project Identifiers in Configuration

Avoid hardcoding project identifiers. Use configuration instead.

```typescript
// ❌ Avoid
const paramPath = `/myapp/${environment}/api-key`;

// ✅ Preferred
const PROJECT_ID = process.env.PROJECT_ID;
const paramPath = `/${PROJECT_ID}/${environment}/api-key`;
```

## 5. Error Handling

Balance between failing fast and graceful degradation based on context.

- **Throw**: Missing configuration, invalid setup, critical dependencies
- **Log + Continue**: UI enhancements, optional features, non-blocking operations
- **Always provide context**: Whether throwing or logging, include actionable information

## 6. Variable Naming

Use meaningful variable names. Avoid single-letter names except in well-established conventions.

```typescript
// ❌ Avoid
arr.map((a) => a.id);
catch (e) { console.error(e); }

// ✅ Preferred
users.map((user) => user.id);
catch (err) { console.error(err); }
```

**Acceptable abbreviations**: `err`, `evt`, `req`, `res`, `ctx`, `idx`, `fn`

**Exceptions**: Math formulas (`x`, `y`, `i`, `j`), trivial arrow functions (`[1,2,3].map(n => n * 2)`)

## 7. Security Baseline

- Keep secrets, API keys, and credentials out of repositories
- Use AWS Secrets Manager or SSM Parameter Store (SecureString)
- Fetch secrets at runtime, not build time
- Prefer IAM roles over hardcoded credentials
- Separate resources and secrets per environment
