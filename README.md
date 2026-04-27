# Demogen

AI-powered demo generator for Solo.io. Generates complete, standalone demo repos for Istio Ambient Mesh + Enterprise Agentgateway, customized to any industry vertical.

## Quick Start

1. Clone this repo
2. Open in Claude Code
3. Run `/generate-demo` (or ask Claude to generate a demo)
4. Answer questions about your target vertical
5. Get a complete repo — ready to `./install.sh`

## What Gets Generated

A standalone repo containing:
- **Streamlit chatbot** with AI function calling, BYO ext-authz toggle, guardrail demos
- **Mock data service** with realistic domain-specific fake data
- **Data product API** secured through the mesh with mTLS
- **Kubernetes manifests** for Istio ambient mesh policies, agent gateway config, observability
- **Multi-cluster setup** with global service routing and failover demo
- **Workshop doc** with guided walkthrough for platform engineers and leaders
- **Install/cleanup scripts** with full-stack and demo-only modes

## Supported Verticals

Any — the skill generates domain-appropriate content for:
- Healthcare (HIPAA), Financial Services (PCI), Education (FERPA)
- Or custom compliance regimes with your own regex patterns

## Prerequisites

- Claude Code with access to this repo
- Two Kubernetes clusters (for multicluster demo) or one (single-cluster mode)
- Solo.io license key and OpenAI API key
