---
type: "always_apply"
description: "Documentation standards for README files and project documentation"
---

#### Documentation Standards

##### README File Length Limit

**Maximum Length:**
- README.md files should **never exceed 400 lines** (not characters)
- Keep content concise, scannable, and focused on essential information
- Remove verbose explanations, redundant examples, and excessive prose

**When Additional Context is Needed:**

If a README would exceed 400 lines with necessary information, use these strategies:

1. **Split into separate markdown files** - Create focused docs:
   - `DEPLOYMENT.md` - Deployment procedures, CI/CD, release processes
   - `DEVELOPMENT.md` - Development setup, local workflows, debugging
   - `TROUBLESHOOTING.md` - Common issues, error messages, solutions
   - `ARCHITECTURE.md` - System design, patterns, technical decisions
   - `API.md` - API documentation, endpoints, examples
   - `CONTRIBUTING.md` - Contribution guidelines, code standards

2. **Use visual diagrams** - Replace lengthy text explanations with:
   - **Mermaid sequence diagrams** for workflows and processes
   - **Mermaid flowcharts** for decision trees and logic flows
   - **Mermaid architecture diagrams** for system overviews
   - **Mermaid state diagrams** for state machines and lifecycle flows

3. **Link to detailed docs** - Keep README as high-level overview:
   ```markdown
   ## Documentation

   - [Getting Started](docs/GETTING_STARTED.md)
   - [Deployment Guide](docs/DEPLOYMENT.md)
   - [Architecture Overview](docs/ARCHITECTURE.md)
   - [Troubleshooting](docs/TROUBLESHOOTING.md)
   ```

##### README Best Practices

**Structure:**
- **Quick-start first** - Get users running in < 5 minutes
- **Table of contents** - For READMEs > 100 lines
- **Sections in priority order:**
  1. Brief description (1-2 sentences)
  2. Quick start / Installation
  3. Common use cases
  4. Configuration (essential only)
  5. Links to detailed docs
  6. Troubleshooting (top 3-5 issues only)

**Content Guidelines:**
- ✅ **Prioritize quick-start information** and common use cases
- ✅ **Use tables, bullet points, inline code** for scannability
- ✅ **Include only essential examples** (one per concept)
- ✅ **Link to external docs** for deep dives
- ✅ **Use diagrams** to replace multi-paragraph explanations
- ❌ **Avoid verbose prose** - prefer concise bullet points
- ❌ **Avoid redundant examples** - one good example beats three similar ones
- ❌ **Avoid copy-pasting** - link to canonical docs instead

**Formatting for Scannability:**

**❌ BAD: Verbose prose**
```markdown
Before you begin, you will need to make sure that you have
Node.js version 18 or higher installed on your system. You
will also need Docker and kubectl.
```

**✅ GOOD: Concise with inline code**
```markdown
**Prerequisites:** Node.js 18+, Docker, `kubectl` installed
```

**❌ BAD: Paragraphs of text**
```markdown
In development mode, we use mock authentication for easier
testing, while in production we use OAuth 2.0. Similarly,
the database is SQLite in development but PostgreSQL in
production.
```

**✅ GOOD: Table for comparison**
```markdown
| Feature | Development | Production |
|---------|-------------|------------|
| Auth    | Mock        | OAuth 2.0  |
| DB      | SQLite      | PostgreSQL |
```

**Example Limits:**
- **One example per concept** - Show the most common use case
- **Inline examples** - Use inline code blocks for simple cases
- **Link to examples/** - For complex examples, link to separate files

**❌ BAD: Multiple similar examples**
```bash
./scripts/deploy.sh --env staging
./scripts/deploy.sh --env production
./scripts/deploy.sh --env staging --config custom.yml
./scripts/deploy.sh --env staging --verbose
```

**✅ GOOD: One clear example**
```bash
./scripts/deploy.sh --env staging
```
See [examples/](examples/) for advanced deployment scenarios.

##### Visual Diagrams Over Text

**When to use diagrams:**
- Workflows with > 3 steps
- System architecture with > 2 components
- Decision trees with > 2 branches
- State machines
- Sequence of interactions between components

**Mermaid Diagram Types:**
- **Flowchart** - Workflows and process flows
- **Sequence Diagram** - Interactions between components
- **State Diagram** - State machines and lifecycle flows
- **Graph** - System architecture and relationships

##### Documentation File Organization

**Project Root:**
```
/
├── README.md              # High-level overview (< 400 lines)
├── CHANGELOG.md           # Version history
├── LICENSE.md             # License information
├── CONTRIBUTING.md        # How to contribute
└── docs/
    ├── GETTING_STARTED.md # Detailed setup guide
    ├── DEVELOPMENT.md     # Development workflows
    ├── DEPLOYMENT.md      # Deployment procedures
    ├── ARCHITECTURE.md    # System architecture
    ├── API.md             # API documentation
    ├── TROUBLESHOOTING.md # Common issues
    └── diagrams/          # Mermaid/image files
```

**Package/Module Level:**
```
/packages/feature/
├── README.md              # Feature overview (< 400 lines)
├── docs/
│   ├── API.md            # Public API
│   └── EXAMPLES.md       # Usage examples
└── examples/             # Code examples
```

##### Enforcement

**Before creating/updating a README:**
1. Check if content exceeds 400 lines
2. If yes, identify sections that can be:
   - Moved to separate docs
   - Replaced with diagrams
   - Condensed or removed
3. Create separate documentation files as needed
4. Update README with links to detailed docs

**Line count check:**
```bash
wc -l README.md
# Should output <= 400
```

##### Examples

**❌ BAD: Verbose, > 400 lines**
```markdown
# My Project

This is a comprehensive guide to my project. In this document,
you will find everything you need to know about installation,
configuration, deployment, troubleshooting, and more.

## Installation

First, you need to install Node.js. You can download it from...
[50 lines of installation instructions]

## Configuration

There are many configuration options available...
[100 lines of configuration details]

## Deployment

To deploy the application, you need to...
[150 lines of deployment procedures]

## Troubleshooting

Here are all the possible errors you might encounter...
[200 lines of troubleshooting]
```

**✅ GOOD: Concise, < 400 lines, with links**
```markdown
# My Project

A brief description of what this project does.

## Quick Start

```bash
npm install
npm start
```

Visit http://localhost:3000

## Documentation

- [Getting Started](docs/GETTING_STARTED.md) - Detailed setup
- [Configuration](docs/CONFIGURATION.md) - All config options
- [Deployment](docs/DEPLOYMENT.md) - Deploy to production
- [Troubleshooting](docs/TROUBLESHOOTING.md) - Common issues

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md)
```

##### Summary

- **400-line hard limit** for all README.md files
- **Split large docs** into focused files (DEPLOYMENT.md, etc.)
- **Use Mermaid diagrams** to replace verbose text explanations
- **Prioritize scannability** - tables, bullets, inline code
- **One example per concept** - link to more examples
- **Link to detailed docs** - README is the entry point, not the encyclopedia

