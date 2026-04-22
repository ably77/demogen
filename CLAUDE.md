# Demogen — AI Demo Generator for Solo.io

## What This Is

A Claude Code skill + template repo that generates complete, standalone demo repositories for Solo.io's Istio Ambient Mesh + Enterprise Agentgateway stack, customized to any industry vertical.

## How to Use

```bash
# Clone the repo
git clone <repo-url> demogen
cd demogen

# Invoke the skill in Claude Code
# /generate-demo
# (or just tell Claude: "generate a healthcare demo")
```

The skill asks 9 questions about the target vertical and generates a complete repo.

## Project Structure

```
demogen/
├── .claude/skills/generate-demo.md   # The generator skill
├── templates/                         # Skeleton + template files
│   ├── demo-ui/                       # Streamlit chatbot UI
│   ├── services/                      # Backend services
│   └── k8s/                           # Kubernetes manifests
├── docs/superpowers/                  # Design specs and plans
├── CLAUDE.md                          # This file
└── README.md
```

## Template Strategy

- **Skeleton files** (copied as-is): theme.py, gateway.py, display.py, kubectl.py, Dockerfiles, backend.yaml, Grafana dashboard
- **Template files** (.tmpl, variable substitution): sidebar.py, route.yaml, install.sh, cleanup.sh, build-and-redeploy.sh
- **Generated files** (Claude writes from scratch): Homepage.py, config.py, mock data, mesh policies, guardrails, workshop.md, CLAUDE.md

## Reference Implementation

The enrollment-agent repo is the reference implementation (WGU education vertical):
`/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent`

## Updating Templates

When the enrollment-agent patterns change:
1. Update the corresponding skeleton/template file in `templates/`
2. If a generated file pattern changed, update the skill instructions in `.claude/skills/generate-demo.md`
