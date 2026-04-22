# Demogen Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the demogen repo — a Claude Code skill + template files that generates complete, standalone demo repos for Solo.io's Ambient Mesh + Enterprise Agentgateway stack, customized to any industry vertical.

**Architecture:** Hybrid template strategy. Skeleton files are copied as-is from the enrollment-agent reference implementation. Template files (.tmpl) use `{{PLACEHOLDER}}` substitution. Domain-specific files (mock data, system prompts, mesh policies, workshop docs) are generated from scratch by Claude based on SE answers. A `.claude/skills/generate-demo.md` skill orchestrates the entire process.

**Tech Stack:** Bash, Python (Streamlit + FastAPI), Kubernetes YAML, Claude Code skills (Markdown)

**Spec:** `docs/superpowers/specs/2026-04-21-demogen-design.md`

**Reference implementation:** `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent`

---

### Task 1: Repository Scaffolding

**Files:**
- Create: `templates/demo-ui/utils/` (directory)
- Create: `templates/demo-ui/assets/` (directory with `.gitkeep`)
- Create: `templates/services/data-product-api/` (directory)
- Create: `templates/k8s/gateway/` (directory)
- Create: `templates/k8s/observability/` (directory)
- Create: `.claude/skills/` (directory)

- [ ] **Step 1: Create all directories**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
mkdir -p templates/demo-ui/utils
mkdir -p templates/demo-ui/assets
mkdir -p templates/services/data-product-api
mkdir -p templates/k8s/gateway
mkdir -p templates/k8s/observability
mkdir -p .claude/skills
```

- [ ] **Step 2: Create .gitkeep for assets directory**

Write an empty `.gitkeep` file to `templates/demo-ui/assets/.gitkeep` so the empty directory is tracked by git.

- [ ] **Step 3: Verify structure**

```bash
find templates -type d | sort
```

Expected:
```
templates
templates/demo-ui
templates/demo-ui/assets
templates/demo-ui/utils
templates/k8s
templates/k8s/gateway
templates/k8s/observability
templates/services
templates/services/data-product-api
```

- [ ] **Step 4: Commit**

```bash
git add -A
git commit -m "scaffold: create demogen repo directory structure"
```

---

### Task 2: Skeleton Files — Demo UI Utilities

Copy the generic, vertical-agnostic utility files from the enrollment-agent reference implementation. These files are identical across all verticals and require no modification.

**Files:**
- Create: `templates/demo-ui/utils/theme.py` — copied from enrollment-agent as-is (402 lines, Solo dark theme CSS)
- Create: `templates/demo-ui/utils/gateway.py` — copied from enrollment-agent as-is (42 lines, gateway HTTP helpers)
- Create: `templates/demo-ui/utils/display.py` — copied from enrollment-agent as-is (52 lines, chat rendering helpers)
- Create: `templates/demo-ui/utils/kubectl.py` — copied from enrollment-agent as-is (10 lines, subprocess wrapper)
- Create: `templates/demo-ui/Dockerfile` — copied from enrollment-agent as-is (18 lines, Streamlit container)

**Source directory:** `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/`

- [ ] **Step 1: Copy theme.py**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/utils/theme.py` and write it to `templates/demo-ui/utils/theme.py` exactly as-is.

- [ ] **Step 2: Copy gateway.py**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/utils/gateway.py` and write it to `templates/demo-ui/utils/gateway.py` exactly as-is.

- [ ] **Step 3: Copy display.py**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/utils/display.py` and write it to `templates/demo-ui/utils/display.py` exactly as-is.

- [ ] **Step 4: Copy kubectl.py**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/utils/kubectl.py` and write it to `templates/demo-ui/utils/kubectl.py` exactly as-is.

- [ ] **Step 5: Copy Dockerfile**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/Dockerfile` and write it to `templates/demo-ui/Dockerfile` exactly as-is.

- [ ] **Step 6: Verify Python syntax**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
python3 -c "import py_compile; py_compile.compile('templates/demo-ui/utils/theme.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('templates/demo-ui/utils/gateway.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('templates/demo-ui/utils/display.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('templates/demo-ui/utils/kubectl.py', doraise=True)"
```

Expected: No output (clean compile).

- [ ] **Step 7: Commit**

```bash
git add templates/demo-ui/
git commit -m "feat: add skeleton files for demo UI utilities and Dockerfile"
```

---

### Task 3: Skeleton Files — Services and K8s

Copy the generic service files and k8s configs that are identical across verticals.

**Files:**
- Create: `templates/services/data-product-api/Dockerfile` — standard FastAPI container (8 lines)
- Create: `templates/services/data-product-api/requirements.txt` — fastapi, uvicorn, requests, pytest, httpx
- Create: `templates/k8s/gateway/backend.yaml` — AgentgatewayBackend for OpenAI
- Create: `templates/k8s/observability/agentgateway-grafana-dashboard-v1.json` — Grafana dashboard (large JSON)
- Create: `templates/.gitignore` — standard Python/K8s ignores

**Source directories:**
- `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/services/`
- `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/`
- `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/.gitignore`

- [ ] **Step 1: Copy data-product-api Dockerfile and requirements.txt**

Read from enrollment-agent and write to templates:
- `services/data-product-api/Dockerfile` → `templates/services/data-product-api/Dockerfile`
- `services/data-product-api/requirements.txt` → `templates/services/data-product-api/requirements.txt`

- [ ] **Step 2: Copy backend.yaml**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/gateway/backend.yaml` and write to `templates/k8s/gateway/backend.yaml` exactly as-is.

- [ ] **Step 3: Copy Grafana dashboard JSON**

```bash
cp /Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/observability/agentgateway-grafana-dashboard-v1.json \
   /Users/alexly-solo/Desktop/solo/solo-github/demogen/templates/k8s/observability/agentgateway-grafana-dashboard-v1.json
```

- [ ] **Step 4: Copy .gitignore**

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/.gitignore` and write to `templates/.gitignore`. Remove the `build-and-redeploy.sh` and `docs/` entries (those should be tracked in generated repos).

- [ ] **Step 5: Commit**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
git add templates/services/ templates/k8s/ templates/.gitignore
git commit -m "feat: add skeleton files for services and k8s configs"
```

---

### Task 4: Template Files

Create the `.tmpl` files that use `{{PLACEHOLDER}}` variable substitution. These are derived from the enrollment-agent versions with hardcoded values replaced by template variables.

**Files:**
- Create: `templates/demo-ui/utils/sidebar.py.tmpl`
- Create: `templates/k8s/gateway/route.yaml.tmpl`
- Create: `templates/install.sh.tmpl`
- Create: `templates/cleanup.sh.tmpl`
- Create: `templates/build-and-redeploy.sh.tmpl`

- [ ] **Step 1: Create sidebar.py.tmpl**

Based on enrollment-agent's `demo-ui/utils/sidebar.py`. Replace:
- `ui.glootest.com` → `{{UI_HOST}}`
- `grafana.glootest.com` → `{{GRAFANA_HOST}}`

Write to `templates/demo-ui/utils/sidebar.py.tmpl`:

```python
import os
import streamlit as st
from utils.theme import inject_theme
from utils.config import APP_TITLE


def render_sidebar():
    """Render the sidebar with gateway config and observability info."""
    inject_theme()

    if "gateway_ip" not in st.session_state:
        st.session_state["gateway_ip"] = os.environ.get("GATEWAY_IP", "localhost")
    if "protocol" not in st.session_state:
        st.session_state["protocol"] = os.environ.get("GATEWAY_PROTOCOL", "http")
    if "port" not in st.session_state:
        st.session_state["port"] = os.environ.get("GATEWAY_PORT", "8080")

    with st.sidebar:
        st.title(APP_TITLE)
        st.caption("Solo.io Ambient Mesh + Agent Gateway")

        st.divider()
        st.markdown("**Gateway Configuration**")

        protocol = st.selectbox(
            "Protocol",
            ["http", "https"],
            index=0 if st.session_state["protocol"] == "http" else 1,
        )
        st.session_state["protocol"] = protocol

        port = st.text_input("Port", value=st.session_state["port"])
        st.session_state["port"] = port

        st.text_input(
            "Gateway IP",
            value=st.session_state["gateway_ip"],
            key="sidebar_gw_ip",
            on_change=lambda: st.session_state.update(
                gateway_ip=st.session_state["sidebar_gw_ip"]
            ),
        )

        st.divider()
        st.markdown("**Observability**")
        st.caption("Gloo UI: [{{UI_HOST}}](http://{{UI_HOST}})")
        st.caption("Grafana: [{{GRAFANA_HOST}}](http://{{GRAFANA_HOST}})")
```

- [ ] **Step 2: Create route.yaml.tmpl**

Based on enrollment-agent's `k8s/gateway/route.yaml`. Replace `wgu-enrollment` → `{{ROUTE_NAME}}`.

Write to `templates/k8s/gateway/route.yaml.tmpl`:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: {{ROUTE_NAME}}
  namespace: agentgateway-system
spec:
  parentRefs:
  - name: agentgateway-proxy
    namespace: agentgateway-system
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /openai
    backendRefs:
    - name: openai
      group: agentgateway.dev
      kind: AgentgatewayBackend
    timeouts:
      request: "120s"
```

- [ ] **Step 3: Create install.sh.tmpl**

Based on enrollment-agent's `install.sh` (892 lines). Replace all hardcoded namespace/service references with template variables:
- `wgu-demo` → `{{NS_BACKEND}}`
- `wgu-demo-frontend` → `{{NS_FRONTEND}}`
- `enrollment-chatbot` → `{{CHATBOT_SERVICE_NAME}}`
- `wgu-demo-waypoint` → `{{NS_BACKEND}}-waypoint`
- `Enrollment Agent Demo` → `{{APP_TITLE}} Demo`
- `enroll.glootest.com` → `{{CHATBOT_HOST}}`
- `grafana.glootest.com` → `{{GRAFANA_HOST}}`
- `ui.glootest.com` → `{{UI_HOST}}`

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/install.sh`, make the substitutions above, and write to `templates/install.sh.tmpl`.

Keep all infrastructure code (Istio install, CA generation, multicluster linking, agent gateway, observability) exactly as-is — only namespace names, service names, and branding strings change.

- [ ] **Step 4: Create cleanup.sh.tmpl**

Based on enrollment-agent's `cleanup.sh`. Replace:
- `wgu-demo` → `{{NS_BACKEND}}`
- `wgu-demo-frontend` → `{{NS_FRONTEND}}`
- `Enrollment Agent Demo` → `{{APP_TITLE}} Demo`

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/cleanup.sh`, make substitutions, write to `templates/cleanup.sh.tmpl`.

- [ ] **Step 5: Create build-and-redeploy.sh.tmpl**

Based on enrollment-agent's `build-and-redeploy.sh`. Replace:
- `ably7/enrollment-chatbot` → `{{REGISTRY}}{{IMAGE_PREFIX}}-chatbot`
- `ly-builder` → `{{DOCKER_BUILDER}}`
- `wgu-demo-frontend` → `{{NS_FRONTEND}}`

Read `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/build-and-redeploy.sh`, make substitutions, write to `templates/build-and-redeploy.sh.tmpl`.

- [ ] **Step 6: Verify YAML syntax on route template**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
# Quick check — placeholders will make this invalid YAML, but structure should be sound
cat templates/k8s/gateway/route.yaml.tmpl
```

Visually confirm the YAML structure is correct and only the placeholder values differ from the original.

- [ ] **Step 7: Commit**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
git add templates/demo-ui/utils/sidebar.py.tmpl templates/k8s/gateway/route.yaml.tmpl templates/install.sh.tmpl templates/cleanup.sh.tmpl templates/build-and-redeploy.sh.tmpl
git commit -m "feat: add template files with placeholder variables"
```

---

### Task 5: The Generate-Demo Skill

This is the core deliverable. Write the Claude Code skill that orchestrates the entire demo generation process.

**Files:**
- Create: `.claude/skills/generate-demo.md`

The skill is a markdown file with structured instructions for Claude Code. When invoked, Claude follows these instructions to:
1. Ask the SE 8 questions
2. Copy skeleton files
3. Process template files (substitute variables)
4. Generate domain-specific files from scratch
5. Verify the output

- [ ] **Step 1: Write the skill file**

Write `.claude/skills/generate-demo.md` with the following content. This is the most critical file in the repo — it must be precise, complete, and unambiguous.

```markdown
---
name: generate-demo
description: Generate a complete Solo.io demo repo for any industry vertical. Creates a standalone repo with Streamlit chatbot, mock data services, Istio mesh policies, agent gateway config, and workshop docs — all customized to the target domain.
---

# Generate Demo

You are generating a complete, standalone demo repository for Solo.io's Istio Ambient Mesh + Enterprise Agentgateway stack, customized to a specific industry vertical.

The output is a fully independent repo (like enrollment-agent) with its own install.sh, workshop.md, CLAUDE.md, etc. No runtime dependency on demogen.

## Step 1: Gather Context

Ask these questions ONE AT A TIME. Wait for the answer before asking the next question.

### Question 1: Output Path
> Where should I create the repo? (e.g., `/Users/you/Desktop/solo/solo-github/healthcare-demo`)

If the path exists, write into it (preserve `.git/`, `.gitattributes`, etc.). If it doesn't exist, create it.

### Question 2: Organization
> What organization is this demo for?
> - **Full name** (e.g., "Kaiser Permanente")
> - **Short name** (e.g., "Kaiser")

### Question 3: Vertical / Domain
> What does the chatbot advise on? (e.g., "patient intake and appointment scheduling", "account management and financial planning", "order tracking and product recommendations")

### Question 4: Entity Model
> What is the primary entity and its key fields?
> Example: "Patient — MRN, name, appointments, diagnoses, insurance"
> Example: "Account — account number, holder name, transactions, balance, credit score"

### Question 5: Advisor Role
> What role does the chatbot play? (e.g., "patient intake coordinator", "financial advisor", "customer service representative")

### Question 6: Compliance Regime
> What compliance guardrails should apply?
> - **HIPAA** — MRN/PHI detection (DOB, SSN, medical record numbers)
> - **PCI** — Credit card numbers, CVV, account numbers
> - **FERPA** — Student ID patterns, SSN
> - **Custom** — Provide your own regex patterns and descriptions

### Question 7: Docker Registry
> Where should images be pushed? (e.g., `ably7/`, `gcr.io/solo-demos/`)

### Question 8: Namespace Names
> What should the Kubernetes namespaces be named?
> - **Backend** (e.g., `health-demo`)
> - **Frontend** (e.g., `health-demo-frontend`)

### Question 9: BYO Ext-Authz
> Do you want BYO ext-authz enabled for this demo?
>
> If yes, a generic gRPC ext-authz server (`ably7/grpc-ext-authz:latest`) will be deployed alongside the agent gateway. The chatbot Homepage will include a toggle to apply/remove the ext-authz policy. The default behavior allows requests with the `x-ext-authz: allow` header — you can fork the upstream repo to implement custom logic.

If the SE says yes, ask: "How should the ext-authz be presented in the demo? (Default: header-based allow/deny, or describe custom logic)"

After all questions are answered, confirm the configuration with the SE before proceeding:

> **Configuration Summary:**
> - Output: `{path}`
> - Org: {full_name} ({short_name})
> - Domain: {domain}
> - Entity: {entity_name} with fields: {fields}
> - Role: {advisor_role}
> - Compliance: {regime}
> - Registry: {registry}
> - Namespaces: {ns_backend}, {ns_frontend}
>
> Proceed?

## Step 2: Derive Variables

From the SE's answers, derive these template variables:

| Variable | Rule |
|----------|------|
| `{{ORG_NAME}}` | Full org name from Q2 |
| `{{ORG_SHORT}}` | Short org name from Q2 |
| `{{APP_TITLE}}` | `{ORG_SHORT} {advisor_role title case}` (e.g., "Kaiser Patient Advisor") |
| `{{ADVISOR_ROLE}}` | Advisor role from Q5 |
| `{{DOMAIN}}` | Domain description from Q3 |
| `{{ENTITY_NAME}}` | Entity name from Q4 (e.g., "Patient") |
| `{{ENTITY_NAME_LOWER}}` | Lowercase entity name (e.g., "patient") |
| `{{ENTITY_ID_PREFIX}}` | Derived from org short name, uppercase (e.g., "KP") |
| `{{NS_BACKEND}}` | Backend namespace from Q8 |
| `{{NS_FRONTEND}}` | Frontend namespace from Q8 |
| `{{REGISTRY}}` | Docker registry from Q7 (ensure trailing `/`) |
| `{{IMAGE_PREFIX}}` | Derived from backend namespace (e.g., "health-demo") |
| `{{ROUTE_NAME}}` | Derived from `{org_short_lower}-{entity_lower}` (e.g., "kaiser-patient") |
| `{{CHATBOT_SERVICE_NAME}}` | `{IMAGE_PREFIX}-chatbot` (e.g., "health-demo-chatbot") |
| `{{CHATBOT_HOST}}` | `{entity_lower}.glootest.com` (e.g., "patient.glootest.com"), ask SE to confirm or override |
| `{{GRAFANA_HOST}}` | Default `grafana.glootest.com`, ask SE to confirm or override |
| `{{UI_HOST}}` | Default `ui.glootest.com`, ask SE to confirm or override |
| `{{DOCKER_BUILDER}}` | Default `default`, ask SE if they have a named buildx builder |

## Step 3: Create Output Directory

```bash
mkdir -p {output_path}
```

If a `.git/` directory already exists, leave it. Otherwise:

```bash
cd {output_path} && git init
```

## Step 4: Copy Skeleton Files

Read each file from the demogen `templates/` directory and write it to the output repo. These files are copied EXACTLY as-is with no modifications.

**Important:** The demogen repo is at the current working directory when this skill is invoked. Use relative paths from the demogen root to read template files.

| Source (in demogen) | Destination (in output repo) |
|---------------------|------------------------------|
| `templates/demo-ui/utils/theme.py` | `demo-ui/utils/theme.py` |
| `templates/demo-ui/utils/gateway.py` | `demo-ui/utils/gateway.py` |
| `templates/demo-ui/utils/display.py` | `demo-ui/utils/display.py` |
| `templates/demo-ui/utils/kubectl.py` | `demo-ui/utils/kubectl.py` |
| `templates/demo-ui/Dockerfile` | `demo-ui/Dockerfile` |
| `templates/demo-ui/assets/.gitkeep` | `demo-ui/assets/.gitkeep` |
| `templates/services/data-product-api/Dockerfile` | `services/data-product-api/Dockerfile` |
| `templates/services/data-product-api/requirements.txt` | `services/data-product-api/requirements.txt` |
| `templates/k8s/gateway/backend.yaml` | `k8s/gateway/backend.yaml` |
| `templates/k8s/observability/*` | `k8s/observability/*` (all files) |
| `templates/.gitignore` | `.gitignore` |

## Step 5: Process Template Files

Read each `.tmpl` file from the demogen `templates/` directory, substitute all `{{VARIABLE}}` placeholders with the derived values from Step 2, and write to the output repo WITHOUT the `.tmpl` extension.

| Source | Destination |
|--------|-------------|
| `templates/demo-ui/utils/sidebar.py.tmpl` | `demo-ui/utils/sidebar.py` |
| `templates/k8s/gateway/route.yaml.tmpl` | `k8s/gateway/route.yaml` |
| `templates/install.sh.tmpl` | `install.sh` (make executable: `chmod +x`) |
| `templates/cleanup.sh.tmpl` | `cleanup.sh` (make executable: `chmod +x`) |
| `templates/build-and-redeploy.sh.tmpl` | `build-and-redeploy.sh` (make executable: `chmod +x`) |

After writing each file, verify no `{{` placeholders remain:

```bash
grep -r '{{' {output_path}/install.sh {output_path}/cleanup.sh {output_path}/build-and-redeploy.sh {output_path}/demo-ui/utils/sidebar.py {output_path}/k8s/gateway/route.yaml
```

This should return no output. If any `{{` remains, fix the substitution.

## Step 6: Generate Domain-Specific Files

These files are written from scratch based on the SE's answers. Use the enrollment-agent's patterns as reference but customize all content for the target vertical.

### 6a: `demo-ui/utils/config.py`

Generate a config module following the same pattern as the enrollment-agent's config.py. Must include:

- `ORG_NAME`, `ORG_SHORT`, `APP_TITLE`, `ADVISOR_ROLE` from env vars with defaults matching the SE's answers
- `DATA_PRODUCT_URL` default: `http://data-product-api.{ns_backend}.svc.cluster.local:8080`
- `GRAPH_DB_URL` default: `http://graph-db-mock.{ns_backend}.svc.cluster.local:8081`
- `NS_BACKEND`, `NS_FRONTEND` from env vars
- `STUDENTS` dict renamed to match the entity (e.g., `PATIENTS`, `ACCOUNTS`) with 3 sample entities matching the mock data you'll generate in step 6e
- Entity ID format: `{ENTITY_ID_PREFIX}_2024_NNNNN` (use underscores, NOT dashes — dashes trigger SSN guardrail regex)
- `SYSTEM_PROMPT_TEMPLATE` customized for the domain and advisor role
- `system_prompt(entity_id)` function
- `DEFAULT_{ENTITY}_ID` pointing to the first entity

### 6b: `demo-ui/Homepage.py`

Generate the main Streamlit chatbot page following the enrollment-agent's Homepage.py pattern. Must include:

- Entity selector (sidebar dropdown with the 3 sample entities from config)
- BYO ext-authz toggle (checkbox that applies/removes `EnterpriseAgentgatewayPolicy` for the generic gRPC ext-authz server)
  - Use the route name `{{ROUTE_NAME}}` in the ext-authz policy YAML
  - When enabled, shows how to test with/without the `x-ext-authz: allow` header
  - Only included if SE answered "yes" to ext-authz question (Q9)
- Tool definition for function calling:
  - Function name: `get_{entity_lower}_data` (e.g., `get_patient_data`)
  - Description: domain-appropriate, references the entity and key fields
  - Parameters: `{entity_lower}_id` (string, required)
- `call_data_product_api(entity_id)` function that calls `{DATA_PRODUCT_URL}/{entity_lower_plural}/{entity_id}`
- `process_tool_calls()` function handling the tool call → API call → result flow
- Chat interface with multi-turn conversation support
- Request chain visualization in sidebar (customized for the vertical)
- Valid prompts and guardrail triggers in sidebar (customized for the vertical and compliance regime)
- Import from `utils.config` using the generated variable names

Critical details from the enrollment-agent pattern to preserve:
- `st.set_page_config()` must come BEFORE importing sidebar (Streamlit requirement)
- Blocked messages (guardrail triggers) are NOT saved to `st.session_state["messages"]`
- Tool calls use OpenAI-compatible format with `tool_calls` and `tool_call_id`
- The ext-authz toggle checks cluster state with `run_kubectl` to detect if the policy is active

### 6c: `demo-ui/pages/1_Mesh_Policies.py`

Generate the mesh policies demo page following enrollment-agent's pattern. Must include:

- Current policy state display (both namespaces)
- Live enforcement tests:
  - Allowed: chatbot → data-product-api (health check and entity lookup)
  - Blocked: chatbot → graph-db-mock (direct access denied)
- Policy toggle (zero-trust on/off)
- Policy YAML reference (all policies with domain-appropriate comments)
- Demo walkthrough instructions
- Compliance framing matching the regime (e.g., "HIPAA boundary" instead of "FERPA boundary")

Use the correct namespace names and service principals from the SE's answers.

### 6d: `services/data-product-api/app.py`

Generate the FastAPI data product API. Must include:

- `GET /health` endpoint
- `GET /{entity_lower_plural}/{entity_id}` endpoint (e.g., `/patients/{patient_id}`)
  - Calls graph-db-mock via `POST /query` with Cypher-like syntax
  - Returns entity data or 404
- `GET /{entity_lower_plural}/{entity_id}/{sub_resource}` endpoint
  - Groups sub-items by status (e.g., `/patients/{id}/appointments` groups by upcoming/completed/cancelled)
  - The sub-resource should be the most natural groupable field from the entity model
- `GRAPH_DB_URL` from env var, default: `http://graph-db-mock:8081`
- Standard FastAPI app with `uvicorn.run()` in `__main__`

### 6e: `services/graph-db-mock/app.py`

Generate the mock graph DB service. Must include:

- 3 sample entities with realistic fake data matching the entity model from Q4
- Each entity has 8-12 sub-items with `status` fields (completed/in_progress/not_started or domain-appropriate equivalents)
- Entity IDs use the format `{ENTITY_ID_PREFIX}_YYYY_NNNNN` with underscores (NOT dashes)
- `POST /query` endpoint (accepts `QueryRequest` with `query` and entity ID field)
- `GET /health` endpoint
- Standard FastAPI app on port 8081
- `Dockerfile` and `requirements.txt` already exist as skeleton files

Generate realistic, domain-appropriate data. Examples:
- Healthcare: patients with appointments (upcoming, completed, cancelled), diagnoses, insurance
- Financial: accounts with transactions (pending, completed, failed), balances, credit scores
- Retail: customers with orders (processing, shipped, delivered, returned), loyalty points

### 6f: `k8s/namespaces.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: {ns_backend}
  labels:
    istio.io/dataplane-mode: ambient
---
apiVersion: v1
kind: Namespace
metadata:
  name: {ns_frontend}
  labels:
    istio.io/dataplane-mode: ambient
```

### 6g: `k8s/services/*.yaml`

Generate 3 service manifests following the enrollment-agent's patterns exactly:

**`k8s/services/{chatbot_service_name}.yaml`:**
- ServiceAccount, ClusterRole (mesh policy RBAC), ClusterRoleBinding, Deployment, Service
- Image: `{registry}{image_prefix}-chatbot:0.1.0`
- Env vars: GATEWAY_IP, GATEWAY_PORT, GATEWAY_PROTOCOL, ORG_NAME, ORG_SHORT, APP_TITLE, DATA_PRODUCT_URL (using `.mesh.internal` domain), GRAPH_DB_URL, NS_BACKEND, NS_FRONTEND
- Port 8501, namespace: {ns_frontend}

**`k8s/services/data-product-api.yaml`:**
- ServiceAccount, Deployment, Service
- Image: `{registry}data-product-api:0.0.1`
- GRAPH_DB_URL env var pointing to graph-db-mock in {ns_backend}
- Port 8080, namespace: {ns_backend}
- Label: `solo.io/service-scope: global`, annotation: `networking.istio.io/traffic-distribution: PreferNetwork`

**`k8s/services/graph-db-mock.yaml`:**
- ServiceAccount, Deployment, Service
- Image: `{registry}graph-db-mock:0.0.1`
- Port 8081, namespace: {ns_backend}

### 6h: `k8s/mesh/*.yaml`

Generate mesh authorization policies:

**`k8s/mesh/deny-all.yaml`** — Default deny for both namespaces

**`k8s/mesh/chatbot-to-data-product.yaml`** — L7 allow with targetRefs:
- Target: data-product-api Service in {ns_backend}
- Principal: `*/ns/{ns_frontend}/sa/{chatbot_service_name}`

**`k8s/mesh/data-product-to-graphdb.yaml`** — L7 allow with targetRefs:
- Target: graph-db-mock Service in {ns_backend}
- Principal: `*/ns/{ns_backend}/sa/data-product-api`

**`k8s/mesh/ingress-to-chatbot.yaml`** — Allow ingress to chatbot:
- Principal: `*/ns/agentgateway-system/sa/ingress`
- Port: 8501, namespace: {ns_frontend}

**`k8s/mesh/waypoint-to-backends.yaml`** — L4 allow for waypoint:
- Principal: `*/ns/{ns_backend}/sa/{ns_backend}-waypoint`

**`k8s/mesh/waypoint.yaml`** — Gateway API waypoint + telemetry:
- Gateway name: `{ns_backend}-waypoint`, namespace: {ns_backend}
- Access logging targeting the waypoint

### 6i: `k8s/gateway/guardrails.yaml`

Generate `EnterpriseAgentgatewayPolicy` with compliance-specific guardrails.

Target the HTTPRoute named `{route_name}`.

**Always include:**
- Prompt injection protection (same regex as enrollment-agent)
- Credentials & secrets detection (AWS keys, API keys, private keys)
- Response PII masking (CreditCard, Ssn, PhoneNumber)

**Compliance-specific request rules:**

For **HIPAA**:
- Builtins: CreditCard, Ssn, Email, PhoneNumber
- Custom regex: MRN patterns (e.g., `\bMRN[-_]?\d{6,10}\b`), date of birth patterns
- Response message: "Request blocked: protected health information (PHI) detected. This policy enforces HIPAA compliance."

For **PCI**:
- Builtins: CreditCard, Ssn
- Custom regex: CVV patterns, account number patterns, routing numbers
- Response message: "Request blocked: payment card data detected. This policy enforces PCI-DSS compliance."

For **FERPA**:
- Same as enrollment-agent's guardrails.yaml
- Response message: "Request blocked: personally identifiable information (PII) detected. This policy enforces FERPA compliance."

For **Custom**: Use the SE's provided regex patterns and descriptions.

### 6j: `k8s/gateway/rate-limit.yaml`

Generate token-level rate limiting config. Same structure as enrollment-agent but with the `{route_name}` descriptor value.

### 6k: `k8s/gateway/ingress.yaml`

Copy the enrollment-agent's ingress.yaml exactly — it's generic (no namespace/service references to the demo workloads).

### 6l: `k8s/gateway/ingress-routes.yaml`

Generate HTTPRoutes for the ingress gateway:
- Chatbot route: hostname `{chatbot_host}`, backend `{chatbot_service_name}` port 8501, namespace {ns_frontend}
- Grafana route: hostname `{grafana_host}`, backend `grafana-prometheus` port 3000, namespace monitoring
- Note about UI route being applied dynamically by install.sh

### 6m: `k8s/gateway/ext-authz.yaml` (conditional — only if SE enabled ext-authz)

Generate the generic gRPC ext-authz deployment + service. Uses the `ably7/grpc-ext-authz:latest` image. Deploy to `agentgateway-system` namespace:
- Deployment: 1 replica, port 9000, image `ably7/grpc-ext-authz:latest`
- Service: port 4444 → targetPort 9000, `appProtocol: kubernetes.io/h2c`

This is always deployed by install.sh. The `EnterpriseAgentgatewayPolicy` that wires it to the route is applied/removed dynamically by the Homepage toggle.

### 6m2: `demo-ui/pages/2_Multi_Cluster.py`

Generate the multi-cluster failover demo page following the enrollment-agent's `2_Multi_Cluster.py` pattern. Must include:

- 4-tab flow: Enable Global Service → Simulate Failover → Verify Failover → Restore
- The data-product-api is the failover service
- Tab 1: Label services as global (`solo.io/service-scope=global`) with `PreferNetwork` annotation
- Tab 2: Scale down data-product-api to 0 replicas on the primary cluster
- Tab 3: Test that data-product-api is still reachable (served from cluster2)
- Tab 4: Scale back up, verify local is serving again
- Demo walkthrough instructions
- Domain-framed descriptions using entity name and namespace names from SE answers
- Uses `DATA_PRODUCT_URL`, `NS_BACKEND`, `DEFAULT_{ENTITY}_ID` from config

### 6n: `workshop.md`

Generate a complete workshop document following the enrollment-agent's structure (7 sections + cleanup). Reframe all content for the target vertical:

1. **Prerequisites & Environment Setup** — Generic infrastructure, org name swapped
2. **Istio Ambient Mesh** — Namespace names swapped, domain-framed descriptions
3. **Agent Gateway & Agent Mesh** — Generic, compliance regime swapped
4. **Security & Governance** — Domain-framed policy descriptions (e.g., "HIPAA boundary" not "FERPA boundary")
5. **BYO External Authorization (ext-authz)** — Generic gRPC ext-authz pattern (if enabled)
6. **The Home Run** — Domain-appropriate end-to-end scenario
7. **Without Solo — The AWS-Native Alternative** — Generic comparison
8. **Cleanup**

Each section should have hands-on commands using the correct namespace names, service names, and hostnames. Include leadership callouts.

### 6o: `CLAUDE.md`

Generate a CLAUDE.md for the output repo following the enrollment-agent's CLAUDE.md structure. Include:

- What This Is (with the domain-specific description)
- Architecture diagram (ASCII, using correct service/namespace names)
- Project Structure
- Quick Start
- Building and Deploying
- Key Design Decisions (same mesh/gateway patterns, domain-specific ID format)
- Branding / Configuration (env var table)
- Adding New Features (pages, entities, policies, guardrails)
- Testing commands
- Source repos referenced (remove if not applicable)

## Step 7: Verification

After generating all files:

1. **List all files:**
```bash
find {output_path} -type f | grep -v '.git/' | sort
```

Show the output to the SE.

2. **Check for leftover placeholders:**
```bash
grep -r '{{' {output_path}/ --include='*.py' --include='*.yaml' --include='*.sh' --include='*.md' | grep -v '.git/'
```

Should return no output.

3. **Python syntax check:**
```bash
cd {output_path}
python3 -c "import py_compile; py_compile.compile('demo-ui/Homepage.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/utils/config.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/pages/1_Mesh_Policies.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/data-product-api/app.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/graph-db-mock/app.py', doraise=True)"
```

4. **Print next steps:**
> Demo repo generated at `{output_path}`.
>
> **Next steps:**
> 1. `cd {output_path}`
> 2. Review the key files: `Homepage.py`, `config.py`, `graph-db-mock/app.py`, `workshop.md`
> 3. Build container images (see `build-and-redeploy.sh` or CLAUDE.md)
> 4. Run `./install.sh` to deploy
> 5. Add `/etc/hosts` entries as shown by the install script
```

- [ ] **Step 2: Verify the skill file is well-formed**

Read back `.claude/skills/generate-demo.md` and verify:
- Frontmatter has `name` and `description`
- All 8 questions are present
- All template variable derivations are documented
- All skeleton file copy instructions are complete
- All template file substitution instructions are complete
- All generated file instructions reference the correct patterns
- Verification steps are included

- [ ] **Step 3: Commit**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
git add .claude/skills/generate-demo.md
git commit -m "feat: add generate-demo skill"
```

---

### Task 6: Demogen CLAUDE.md and README.md

**Files:**
- Create: `CLAUDE.md`
- Create: `README.md`

- [ ] **Step 1: Write CLAUDE.md**

Write `CLAUDE.md` at the demogen repo root:

```markdown
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

The skill asks 8 questions about the target vertical and generates a complete repo.

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
```

- [ ] **Step 2: Write README.md**

Write `README.md` at the demogen repo root:

```markdown
# Demogen

AI-powered demo generator for Solo.io. Generates complete, standalone demo repos for Istio Ambient Mesh + Enterprise Agentgateway, customized to any industry vertical.

## Quick Start

1. Clone this repo
2. Open in Claude Code
3. Run `/generate-demo` (or ask Claude to generate a demo)
4. Answer 8 questions about your target vertical
5. Get a complete repo — ready to `./install.sh`

## What Gets Generated

A standalone repo containing:
- **Streamlit chatbot** with AI function calling, BYO ext-authz toggle, guardrail demos
- **Mock data service** with realistic domain-specific fake data
- **Data product API** secured through the mesh with mTLS
- **Kubernetes manifests** for Istio ambient mesh policies, agent gateway config, observability
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
```

- [ ] **Step 3: Commit**

```bash
cd /Users/alexly-solo/Desktop/solo/solo-github/demogen
git add CLAUDE.md README.md
git commit -m "docs: add CLAUDE.md and README.md"
```

---

### Task 7: Validation — Generate a Test Demo

Validate the entire system by invoking the skill to generate a healthcare demo.

**This task is manual / interactive.** The SE (you) will:

- [ ] **Step 1: Invoke the skill**

From the demogen repo directory, tell Claude Code:

> Generate a healthcare patient intake demo

The skill should ask 8 questions. Answer with:
1. Output path: `/tmp/healthcare-demo` (temporary, for testing)
2. Organization: "Mercy Health System", "Mercy"
3. Domain: "patient intake and appointment scheduling"
4. Entity: "Patient — MRN, name, appointments, diagnoses, insurance, primary physician"
5. Role: "patient intake coordinator"
6. Compliance: HIPAA
7. Registry: `ably7/`
8. Namespaces: `health-demo`, `health-demo-frontend`

- [ ] **Step 2: Verify generated output**

After generation completes, check:

```bash
# All expected files exist
find /tmp/healthcare-demo -type f | grep -v '.git/' | sort

# No leftover placeholders
grep -r '{{' /tmp/healthcare-demo/ --include='*.py' --include='*.yaml' --include='*.sh' --include='*.md'

# Python files compile
cd /tmp/healthcare-demo
python3 -c "import py_compile; py_compile.compile('demo-ui/Homepage.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/utils/config.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/graph-db-mock/app.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/data-product-api/app.py', doraise=True)"

# Namespace names are correct in mesh policies
grep -r 'health-demo' /tmp/healthcare-demo/k8s/mesh/ | head -20

# Guardrails reference HIPAA
grep -i 'hipaa\|PHI\|protected health' /tmp/healthcare-demo/k8s/gateway/guardrails.yaml

# Entity IDs use underscores (not dashes)
grep -E '[A-Z]+_[0-9]+_[0-9]+' /tmp/healthcare-demo/services/graph-db-mock/app.py
```

- [ ] **Step 3: Review key generated files**

Manually review:
- `demo-ui/Homepage.py` — tool definition uses `get_patient_data`, ext-authz toggle works
- `demo-ui/utils/config.py` — defaults are Mercy-branded, entity list has 3 patients
- `demo-ui/pages/1_Mesh_Policies.py` — correct namespace names, HIPAA framing
- `demo-ui/pages/2_Multi_Cluster.py` — failover demo uses data-product-api, correct namespaces
- `services/graph-db-mock/app.py` — 3 realistic patients with appointments and diagnoses
- `k8s/gateway/guardrails.yaml` — HIPAA-specific PII patterns
- `k8s/gateway/ext-authz.yaml` — generic gRPC ext-authz deployment (if ext-authz enabled)
- `workshop.md` — healthcare-framed narrative, correct namespace names throughout, multi-cluster section included
- `CLAUDE.md` — accurate architecture description for the generated repo

- [ ] **Step 4: Clean up**

```bash
rm -rf /tmp/healthcare-demo
```
