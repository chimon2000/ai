---
trigger: model_decision
description: "Best practices for creating and maintaining agent rules in the .augment/rules/ directory"
---

#### Rule Writing Best Practices

##### Rule Quality Standards

**Focused and Actionable:**
- Each rule addresses a single domain or concern (e.g., testing, architecture, error handling)
- Provide specific, actionable guidance that can be immediately applied
- Use imperative language: "Use X for Y" instead of "You should consider using X"
- Avoid vague phrases like "consider", "might want to", "it's good to"

**Scoped Appropriately:**
- Keep rules under 500 lines maximum
- If a rule exceeds 500 lines, split it into multiple composable rules
- Example: Instead of one "frontend.md" rule, create separate rules for "flutter.md", "riverpod.md", "testing.md"

**Concrete and Specific:**
- Always include code examples showing ❌ BAD and ✅ GOOD patterns
- Reference actual files in the codebase when applicable
- Use decision trees or flowcharts for complex decision-making
- Provide specific file paths, class names, or function signatures
- Include at least one complete example per concept

**Clear and Scannable:**
- Write rules like internal documentation, not conversational guidance
- Use bullet points, tables, and code blocks for scannability
- Use clear section headings with #### for subsections
- Keep explanations concise - one-liners preferred over paragraphs
- Use bold for key terms and concepts

**Composability:**
- Rules should be composable and reference each other when needed
- Example: `testing.md` can reference patterns from `riverpod.md`
- Avoid duplicating content across rules - link to canonical source
- Use consistent terminology across all rules

**Frontmatter:**
```yaml
---
type: "agent_requested"
description: "Brief description of when this rule applies"
---
```

##### When to Create New Rules

**Create a new rule when:**
- You find yourself repeating the same guidance across multiple conversations
- A domain has grown complex enough to warrant dedicated documentation (>100 lines of guidance)
- You need to establish patterns for a new technology or framework
- Existing rules exceed 500 lines and can be logically split

**Don't create a rule when:**
- The guidance is project-specific and won't apply to future work
- The pattern is already well-documented in existing rules
- The guidance is too vague or situational to be actionable

##### Rule Maintenance

**Regular Review:**
- When simplifying rules, aim for 30-50% reduction while preserving all unique patterns
- Remove verbose explanations and redundant examples
- Keep all decision trees, anti-pattern examples (❌/✅), and migration guides
- Consolidate duplicate content across sections
- Verify all code examples still compile and follow current patterns

**Quality Checklist:**
- [ ] Under 500 lines
- [ ] Includes concrete code examples
- [ ] Uses ❌ BAD / ✅ GOOD comparisons for anti-patterns
- [ ] Scannable with clear headings and bullet points
- [ ] No vague language ("consider", "might", "should probably")
- [ ] Frontmatter metadata present
- [ ] References specific files/patterns when applicable
- [ ] All code examples are concise and focused
- [ ] No redundant or duplicate content

##### Rule Reuse Workflow

When you find yourself giving the same guidance repeatedly:

1. **Check existing rules** - Does an existing rule cover this pattern?
2. **Create if needed** - If not, create a new rule file
3. **Reference in chat** - Link to the rule in future conversations
4. **Update patterns** - Refine the rule if you discover better approaches
5. **Consolidate** - Merge related rules if they become redundant

##### Rule Organization

**File naming:**
- Use lowercase with underscores: `rule_name.md`
- Be specific: `riverpod.md` not `state_management.md`
- Group related concepts: `testing.md`, `architecture.md`, `flutter.md`

**Directory structure:**
```
.augment/rules/
├── general.md              # Cross-cutting principles
├── testing.md              # Testing strategy and patterns
├── architecture.md         # Project structure and organization
├── flutter.md              # Flutter-specific patterns
├── riverpod.md             # Riverpod state management
├── documentation.md        # Documentation standards
├── refactoring.md          # Refactoring guidelines
├── tidy_first.md           # Structural vs behavioral changes
├── exhaustive_type_checking.md  # Type checking patterns
└── rule_writing.md         # This file
```