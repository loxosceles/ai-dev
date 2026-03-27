---
name: pattern-writing
description: Guidelines for creating consistent architecture pattern documents with proper structure. Follow when creating or updating pattern files.
type: guideline
---

# Pattern Writing Guide

**This is a strict guideline.** Follow these rules exactly.

Guidelines for creating consistent, useful architecture pattern documents.

---

## Pattern Document Structure

Every pattern document MUST follow this structure:

### 1. Title (Descriptive, Not Generic)

✅ Good: "Static Frontend Hosting with Edge Authentication"
✅ Good: "Environment Deployment Strategy"
❌ Bad: "Frontend Architecture Overview"
❌ Bad: "Infrastructure Patterns"

**Rule**: Title should clearly indicate what problem is being solved or what approach is being described.

---

### 2. Problem Statement

**Format**:
```markdown
**Problem**: [One sentence describing the problem this pattern solves]
```

**Example**:
```markdown
**Problem**: Need low-cost, globally distributed frontend hosting with authentication but without requiring users to log in.
```

**Rules**:
- One clear sentence
- Focus on the problem, not the solution
- Should resonate with someone facing this issue

---

### 3. Solution Statement

**Format**:
```markdown
**Solution**: [One sentence describing the approach]
```

**Example**:
```markdown
**Solution**: Static site on S3 + CloudFront CDN + Lambda@Edge for transparent authentication.
```

**Rules**:
- One clear sentence
- High-level approach only
- Details come later

---

### 4. Pattern Section

**Required Content**:
- Architecture diagram (ASCII or description)
- Key components (bullet list)
- Brief explanation of how it works

**Example**:
```markdown
## Pattern

**Architecture**:
```
[ASCII diagram or description]
```

**Key Components**:
- **Component A**: What it does
- **Component B**: What it does
```

**Rules**:
- Keep it visual and scannable
- No implementation details yet
- Focus on the "what" not the "how"

---

### 5. Why This Pattern?

**Required Content**:
- Benefits (why use this)
- Use cases (when to use this)

**Example**:
```markdown
## Why This Pattern?

**Benefits**:
- **Low Cost**: No servers running 24/7
- **Global Performance**: CloudFront edge locations worldwide
- **Scalability**: Handles traffic spikes automatically

**Use Cases**:
- Portfolio sites with personalized access
- Demo applications with invite-only access
```

**Rules**:
- Be specific about benefits (not vague like "better performance")
- Include real use cases
- Help reader decide if this applies to them

---

### 6. Implementation (Optional)

**When to Include**: Only if implementation is non-obvious or has important details

**Format**:
```markdown
## Implementation

**Component A**:
```code example```

**Component B**:
```code example```
```

**Rules**:
- Keep code examples minimal and generic
- Use placeholder names (PROJECT_ID, environment, etc.)
- Focus on the pattern, not project-specific details
- Language-agnostic when possible

---

### 7. Tradeoffs / Constraints (If Applicable)

**When to Include**: When pattern has important limitations or constraints

**Example**:
```markdown
## Critical Tradeoff: Lambda@Edge Region Requirement

**Constraint**: Lambda@Edge functions MUST be deployed in **us-east-1** region.

**Why**: CloudFront is a global service managed from us-east-1.

**Implications**:
- Your CDK stack must deploy to us-east-1
- Cross-region references become complex
```

**Rules**:
- Be explicit about constraints
- Explain WHY the constraint exists
- Describe implications for implementation

---

### 8. When NOT to Use (Optional but Recommended)

**Format**:
```markdown
## When NOT to Use

- **Scenario A**: Use X instead
- **Scenario B**: Use Y instead
```

**Rules**:
- Help readers avoid misapplying the pattern
- Suggest alternatives when appropriate

---

### 9. Related Patterns

**Format**:
```markdown
## Related Patterns

- [Pattern Name](path/to/pattern.md) - Brief description of relationship
```

**Rules**:
- Link to related patterns
- Explain the relationship briefly

---

## What NOT to Include

❌ **Project-Specific Details**:
- Don't mention specific project names
- Don't list specific database tables
- Don't include project-specific configuration

❌ **Technology Lock-In** (unless necessary):
- Avoid "You must use GraphQL"
- Prefer "API layer (GraphQL/REST)"
- Show the pattern, not the specific tech

❌ **Outdated Information**:
- Don't reference deleted files
- Don't include deprecated approaches
- Keep it current

❌ **Generic Overviews**:
- Don't create "Overview" documents
- Every document should solve a specific problem
- If it's too broad, split it up

---

## Writing Checklist

Before submitting a pattern document, verify:

- [ ] Title is descriptive and specific
- [ ] Problem statement is one clear sentence
- [ ] Solution statement is one clear sentence
- [ ] Pattern section includes architecture and key components
- [ ] "Why This Pattern?" explains benefits and use cases
- [ ] Tradeoffs/constraints are documented (if applicable)
- [ ] Code examples are minimal and generic
- [ ] No project-specific details included
- [ ] Related patterns are linked
- [ ] Document is scannable (good use of headers, bullets, bold)

---

## Example Pattern Outline

```markdown
# [Descriptive Pattern Name]

**Problem**: [One sentence]

**Solution**: [One sentence]

---

## Pattern

**Architecture**:
[Diagram or description]

**Key Components**:
- **Component**: Description

---

## Why This Pattern?

**Benefits**:
- **Benefit**: Explanation

**Use Cases**:
- Scenario description

---

## Implementation

[Optional: Key implementation details]

---

## Tradeoffs

[Optional: Important constraints or limitations]

---

## When NOT to Use

[Optional: Anti-patterns or alternatives]

---

## Related Patterns

- [Pattern](link) - Relationship
```

---

## For AI/LLM Assistants

When asked to create or update a pattern document:

1. **Extract the pattern**: What problem does this solve? What's the approach?
2. **Remove project specifics**: Generalize names, technologies, details
3. **Focus on WHY**: Why was this approach chosen? What are the tradeoffs?
4. **Keep it scannable**: Use headers, bullets, bold text
5. **Link related patterns**: Help readers navigate
6. **Validate against checklist**: Ensure all required sections present

**Remember**: Patterns explain proven solutions to recurring problems. They're not project documentation or technology tutorials.
