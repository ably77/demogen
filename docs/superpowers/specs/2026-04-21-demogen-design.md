# Demogen: AI-Powered Demo Generator for Solo.io

## Overview

Demogen is a Claude Code skill + template repo that generates complete, standalone demo repositories for Solo.io's Istio Ambient Mesh + Enterprise Agentgateway stack, customized to any industry vertical. An SE clones demogen, invokes the skill, answers questions about their customer's domain, and gets a repo ready to `./install.sh` and demo.

The reference implementation is the [enrollment-agent](../../../) repo (WGU education vertical). Demogen extracts the reusable patterns from that repo and makes them vertical-agnostic.

## Design Decisions

### Hybrid template strategy

Files are split into three categories:

- **Skeleton files** — Structurally identical across verticals. Copied as-is to the output repo. Examples: theme.py, gateway.py, Dockerfiles.
- **Template files** (.tmpl) — Same structure but with variable placeholders (`{{NS_BACKEND}}`, `{{REGISTRY}}`). Processed via string substitution.
- **Generated files** — Domain-specific content that Claude writes from scratch based on the SE's answers. Examples: mock data, system prompts, guardrails, workshop docs.

**Why:** Infrastructure plumbing rarely changes and benefits from being concrete files (no drift, easy to diff). Domain-specific content is exactly what Claude excels at generating contextually — a template with placeholders can't produce realistic mock patient records or a HIPAA-framed workshop narrative.

### Skill lives inside the demogen repo

The skill is at `.claude/skills/generate-demo.md` within the demogen repo itself. SEs clone demogen and invoke the skill from within it.

**Why:** Single repo to maintain. The skill can reference template files via relative paths. No plugin installation step.

### Output is a complete standalone repo

The skill generates a fully independent repo (like enrollment-agent). No runtime dependency on demogen. The generated repo has its own install.sh, workshop.md, CLAUDE.md, etc.

**Why:** Generated repos are handed off to customers or used in workshops. They must be self-contained and not reference internal tooling.

### Output path can be new or existing

If the target path already exists (e.g., an already-created git repo), the skill writes into it, preserving existing `.git/`, `.gitattributes`, and other files. If it doesn't exist, the skill creates it and initializes a git repo.

### Both install modes (full + demo-only)

Generated repos include an `install.sh` that supports both full-stack deployment (Istio + agent gateway + workloads) and demo-only mode (workloads only, assumes infrastructure exists).

**Why:** SEs may be working with pre-provisioned clusters (from solo-field-installer) or starting from scratch.

### Always generate a mock data service

Every generated repo includes a graph-db-mock equivalent with domain-appropriate fake data (3 sample entities). No real DB dependency.

**Why:** Demos must be self-contained and deployable in minutes. Real backends add setup friction and failure modes.

### Generic gRPC ext-authz (not ABAC-specific)

The template includes a generic gRPC ext-authz server (`ably7/grpc-ext-authz:latest`) that conforms to the Envoy External Authorization gRPC proto. It's always deployed as part of install.sh. The default behavior allows requests with the `x-ext-authz: allow` header.

During demo generation, the skill asks whether ext-authz should be enabled and how it should be presented in the demo. The Homepage includes a toggle that applies/removes the `EnterpriseAgentgatewayPolicy` to show the ext-authz flow live.

SEs can fork the upstream repo (github.com/ably77/grpc-ext-authz) to implement custom authorization logic (ABAC, RBAC, API key validation, etc.) for their specific customer demo.

**Why:** The ABAC-specific server from enrollment-agent is too opinionated for a generic template. The generic ext-authz pattern showcases the Enterprise Agentgateway's extensibility without prescribing a specific authorization model.

### Multi-cluster is always included

Every generated repo includes multi-cluster Istio setup with global service routing and failover. The data-product-api equivalent is always the service where failover is demonstrated — deployed to both clusters with `solo.io/service-scope=global` labels.

The generated repo includes:
- Multi-cluster Istio linking in install.sh
- Backend service deployment to cluster2
- A Multi-Cluster demo page showing failover (scale down → automatic routing to cluster2 → restore)
- Global service labels and `PreferNetwork` annotations

**Why:** Multi-cluster routing and failover is a core Solo.io value proposition. Every demo should showcase this capability.

## Repo Structure

```
demogen/
├── .claude/
│   └── skills/
│       └── generate-demo.md              # The generator skill
├── templates/
│   ├── demo-ui/
│   │   ├── utils/
│   │   │   ├── theme.py                  # Skeleton
│   │   │   ├── gateway.py                # Skeleton
│   │   │   ├── display.py                # Skeleton
│   │   │   ├── kubectl.py                # Skeleton
│   │   │   └── sidebar.py.tmpl           # Template — {{GRAFANA_HOST}}, {{UI_HOST}}
│   │   ├── assets/                       # Empty dir for architecture diagrams
│   │   └── Dockerfile                    # Skeleton
│   ├── services/
│   │   └── data-product-api/
│   │       ├── Dockerfile                # Skeleton
│   │       └── requirements.txt          # Skeleton
│   ├── k8s/
│   │   ├── gateway/
│   │   │   ├── route.yaml.tmpl           # Template — {{ROUTE_NAME}}
│   │   │   └── backend.yaml              # Skeleton
│   │   └── observability/                # Skeleton — Grafana dashboard JSON
│   ├── install.sh.tmpl                   # Template — namespace names, versions
│   ├── cleanup.sh.tmpl                   # Template — namespace names
│   ├── build-and-redeploy.sh.tmpl        # Template — {{REGISTRY}}, {{IMAGE_NAME}}
│   └── .gitignore                        # Skeleton
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-04-21-demogen-design.md  # This document
├── CLAUDE.md
└── README.md
```

## Skill Workflow

### Step 1: Gather Context

The skill asks these questions one at a time:

1. **Output path** — Where to create the repo. If the path exists, write into it (preserve `.git/` etc.). If not, create it.
2. **Organization** — Full name and short name (e.g., "Kaiser Permanente", "Kaiser").
3. **Vertical/domain** — What the chatbot advises on (e.g., "patient intake and appointment scheduling").
4. **Entity model** — Primary entity and key fields (e.g., "Patient — MRN, name, appointments, diagnoses, insurance").
5. **Advisor role** — What role the chatbot plays (e.g., "patient intake coordinator").
6. **Compliance regime** — What guardrails apply (e.g., HIPAA → MRN/PHI detection). Can be a known regime or custom regex patterns.
7. **Docker registry** — Where to push images (e.g., `ably7/`, `gcr.io/solo-demos/`).
8. **Namespace names** — Backend and frontend Kubernetes namespace names (e.g., `health-demo`, `health-demo-frontend`).
9. **BYO ext-authz** — "Do you want BYO ext-authz enabled for this demo?" If yes, the generic gRPC ext-authz server is deployed and the Homepage includes a toggle to apply/remove the policy. The SE can describe custom authorization logic to showcase, or use the default header-based allow/deny.

### Step 2: Generate the Repo

In order:

1. Create output directory (or verify existing path)
2. Copy all skeleton files as-is from `templates/`
3. Process all `.tmpl` files — substitute `{{VARIABLES}}` and write without the `.tmpl` extension
4. Generate domain-specific files using Claude (see Generated Files below)
5. Generate CLAUDE.md for the output repo (architecture docs, build instructions)
6. Initialize git repo if one doesn't exist

### Step 3: Verification

1. List all generated files for the SE to review
2. Run syntax checks (Python, YAML) if tools available
3. Print next steps: cd into the repo, review key files, then `./install.sh`

## Template Variables

These are substituted into `.tmpl` files:

| Variable | Source | Example |
|----------|--------|---------|
| `{{ORG_NAME}}` | Question 2 | Kaiser Permanente |
| `{{ORG_SHORT}}` | Question 2 | Kaiser |
| `{{APP_TITLE}}` | Derived | Kaiser Patient Advisor |
| `{{ADVISOR_ROLE}}` | Question 5 | patient intake coordinator |
| `{{DOMAIN}}` | Question 3 | patient intake and appointment scheduling |
| `{{ENTITY_NAME}}` | Question 4 | Patient |
| `{{ENTITY_ID_FORMAT}}` | Question 4 | KP_2024_00142 |
| `{{NS_BACKEND}}` | Question 8 | health-demo |
| `{{NS_FRONTEND}}` | Question 8 | health-demo-frontend |
| `{{REGISTRY}}` | Question 7 | ably7/ |
| `{{IMAGE_PREFIX}}` | Derived from domain | health-demo |
| `{{ROUTE_NAME}}` | Derived from domain | kaiser-patient |
| `{{GRAFANA_HOST}}` | Derived (default: grafana.glootest.com, SE can override) | grafana.glootest.com |
| `{{UI_HOST}}` | Derived (default: ui.glootest.com, SE can override) | ui.glootest.com |
| `{{CHATBOT_HOST}}` | Derived from ORG_SHORT (default: `{short}.glootest.com`, SE can override) | patient.glootest.com |

## Generated Files

These are written from scratch by Claude based on SE answers:

### `demo-ui/Homepage.py`

- **Tool definition:** Function name derived from entity (e.g., `get_patient_data`), description matching the domain, required parameters (entity ID)
- **System prompt:** Role, org name, what the advisor helps with, compliance context
- **Entity selector:** Sidebar dropdown with sample entity IDs from the mock data
- **BYO ext-authz toggle** (if enabled): applies/removes `EnterpriseAgentgatewayPolicy`, shows header-based allow/deny flow
- **Function calling flow:** Same pattern as enrollment-agent — LLM requests data, chatbot executes via data-product-api through the mesh

### `demo-ui/utils/config.py`

- Env var defaults swapped to the vertical
- DATA_PRODUCT_URL and GRAPH_DB_URL using chosen namespace names
- Default entity list matching the mock data
- System prompt template for the domain

### `demo-ui/pages/1_Mesh_Policies.py`

- Policy toggle demo reframed for the vertical
- YAML examples use correct namespace names and service principals
- Compliance framing matches the regime (HIPAA instead of FERPA, etc.)

### `demo-ui/pages/2_Multi_Cluster.py`

- Multi-cluster failover demo showing global service routing
- 4-tab flow: Enable Global Service → Simulate Failover → Verify Failover → Restore
- The data-product-api is the failover service (same pattern as enrollment-agent)
- Domain-framed descriptions (e.g., "patient data API" instead of "student data API")
- Uses correct namespace names and service names from SE answers

### `services/data-product-api/app.py`

- Generated (not templated) because route paths and query format depend on the entity model
- Endpoints use entity-specific paths (e.g., `GET /patients/{patient_id}` instead of `/students/{student_id}`)
- Graph DB query format uses entity-appropriate Cypher syntax
- GRAPH_DB_URL from env var, default pointing to graph-db-mock in backend namespace

### `services/graph-db-mock/app.py`

- 3 sample entities with realistic fake data for the domain
- Same API shape: `POST /query` with Cypher-like syntax, `GET /health`
- Data model matches the entity fields from Question 4
- Example for healthcare: 3 patients with MRNs, appointment histories, diagnoses, insurance providers

### `k8s/namespaces.yaml`

- Two namespaces with ambient mesh labels
- Names from Question 8

### `k8s/services/*.yaml`

- **chatbot.yaml** — Deployment + Service + ServiceAccount for the demo UI. Env vars wired: ORG_NAME, ORG_SHORT, APP_TITLE, DATA_PRODUCT_URL, GRAPH_DB_URL, NS_BACKEND, NS_FRONTEND, gateway config.
- **data-product-api.yaml** — Deployment + Service + ServiceAccount. GRAPH_DB_URL pointing to mock service in backend namespace.
- **graph-db-mock.yaml** — Deployment + Service + ServiceAccount. No env vars (data in-memory).
- Image names use chosen registry + vertical-specific names.

### `k8s/mesh/*.yaml`

- **deny-all.yaml** — Default deny for both namespaces
- **chatbot-to-data-product.yaml** — L7 allow with targetRefs, principal = chatbot SA in frontend namespace
- **data-product-to-graphdb.yaml** — L7 allow with targetRefs, principal = data-product-api SA in backend namespace
- **waypoint-to-backends.yaml** — L4 allow for waypoint SA (second hop through ztunnel)
- **waypoint.yaml** — Gateway API waypoint for backend namespace
- **telemetry.yaml** — Access logging for the backend namespace

### `k8s/gateway/guardrails.yaml`

- Compliance-specific regex patterns based on the regime:
  - **HIPAA:** MRN patterns, PHI detection (DOB, SSN), medical record numbers
  - **PCI:** Credit card numbers, CVV, account numbers
  - **FERPA:** Student ID patterns, SSN
  - **Custom:** SE-provided regex patterns
- Route name matching the vertical
- Response messages framed for the compliance context

### `workshop.md`

- Full 7-section guided walkthrough, reframed for the vertical:
  1. Infrastructure setup (Istio ambient, agent gateway) — mostly generic, org name swapped
  2. Mesh enrollment and mTLS verification — namespace names swapped
  3. Agent gateway configuration — generic
  4. Authorization policies — domain-framed (e.g., "ensuring only the intake coordinator can access patient records")
  5. Guardrails and compliance — framed for the regime (HIPAA, PCI, etc.)
  6. BYO External Authorization (ext-authz) — if enabled
  7. End-to-end "home run" — domain-appropriate scenario (e.g., patient checking appointment status)

### `CLAUDE.md`

- Architecture documentation for the generated repo
- Build and deploy instructions
- Key design decisions (same patterns as enrollment-agent's CLAUDE.md but domain-swapped)

## Skeleton Files (Copied As-Is)

These files are identical across all verticals:

| File | Why it's generic |
|------|-----------------|
| `demo-ui/utils/theme.py` | Pure CSS, Solo design system |
| `demo-ui/utils/gateway.py` | Session-based gateway config, no domain logic |
| `demo-ui/utils/display.py` | Rendering helpers for errors, tool calls, request chains |
| `demo-ui/utils/kubectl.py` | kubectl subprocess wrapper |
| `demo-ui/Dockerfile` | Standard Streamlit container build |
| `services/data-product-api/Dockerfile` | Standard FastAPI container |
| `services/data-product-api/requirements.txt` | FastAPI + httpx + pytest |
| `k8s/gateway/backend.yaml` | Standard agent gateway backend config |
| `k8s/observability/*` | Grafana dashboard JSON |
| `.gitignore` | Standard Python/K8s ignores |

## Future Considerations (Out of Scope)

- **MCP support:** Agent gateway supports MCP natively. Future version could offer MCP tool mode instead of OpenAI function calling.
- **Real database backends:** Pluggable data layer (Neptune, Postgres, etc.) instead of always mocking.
- **Multi-page demo expansion:** Additional demo pages beyond Mesh Policies (e.g., rate limiting demo, tracing demo).
- **CI/CD templates:** GitHub Actions for building and deploying generated repos.
- **Vertical-specific architecture diagrams:** Auto-generated diagrams for the assets/ directory.
