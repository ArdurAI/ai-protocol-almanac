# Setup Guide

How to set up the AI Protocol Almanac repository for local development, GitHub publication, and automated updates.

## Prerequisites

- Git (2.30+)
- GitHub CLI (`gh`) (optional, for PRs and issues)
- Python 3.10+ (for roster validation and automation scripts)
- `jq` (optional, for querying roster.json)

## Clone the repository

```bash
git clone https://github.com/ArdurAI/ai-protocol-almanac.git
cd ai-protocol-almanac
```

## Validate the roster

```bash
python3 -c "import json; json.load(open('data/roster.json'))" && echo "Valid JSON"
```

For detailed validation:

```bash
python3 -m json.tool data/roster.json > /dev/null && echo "Valid JSON"
```

## Query the roster

```bash
# Count tools by tier
jq '[.tools[] | .tier] | group_by(.) | map({tier: .[0], count: length})' data/roster.json

# List all Tier A protocols
jq '.tools[] | select(.tier == "A") | .name' data/roster.json

# Find all protocols
jq '.tools[] | select(.type == "Protocol") | .name' data/roster.json

# Find all MIT-licensed tools
jq '.tools[] | select(.license | contains("MIT")) | .name' data/roster.json
```

## Git workflow

### Create a new edition

```bash
# Create the new edition file
cp editions/2026-06.md editions/2026-07.md

# Edit the new edition
# ...

# Update README to reference the latest edition
# Edit README.md: change "The latest file in editions/" to point to 2026-07.md

# Update roster.json version
# Edit data/roster.json: change meta.version to "2026-07"

# Commit
git add -A
git commit -m "Edition 2026-07: [title]"
```

### Add a new tool

```bash
# 1. Edit data/roster.json to add the tool
# 2. Create tools/<tool-name>.md
# 3. If Tier A, create standards/<tool-key>/README.md
# 4. Update the latest edition
# 5. Update README.md if Tier A

# Validate
git add -A
git commit -m "Add [Tool Name] to roster (Tier [A/B/C])"
```

## GitHub setup

### Configure remote

```bash
git remote -v
git remote set-url origin https://github.com/ArdurAI/ai-protocol-almanac.git
```

### Authenticate with GitHub CLI

```bash
gh auth status
gh auth login
```

### Push

```bash
git push origin main
```

## Automation

### Monthly edition cron

The monthly edition update is run by a scheduled job. See the cron configuration in `IMPLEMENTATION.md` for details.

### GitHub Actions (optional)

A GitHub Actions workflow for monthly metadata refresh is documented in `IMPLEMENTATION.md`.

## Directory structure

```
ai-protocol-almanac/
├── README.md              # Project overview
├── INTENT.md              # Philosophy and design principles
├── IMPLEMENTATION.md      # Implementation guide (this file)
├── TESTING.md             # Benchmark methodology
├── TROUBLESHOOTING.md     # Common issues and debugging
├── CONTRIBUTING.md        # How to contribute
├── architecture.md        # Stack architecture
├── SETUP.md               # This file
├── .gitignore
│
├── standards/             # Per-standard deep references (Tier A)
│   ├── mcp/README.md
│   ├── a2a/README.md
│   └── ...
│
├── editions/              # Monthly editions
│   └── 2026-06.md
│
├── benchmarks/            # Benchmark results (rolling)
│   └── README.md
│
├── methodology/
│   └── benchmark-harness.md
│
├── data/
│   └── roster.json        # Machine-readable catalog (46 tools)
│
├── tools/                 # Per-tool deep-dive pages
│   └── ...
│
└── assets/                # Diagrams, charts, tables
    └── ...
```

## License

Content: CC BY 4.0
Code: MIT
