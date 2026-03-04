# spec-simple

Simplified Spec-Driven Development (SDD) toolkit for Claude Code.

Single binary CLI that bootstraps SDD projects with slash commands and templates.

## Install

```bash
brew tap letsur-dev/tap
brew install spec-simple
```

Or download from [GitHub Releases](https://github.com/letsur-dev/homebrew-tap/releases).

## Usage

### Initialize a project

```bash
# Create new project directory
spec-simple init my-project

# Initialize in current directory
spec-simple init .

# Options
spec-simple init my-project --no-git    # Skip git init
spec-simple init my-project --force     # Overwrite existing
```

### Check prerequisites

```bash
spec-simple check
```

### SDD Workflow

After `init`, use the slash commands in Claude Code:

1. `/spec-simple.specify` — Create feature specification from description
2. `/spec-simple.plan` — Generate implementation plan from spec
3. `/spec-simple.implement` — Execute plan phase by phase with review

## Project Structure

After `spec-simple init`, your project will contain:

```text
my-project/
├── .claude/commands/
│   ├── spec-simple.specify.md
│   ├── spec-simple.plan.md
│   └── spec-simple.implement.md
├── .spec-simple/
│   ├── config.json
│   └── templates/
│       ├── spec-template.md
│       └── plan-template.md
└── specs/
```

## Build from Source

```bash
cd packages/spec-simple
go build -o spec-simple .
./spec-simple --version
```

## Release

Tag push triggers automated cross-platform build via GoReleaser:

```bash
git tag spec-simple/v0.x.x
git push origin spec-simple/v0.x.x
```

Builds for: macOS (arm64/amd64), Linux (arm64/amd64), Windows (arm64/amd64)
