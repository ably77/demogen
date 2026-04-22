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
> Example: "Patient -- MRN, name, appointments, diagnoses, insurance"
> Example: "Account -- account number, holder name, transactions, balance, credit score"

### Question 5: Advisor Role
> What role does the chatbot play? (e.g., "patient intake coordinator", "financial advisor", "customer service representative")

### Question 6: Compliance Regime
> What compliance guardrails should apply?
> - **HIPAA** -- MRN/PHI detection (DOB, SSN, medical record numbers)
> - **PCI** -- Credit card numbers, CVV, account numbers
> - **FERPA** -- Student ID patterns, SSN
> - **Custom** -- Provide your own regex patterns and descriptions

### Question 7: Docker Registry
> Where should images be pushed? (e.g., `ably7/`, `gcr.io/solo-demos/`)

### Question 8: Namespace Names
> What should the Kubernetes namespaces be named?
> - **Backend** (e.g., `health-demo`)
> - **Frontend** (e.g., `health-demo-frontend`)

### Question 9: BYO Ext-Authz
> Do you want BYO ext-authz enabled for this demo?
>
> If yes, a generic gRPC ext-authz server (`ably7/grpc-ext-authz:latest`) will be deployed alongside the agent gateway. The chatbot Homepage will include a toggle to apply/remove the ext-authz policy. The default behavior allows requests with the `x-ext-authz: allow` header -- you can fork the upstream repo to implement custom logic.

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
> - Ext-authz: {yes/no} ({description if yes})
>
> Proceed?

## Step 2: Derive Variables

From the SE's answers, derive these template variables:

| Variable | Rule |
|----------|------|
| `{{ORG_NAME}}` | Full org name from Q2 |
| `{{ORG_SHORT}}` | Short org name from Q2 |
| `{{APP_TITLE}}` | `{ORG_SHORT} {advisor_role title case}` (e.g., "Kaiser Patient Advisor") |
| `{{ADVISOR_ROLE}}` | Advisor role from Q5 (lowercase) |
| `{{DOMAIN}}` | Domain description from Q3 |
| `{{ENTITY_NAME}}` | Entity name from Q4 in title case (e.g., "Patient") |
| `{{ENTITY_NAME_LOWER}}` | Lowercase entity name (e.g., "patient") |
| `{{ENTITY_NAME_PLURAL}}` | Plural lowercase entity name (e.g., "patients") |
| `{{ENTITY_ID_PREFIX}}` | Derived from org short name, uppercase, typically 2-4 chars (e.g., "KP" for Kaiser Permanente, "MH" for Mercy Health) |
| `{{NS_BACKEND}}` | Backend namespace from Q8 |
| `{{NS_FRONTEND}}` | Frontend namespace from Q8 |
| `{{REGISTRY}}` | Docker registry from Q7 (ensure trailing `/`) |
| `{{IMAGE_PREFIX}}` | Same as backend namespace value (e.g., "health-demo") |
| `{{ROUTE_NAME}}` | `{org_short_lower}-{entity_lower}` (e.g., "kaiser-patient") |
| `{{CHATBOT_SERVICE_NAME}}` | `{IMAGE_PREFIX}-chatbot` (e.g., "health-demo-chatbot") |
| `{{CHATBOT_HOST}}` | `{entity_lower}.glootest.com` (e.g., "patient.glootest.com") -- ask SE to confirm or override |
| `{{GRAFANA_HOST}}` | Default `grafana.glootest.com` -- ask SE to confirm or override |
| `{{UI_HOST}}` | Default `ui.glootest.com` -- ask SE to confirm or override |
| `{{DOCKER_BUILDER}}` | Default `default` -- ask SE if they have a named buildx builder |

Show the derived variables to the SE for confirmation before continuing.

## Step 3: Create Output Directory

```bash
mkdir -p {output_path}
```

If a `.git/` directory already exists, leave it. Otherwise:

```bash
cd {output_path} && git init
```

Create all required subdirectories:

```bash
mkdir -p {output_path}/demo-ui/utils
mkdir -p {output_path}/demo-ui/pages
mkdir -p {output_path}/demo-ui/assets
mkdir -p {output_path}/services/data-product-api
mkdir -p {output_path}/services/graph-db-mock
mkdir -p {output_path}/k8s/services
mkdir -p {output_path}/k8s/mesh
mkdir -p {output_path}/k8s/gateway
mkdir -p {output_path}/k8s/observability
```

## Step 4: Copy Skeleton Files

Read each file from the demogen `templates/` directory and write it to the output repo. These files are copied EXACTLY as-is with no modifications.

**Important:** The demogen repo root is at `/Users/alexly-solo/Desktop/solo/solo-github/demogen`. Use absolute paths to read template files.

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
| `templates/k8s/observability/*` | `k8s/observability/*` (all files in that directory) |
| `templates/.gitignore` | `.gitignore` |

## Step 5: Process Template Files

Read each `.tmpl` file from the demogen `templates/` directory, substitute ALL `{{VARIABLE}}` placeholders with the derived values from Step 2, and write to the output repo WITHOUT the `.tmpl` extension.

| Source | Destination |
|--------|-------------|
| `templates/demo-ui/utils/sidebar.py.tmpl` | `demo-ui/utils/sidebar.py` |
| `templates/k8s/gateway/route.yaml.tmpl` | `k8s/gateway/route.yaml` |
| `templates/install.sh.tmpl` | `install.sh` (make executable: `chmod +x`) |
| `templates/cleanup.sh.tmpl` | `cleanup.sh` (make executable: `chmod +x`) |
| `templates/build-and-redeploy.sh.tmpl` | `build-and-redeploy.sh` (make executable: `chmod +x`) |

After writing each file, verify no `{{` placeholders remain:

```bash
grep -rn '{{' {output_path}/install.sh {output_path}/cleanup.sh {output_path}/build-and-redeploy.sh {output_path}/demo-ui/utils/sidebar.py {output_path}/k8s/gateway/route.yaml
```

This should return no output. If any `{{` remains, fix the substitution.

## Step 6: Generate Domain-Specific Files

These files are written from scratch based on the SE's answers. Use the enrollment-agent's patterns as reference but customize all content for the target vertical.

**CRITICAL RULES for all generated files:**
- Entity IDs use UNDERSCORES not dashes. Dashes trigger the SSN guardrail regex. Format: `{ENTITY_ID_PREFIX}_YYYY_NNNNN` (e.g., `KP_2024_00142`).
- `st.set_page_config()` MUST come BEFORE any other Streamlit import or call (including `from utils.sidebar import render_sidebar`). This is a hard Streamlit requirement.
- Blocked messages (guardrail triggers returning non-200) are NOT saved to `st.session_state["messages"]`. Otherwise the PII content replays on every subsequent request.
- Tool calls use OpenAI-compatible format with `tool_calls` array and `tool_call_id` fields.
- Waypoint + AuthorizationPolicy: With a namespace waypoint, L7 policies MUST use `targetRefs` (pointing to the Service), not `selector`. The waypoint preserves original caller identity at L7 but presents its own identity at L4.
- The agentgateway-system namespace is NOT enrolled in the ambient mesh.

---

### 6a: `demo-ui/utils/config.py`

Generate a config module following the enrollment-agent's `demo-ui/utils/config.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/utils/config.py`.

Must include:

- `ORG_NAME`, `ORG_SHORT`, `APP_TITLE`, `ADVISOR_ROLE` from env vars with defaults matching the SE's answers
- `DATA_PRODUCT_URL` default: `http://data-product-api.{ns_backend}.svc.cluster.local:8080`
- `GRAPH_DB_URL` default: `http://graph-db-mock.{ns_backend}.svc.cluster.local:8081`
- `NS_BACKEND`, `NS_FRONTEND` from env vars
- Entity dict named after the entity in uppercase plural (e.g., `PATIENTS`, `ACCOUNTS`, `CUSTOMERS`). Contains 3 sample entities as `{"id": "...", "label": "..."}` dicts. Entity IDs MUST use underscores: `{ENTITY_ID_PREFIX}_YYYY_NNNNN`.
- `DEFAULT_{ENTITY_UPPER}_ID` pointing to the first entity (e.g., `DEFAULT_PATIENT_ID`)
- `SYSTEM_PROMPT_TEMPLATE` customized for the domain and advisor role, with `{entity_lower}_id` as the template variable. The prompt should describe the chatbot's role, what data it can look up, and how to use the tool.
- `system_prompt(entity_id)` function that formats the template

Example for healthcare:
```python
PATIENTS = {s["id"]: s["label"] for s in _patients_list}
DEFAULT_PATIENT_ID = list(PATIENTS.keys())[0]

SYSTEM_PROMPT_TEMPLATE = os.environ.get("SYSTEM_PROMPT", f"""\
You are a helpful {ADVISOR_ROLE} for {ORG_NAME}. \
You help patients understand their appointments, diagnoses, and care plans.

When a patient asks about their records, appointments, or health data, use the get_patient_data tool \
to look up their information. The current patient ID is {{patient_id}}.

Be professional, empathetic, and specific. Reference actual appointment dates and diagnoses when discussing care. \
If the patient asks about upcoming appointments, identify which ones are scheduled from the data.""")
```

---

### 6b: `demo-ui/Homepage.py`

Generate the main Streamlit chatbot page following the enrollment-agent's `Homepage.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/Homepage.py`.

**Structure (must follow this exact order):**

```python
"""
{APP_TITLE} -- AI Chatbot Demo
Demonstrates the full chain: chatbot -> agent gateway -> LLM -> data product API -> graph DB
All secured and governed through Solo's Ambient Mesh + Agent Gateway.
"""

import json
import requests
import streamlit as st

from utils.config import (
    APP_TITLE, ORG_SHORT, DATA_PRODUCT_URL,
    {ENTITY_PLURAL_UPPER}, DEFAULT_{ENTITY_UPPER}_ID, system_prompt,
)

# CRITICAL: st.set_page_config() MUST come before any other st.* calls or imports that use st
st.set_page_config(
    page_title=APP_TITLE,
    page_icon="{domain_appropriate_material_icon}",
    layout="wide",
)

# Now safe to import sidebar and other utils
from utils.sidebar import render_sidebar
from utils.gateway import chat_completion
from utils.display import render_tool_call, render_error
from utils.kubectl import run_kubectl

render_sidebar()
```

**Must include these sections:**

1. **Entity selector** (sidebar dropdown) -- select from the 3 sample entities defined in config.py. Clear chat on entity change.

2. **Ext-authz toggle** (ONLY if SE enabled ext-authz in Q9):
   - Sidebar checkbox that applies/removes `EnterpriseAgentgatewayPolicy` for the generic gRPC ext-authz
   - Check cluster state with `run_kubectl` to detect if the policy is currently active
   - When enabled, show explanatory text about how to test with/without the `x-ext-authz: allow` header
   - The policy YAML uses the route name from `{{ROUTE_NAME}}` and references `ext-authz` service on port 4444
   - Include `_enable_extauth()` and `_disable_extauth()` helper functions
   - Pattern to follow for the EnterpriseAgentgatewayPolicy:
   ```python
   def _enable_extauth():
       run_kubectl(
           'kubectl apply -f - <<EOF\n'
           'apiVersion: enterpriseagentgateway.solo.io/v1alpha1\n'
           'kind: EnterpriseAgentgatewayPolicy\n'
           'metadata:\n'
           '  namespace: agentgateway-system\n'
           '  name: ext-auth-policy\n'
           'spec:\n'
           '  targetRefs:\n'
           '  - group: gateway.networking.k8s.io\n'
           '    kind: HTTPRoute\n'
           '    name: {ROUTE_NAME}\n'
           '  traffic:\n'
           '    extAuth:\n'
           '      backendRef:\n'
           '        name: ext-authz\n'
           '        namespace: agentgateway-system\n'
           '        port: 4444\n'
           '      grpc: {}\n'
           'EOF'
       )
   ```
   - When enabled, show info about testing: "Send `x-ext-authz: allow` header to allow, omit to deny"

3. **Tool definition** for function calling:
   ```python
   TOOLS = [
       {
           "type": "function",
           "function": {
               "name": "get_{entity_lower}_data",
               "description": "Retrieve {entity_lower} data including {describe key fields from Q4}.",
               "parameters": {
                   "type": "object",
                   "properties": {
                       "{entity_lower}_id": {
                           "type": "string",
                           "description": f"The {entity_lower} ID (e.g., {DEFAULT_{ENTITY_UPPER}_ID})",
                       }
                   },
                   "required": ["{entity_lower}_id"],
               },
           },
       }
   ]
   ```

4. **`call_data_product_api(entity_id)`** function:
   - GET request to `{DATA_PRODUCT_URL}/{entity_lower_plural}/{entity_id}`
   - 10 second timeout, handle errors

5. **`process_tool_calls(tool_calls, messages)`** function:
   - Same pattern as enrollment-agent -- iterate tool calls, call API, render results, append messages
   - Use `render_tool_call(fn_name, fn_args, result)` for display
   - Return updated messages list

6. **Chat interface**:
   - Session state for messages
   - Render chat history (user + assistant messages with content)
   - Chat input with domain-appropriate placeholder (e.g., "Ask about your appointments, diagnoses, or care plan...")
   - Build API messages: system prompt + history + current prompt
   - Call `chat_completion(api_messages, model=model, tools=TOOLS, extra_headers=extra_headers)`
   - Handle tool calls (first call triggers tool, second call gets final response)
   - **CRITICAL**: Only save messages to session state on success (200 status). Do NOT save on guardrail blocks.
   - Model defaults to "gpt-4o-mini" (or uses ext-authz model selection if enabled)

7. **Sidebar: Request chain visualization**:
   ```python
   with st.sidebar:
       st.divider()
       st.markdown("**Request Chain**")
       st.caption("{Entity} :material/arrow_forward: Chatbot")
       st.caption("  :material/arrow_forward: Agent Gateway (guardrails, tokens)")
       st.caption("  :material/arrow_forward: LLM Provider")
       st.caption("  :material/arrow_forward: Data Product API (via mesh)")
       st.caption("  :material/arrow_forward: Graph DB ({db_type} mock)")
   ```

8. **Sidebar: Valid prompts and guardrail triggers** (domain-appropriate):
   ```python
   st.divider()
   st.markdown("**Valid Prompts** (will trigger {entity_lower} data lookup)")
   st.markdown(
       "{list 4-5 domain-appropriate prompts that trigger tool calls}"
   )
   st.markdown("**Guardrail Triggers** (blocked by the agent gateway)")
   st.markdown(
       "{list compliance-appropriate guardrail examples with expected status codes}"
   )
   ```
   - For HIPAA: SSN, credit card, email examples (422) and prompt injection (403)
   - For PCI: credit card, CVV, account number examples (422) and prompt injection (403)
   - For FERPA: SSN, credit card, email examples (422) and prompt injection (403)

---

### 6c: `demo-ui/pages/1_Mesh_Policies.py`

Generate the mesh policies demo page following the enrollment-agent's `pages/1_Mesh_Policies.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/pages/1_Mesh_Policies.py`.

Must include these sections:

1. **Page config** (set_page_config FIRST, then imports):
   ```python
   st.set_page_config(
       page_title="Mesh Policies",
       page_icon=":material/shield:",
       layout="wide",
   )
   ```

2. **YAML string constants** for display -- define the full YAML for each policy with domain-appropriate comments:
   - `DENY_ALL_YAML` -- deny-all for both namespaces
   - `CHATBOT_TO_DATA_YAML` -- chatbot SA to data-product-api using targetRefs
   - `DATA_TO_GRAPH_YAML` -- data-product-api SA to graph-db-mock using targetRefs
   - `WAYPOINT_TO_BACKENDS_YAML` -- waypoint SA to backends (L4)
   - Use correct namespace names, service account names, and service names from the derived variables

3. **Architecture diagram placeholder**: If an `assets/mesh-architecture.html` file exists, include it. Otherwise skip this section. Do NOT error if the file is missing.

4. **Section 1: Current Policy State** -- two-column display for both namespaces, showing `kubectl get authorizationpolicies` output

5. **Section 2: Live Enforcement Tests**:
   - **Allowed paths** column:
     - Chatbot -> Data Product API health check
     - Chatbot -> Entity data lookup (using DEFAULT entity ID from config)
   - **Blocked paths** column:
     - Chatbot -> Graph DB direct health check (expect 403 or connection refused)
     - Chatbot -> Graph DB query (expect 403 or connection refused)
   - Frame the blocked paths with the compliance regime (e.g., "This is the HIPAA boundary" instead of "FERPA boundary")

6. **Section 3: Policy Toggle**:
   - Check current zero-trust state by counting policies
   - "Remove ALL policies (open access)" button
   - "Restore zero-trust (all policies)" button -- applies from `/app/k8s/mesh/` directory (the Dockerfile copies k8s/ into the image)
   - Show last action results with YAML

7. **Section 4: Policy YAML Reference** -- expandable sections for each policy

8. **Section 5: Demo Walkthrough** -- domain-framed instructions for the recommended demo flow. Reference the compliance regime. Frame the mesh policies as the compliance boundary.

---

### 6c2: `demo-ui/pages/2_Multi_Cluster.py`

Generate the multi-cluster failover demo page following the enrollment-agent's `pages/2_Multi_Cluster.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/demo-ui/pages/2_Multi_Cluster.py`.

Must include:

1. **Page config** (set_page_config FIRST):
   ```python
   st.set_page_config(
       page_title="Multi-Cluster",
       page_icon=":material/language:",
       layout="wide",
   )
   ```

2. **Imports** from utils: theme, kubectl, config (DATA_PRODUCT_URL, NS_BACKEND, DEFAULT entity ID)

3. **Architecture diagram placeholder**: If an `assets/multicluster-failover-architecture.html` file exists, include it. Otherwise skip.

4. **Global service label YAML** constant for display:
   ```python
   GLOBAL_LABEL_YAML = """\
   # Label a service as globally available across the mesh
   apiVersion: v1
   kind: Service
   metadata:
     name: data-product-api
     namespace: {ns_backend}
     labels:
       solo.io/service-scope: global
     annotations:
       networking.istio.io/traffic-distribution: PreferNetwork
   """
   ```

5. **4-tab flow**:
   - **Tab 1: Enable Global Service** -- Label data-product-api and graph-db-mock with `solo.io/service-scope=global` and `PreferNetwork` annotation. Show explainer of what the labels do. Include "Check ServiceEntries" button. Show current deployments.
   - **Tab 2: Simulate Failover** -- Scale data-product-api to 0 replicas. Show warning about scaled state.
   - **Tab 3: Verify Failover** -- Test health check and entity data lookup through the API. Show success/failure with domain-appropriate messages (e.g., "served from cluster2!"). Include link back to Homepage for end-to-end test.
   - **Tab 4: Restore** -- Scale data-product-api back to 1 replica. Wait for rollout. Verify health.

6. **Demo Walkthrough** section with recommended flow instructions.

All descriptions should be domain-framed using the entity name and namespace names from the SE's answers. The data-product-api is always the failover service.

---

### 6d: `services/data-product-api/app.py`

Generate the FastAPI data product API following the enrollment-agent's `services/data-product-api/app.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/services/data-product-api/app.py`.

Must include:

- `GET /health` endpoint returning `{"status": "healthy"}`
- `GET /{entity_lower_plural}/{entity_id}` endpoint (e.g., `/patients/{patient_id}`):
  - Calls graph-db-mock via `POST /query` with a Cypher-like query string and entity ID
  - Returns entity data or raises HTTPException(404)
- `GET /{entity_lower_plural}/{entity_id}/{sub_resource}` endpoint:
  - Same graph-db-mock call, then groups sub-items by status
  - Sub-resource should be the most natural groupable field from the entity model (e.g., `/patients/{id}/appointments`, `/accounts/{id}/transactions`)
  - Groups by status values appropriate for the domain (e.g., scheduled/completed/cancelled for appointments, pending/completed/failed for transactions)
- `GRAPH_DB_URL` from env var, default: `http://graph-db-mock:8081`
- Title: `"{Entity} Data Product API"`
- Standard FastAPI app with `uvicorn.run(app, host="0.0.0.0", port=8080)` in `__main__`

---

### 6e: `services/graph-db-mock/app.py` + `Dockerfile` + `requirements.txt`

Generate the mock graph DB service following the enrollment-agent's `services/graph-db-mock/app.py` pattern. Reference the enrollment-agent file at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/services/graph-db-mock/app.py`.

**`app.py`** must include:

- A dict of 3 sample entities with realistic fake data matching the entity model from Q4
- Each entity has 8-12 sub-items with `status` fields (domain-appropriate status values)
- Entity IDs use the format `{ENTITY_ID_PREFIX}_YYYY_NNNNN` with UNDERSCORES (NOT dashes)
- Each entity has a name, the entity-specific fields from Q4, and the sub-items list
- `QueryRequest` pydantic model with `query: str` and `{entity_lower}_id: str`
- `POST /query` endpoint that looks up entity by ID, returns 404 if not found
- `GET /health` endpoint returning `{"status": "healthy"}`
- Title: `"Graph DB Mock ({db_type} Simulator)"`
- Standard FastAPI app with `uvicorn.run(app, host="0.0.0.0", port=8081)` in `__main__`

Generate realistic, domain-appropriate data. Examples by vertical:
- **Healthcare**: patients with appointments (scheduled/completed/cancelled), diagnoses, insurance info, primary physician
- **Financial**: accounts with transactions (pending/completed/failed), balances, credit scores, account types
- **Education**: students with courses (completed/in_progress/not_started), GPA, program, competency units
- **Retail**: customers with orders (processing/shipped/delivered/returned), loyalty points, preferences

**`Dockerfile`** (write to `services/graph-db-mock/Dockerfile`):
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
EXPOSE 8081
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8081"]
```

**`requirements.txt`** (write to `services/graph-db-mock/requirements.txt`):
```
fastapi
uvicorn[standard]
pytest
httpx
```

---

### 6f: `k8s/namespaces.yaml`

Generate namespace definitions with ambient mesh labels:

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

---

### 6g: `k8s/services/*.yaml`

Generate 3 service manifests following the enrollment-agent's patterns. Reference the enrollment-agent files at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/services/`.

**`k8s/services/{chatbot_service_name}.yaml`:**
- ServiceAccount (name: `{chatbot_service_name}`, namespace: `{ns_frontend}`)
- ClusterRole with rules for:
  - `security.istio.io` / `authorizationpolicies`: get, list, create, update, patch, delete
  - Core / `pods`: get, list
  - Core / `services`: get, list, patch
  - `apps` / `deployments`: get, list, patch
  - `apps` / `deployments/scale`: get, update, patch
  - `networking.istio.io` / `serviceentries`: get, list
  - `enterpriseagentgateway.solo.io` / `enterpriseagentgatewaypolicies`: get, list, create, apply, patch, delete
- ClusterRoleBinding linking the SA to the ClusterRole
- Deployment:
  - Image: `{registry}{image_prefix}-chatbot:0.1.0`
  - Port: 8501
  - Env vars: GATEWAY_IP (`agentgateway-proxy.agentgateway-system.svc.cluster.local`), GATEWAY_PORT (`8080`), GATEWAY_PROTOCOL (`http`), ORG_NAME, ORG_SHORT, APP_TITLE, DATA_PRODUCT_URL (`http://data-product-api.{ns_backend}.mesh.internal:8080`), GRAPH_DB_URL (`http://graph-db-mock.{ns_backend}.svc.cluster.local:8081`), NS_BACKEND, NS_FRONTEND
  - Resources: requests cpu 100m/memory 256Mi, limits cpu 500m/memory 512Mi
- Service: ClusterIP, port 8501

**`k8s/services/data-product-api.yaml`:**
- ServiceAccount (name: `data-product-api`, namespace: `{ns_backend}`)
- Deployment:
  - Image: `{registry}data-product-api:0.0.1`
  - Port: 8080
  - GRAPH_DB_URL env var: `http://graph-db-mock.{ns_backend}.svc.cluster.local:8081`
  - Readiness probe: GET /health port 8080
  - Resources: requests cpu 100m/memory 128Mi, limits cpu 200m/memory 256Mi
- Service:
  - Port 8080
  - Labels include `solo.io/service-scope: global`
  - Annotations include `networking.istio.io/traffic-distribution: PreferNetwork`

**`k8s/services/graph-db-mock.yaml`:**
- ServiceAccount (name: `graph-db-mock`, namespace: `{ns_backend}`)
- Deployment:
  - Image: `{registry}graph-db-mock:0.0.1`
  - Port: 8081
  - Readiness probe: GET /health port 8081
  - Resources: requests cpu 100m/memory 128Mi, limits cpu 200m/memory 256Mi
- Service: port 8081

---

### 6h: `k8s/mesh/*.yaml`

Generate mesh authorization policies following the enrollment-agent's patterns. Reference files at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/mesh/`.

**`k8s/mesh/deny-all.yaml`:**
Default deny for both namespaces. Comment references the compliance regime (e.g., "HIPAA boundary" instead of "FERPA boundary").
```yaml
# Deny-all baseline: no service can talk to anything unless explicitly allowed.
# This is the {COMPLIANCE} boundary -- {entity_lower} data is locked down by default.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: {ns_backend}
spec:
  {}
---
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: deny-all
  namespace: {ns_frontend}
spec:
  {}
```

**`k8s/mesh/chatbot-to-data-product.yaml`:**
L7 allow with targetRefs for chatbot to call data-product-api.
```yaml
# The {chatbot_service_name} can call the data product API
# to fetch {entity_lower} data when the LLM requests it via function calling.
# Uses targetRefs (L7 enforcement at the waypoint) to preserve original caller identity.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: chatbot-to-data-product
  namespace: {ns_backend}
spec:
  targetRefs:
  - kind: Service
    group: ""
    name: data-product-api
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "*/ns/{ns_frontend}/sa/{chatbot_service_name}"
```

**`k8s/mesh/data-product-to-graphdb.yaml`:**
L7 allow with targetRefs for data-product-api to query graph-db-mock.
```yaml
# Only the data product API can query the graph database.
# This is the innermost {COMPLIANCE} boundary -- raw {entity_lower} graph data
# is only accessible to the authorized data product service.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: data-product-to-graphdb
  namespace: {ns_backend}
spec:
  targetRefs:
  - kind: Service
    group: ""
    name: graph-db-mock
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "*/ns/{ns_backend}/sa/data-product-api"
```

**`k8s/mesh/ingress-to-chatbot.yaml`:**
Allow ingress gateway to reach chatbot.
```yaml
# Allow the ingress gateway to reach the chatbot.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: ingress-to-chatbot
  namespace: {ns_frontend}
spec:
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "*/ns/agentgateway-system/sa/ingress"
    to:
    - operation:
        ports: ["8501"]
```

**`k8s/mesh/waypoint-to-backends.yaml`:**
L4 allow for waypoint proxy. Uses the `{ns_backend}-waypoint` service account.
```yaml
# Allow the waypoint proxy to reach backend services at L4.
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy
metadata:
  name: waypoint-to-backends
  namespace: {ns_backend}
spec:
  action: ALLOW
  rules:
  - from:
    - source:
        principals:
        - "*/ns/{ns_backend}/sa/{ns_backend}-waypoint"
```

**`k8s/mesh/waypoint.yaml`:**
Gateway API waypoint + access logging telemetry.
```yaml
# Waypoint proxy for the {ns_backend} namespace.
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: {ns_backend}-waypoint
  namespace: {ns_backend}
spec:
  gatewayClassName: istio-waypoint
  listeners:
  - name: mesh
    port: 15008
    protocol: HBONE
    allowedRoutes:
      namespaces:
        from: All
---
apiVersion: telemetry.istio.io/v1
kind: Telemetry
metadata:
  name: {ns_backend}-access-logging
  namespace: {ns_backend}
spec:
  targetRefs:
  - kind: Gateway
    group: gateway.networking.k8s.io
    name: {ns_backend}-waypoint
  accessLogging:
  - providers:
    - name: envoy
```

---

### 6i: `k8s/gateway/guardrails.yaml`

Generate `EnterpriseAgentgatewayPolicy` with compliance-specific guardrails. Reference the enrollment-agent's guardrails at `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/gateway/guardrails.yaml`.

Target the HTTPRoute named `{route_name}`.

**Always include these sections:**

1. **PII Detection** (compliance-specific):
   - Regex action: Reject
   - Builtins and custom patterns vary by compliance regime (see below)
   - Response message references the compliance framework
   - Status code: 422

2. **Prompt Injection Protection** (same for all regimes):
   ```yaml
   - regex:
       action: Reject
       matches:
       - "(?i)(ignore|disregard|forget|override|bypass)\\s+(all\\s+|any\\s+|your\\s+)?(previous|prior|earlier|above|existing)\\s+(instructions|rules|guidelines|directives)"
       - "(?i)(you are now|from now on you are|henceforth you are)\\s+(a |an |the )?(unrestricted|unfiltered|uncensored|DAN)"
       - "(?i)(do anything now|DAN mode|enable DAN|activate DAN)"
     response:
       message: "Request blocked: prompt injection attempt detected."
       statusCode: 403
   ```

3. **Credentials & Secrets** (same for all regimes):
   ```yaml
   - regex:
       action: Reject
       matches:
       - "\\bAKIA[0-9A-Z]{16}\\b"
       - "\\bsk-[a-zA-Z0-9_-]{20,}\\b"
       - "-----BEGIN\\s+(RSA\\s+|EC\\s+)?PRIVATE KEY-----"
       - "(?i)(password|secret|token|api[_-]?key)\\s*[=:]\\s*[\"']?[^\\s\"']{8,}"
     response:
       message: "Request blocked: credential or secret detected in prompt."
       statusCode: 422
   ```

4. **Response PII masking** (same for all regimes):
   ```yaml
   response:
   - regex:
       action: Mask
       builtins:
       - CreditCard
       - Ssn
       - PhoneNumber
     response:
       message: "Response filtered: PII redacted from model output."
   ```

**Compliance-specific PII detection:**

For **HIPAA**:
```yaml
- regex:
    action: Reject
    builtins:
    - CreditCard
    - Ssn
    - Email
    - PhoneNumber
    matches:
    - "\\bMRN[-_]?\\d{6,10}\\b"
    - "(?i)\\b(date of birth|dob)\\s*[:=]?\\s*\\d{1,2}[/-]\\d{1,2}[/-]\\d{2,4}\\b"
  response:
    message: "Request blocked: protected health information (PHI) detected. Do not include SSNs, MRNs, DOBs, or other PHI in prompts. This policy enforces HIPAA compliance."
    statusCode: 422
```

For **PCI**:
```yaml
- regex:
    action: Reject
    builtins:
    - CreditCard
    - Ssn
    matches:
    - "\\b\\d{3,4}\\s*$"
    - "(?i)\\b(routing|aba)\\s*(number|#)?\\s*[:=]?\\s*\\d{9}\\b"
    - "(?i)\\baccount\\s*(number|#|no\\.?)\\s*[:=]?\\s*\\d{8,17}\\b"
  response:
    message: "Request blocked: payment card or financial data detected. Do not include credit card numbers, CVVs, or account numbers in prompts. This policy enforces PCI-DSS compliance."
    statusCode: 422
```

For **FERPA**:
```yaml
- regex:
    action: Reject
    builtins:
    - CreditCard
    - Ssn
    - Email
    - PhoneNumber
  response:
    message: "Request blocked: personally identifiable information (PII) detected. Do not include SSNs, credit cards, emails, or phone numbers in prompts. This policy enforces FERPA compliance."
    statusCode: 422
```

For **Custom**: Use the SE's provided regex patterns and descriptions.

---

### 6j: `k8s/gateway/rate-limit.yaml`

Generate token-level rate limiting config following the enrollment-agent's pattern. Reference `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/gateway/rate-limit.yaml`.

Use `{route_name}` as the descriptor value and in the resource names. Structure:

```yaml
# Token-level rate limiting -- 100,000 tokens per hour per user.
apiVersion: ratelimit.solo.io/v1alpha1
kind: RateLimitConfig
metadata:
  name: {route_name}-token-limit
  namespace: agentgateway-system
spec:
  raw:
    descriptors:
    - key: generic_key
      value: {route_name}
      rateLimit:
        requestsPerUnit: 100000
        unit: HOUR
    rateLimits:
    - actions:
      - genericKey:
          descriptorValue: {route_name}
      type: TOKEN
---
apiVersion: enterpriseagentgateway.solo.io/v1alpha1
kind: EnterpriseAgentgatewayPolicy
metadata:
  name: {route_name}-rate-limit
  namespace: agentgateway-system
spec:
  targetRefs:
  - name: agentgateway-proxy
    group: gateway.networking.k8s.io
    kind: Gateway
  traffic:
    entRateLimit:
      global:
        rateLimitConfigRefs:
        - name: {route_name}-token-limit
```

---

### 6k: `k8s/gateway/ingress.yaml`

Generate the ingress gateway config. This is generic infrastructure with no demo-specific references. Copy the pattern exactly from the enrollment-agent. Reference `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/gateway/ingress.yaml`.

```yaml
# Ingress gateway -- exposes user-facing services via hostname-based routing.
# Enrolled in the ambient mesh for mTLS on the ingress-to-service hop.
---
apiVersion: enterpriseagentgateway.solo.io/v1alpha1
kind: EnterpriseAgentgatewayParameters
metadata:
  name: ingress-agentgateway-config
  namespace: agentgateway-system
spec:
  logging:
    level: info
  service:
    metadata:
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    spec:
      type: LoadBalancer
  deployment:
    spec:
      replicas: 2
      template:
        metadata:
          labels:
            istio.io/dataplane-mode: ambient
        spec:
          containers:
          - name: agentgateway
            resources:
              requests:
                cpu: 50m
                memory: 64Mi
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ingress
  namespace: agentgateway-system
spec:
  gatewayClassName: enterprise-agentgateway
  infrastructure:
    parametersRef:
      group: enterpriseagentgateway.solo.io
      kind: EnterpriseAgentgatewayParameters
      name: ingress-agentgateway-config
  listeners:
  - allowedRoutes:
      namespaces:
        from: All
    name: http
    port: 80
    protocol: HTTP
```

---

### 6l: `k8s/gateway/ingress-routes.yaml`

Generate HTTPRoutes for the ingress gateway:

```yaml
# HTTPRoutes for the ingress gateway -- hostname-based routing to user-facing services.
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: chatbot-ingress-route
  namespace: {ns_frontend}
spec:
  hostnames:
  - "{chatbot_host}"
  parentRefs:
  - name: ingress
    namespace: agentgateway-system
  rules:
  - backendRefs:
    - name: {chatbot_service_name}
      port: 8501
    matches:
    - path:
        type: PathPrefix
        value: /
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: grafana-ingress-route
  namespace: monitoring
spec:
  hostnames:
  - "{grafana_host}"
  parentRefs:
  - name: ingress
    namespace: agentgateway-system
  rules:
  - backendRefs:
    - name: grafana-prometheus
      port: 3000
    matches:
    - path:
        type: PathPrefix
        value: /
# NOTE: The ui-ingress-route for solo-enterprise-ui is applied dynamically
# by install.sh because the service may be in agentgateway-system or kagent.
```

---

### 6m: `k8s/gateway/ext-authz.yaml` (conditional -- ONLY if SE enabled ext-authz in Q9)

Generate the generic gRPC ext-authz deployment + service. Reference `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/k8s/gateway/abac-ext-authz.yaml` for structure, but use the generic `ably7/grpc-ext-authz:latest` image.

```yaml
# Generic gRPC ext-authz server for the agent gateway.
# Deployment + Service are deployed by install.sh (always running).
# The EnterpriseAgentgatewayPolicy is applied/removed dynamically
# by the Homepage ext-authz toggle.
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: agentgateway-system
  name: ext-authz
  labels:
    app: ext-authz
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ext-authz
  template:
    metadata:
      labels:
        app: ext-authz
        app.kubernetes.io/name: ext-authz
    spec:
      containers:
      - image: ably7/grpc-ext-authz:latest
        imagePullPolicy: Always
        name: ext-authz
        ports:
        - containerPort: 9000
        env:
        - name: PORT
          value: "9000"
---
apiVersion: v1
kind: Service
metadata:
  namespace: agentgateway-system
  name: ext-authz
  labels:
    app: ext-authz
spec:
  ports:
  - port: 4444
    targetPort: 9000
    protocol: TCP
    appProtocol: kubernetes.io/h2c
  selector:
    app: ext-authz
```

Also add `kubectl apply -f k8s/gateway/ext-authz.yaml` to the `deploy_workloads()` function in the already-processed `install.sh` if ext-authz is enabled. Since install.sh was already processed from the template in Step 5, you will need to edit it to add the ext-authz apply command in the gateway section (after the line that applies `k8s/gateway/rate-limit.yaml`). Add:
```bash
  kubectl apply -f k8s/gateway/ext-authz.yaml --context $ctx
  kubectl rollout status deploy/ext-authz -n agentgateway-system --watch --timeout=60s --context $ctx
```

---

### 6n: `workshop.md`

Generate a complete workshop document following the enrollment-agent's `workshop.md` structure. Reference `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/workshop.md`.

The workshop should have 7 sections plus cleanup, reframed for the target vertical:

1. **Prerequisites & Environment Setup**
   - Cluster setup (2 clusters), CLI tools, license/API keys
   - Build container images commands using the generated registry/image names
   - Verification checkpoint

2. **Istio Ambient Mesh**
   - Install on both clusters with shared root CA
   - Deploy demo workloads (using correct namespace and service names)
   - Verify mTLS enrollment
   - Multi-cluster connectivity (linking clusters)
   - Deploy waypoint for L7 traffic management
   - AuthorizationPolicy -- Zero Trust (using the compliance regime framing)

3. **Agent Gateway & Agent Mesh**
   - Install Enterprise Agentgateway
   - Observability stack (Prometheus + Grafana + Solo UI)
   - Configure LLM backend (OpenAI)
   - Add guardrails (compliance-specific)
   - Token-level rate limiting
   - Observability (access logs, tracing)

4. **Security & Governance (Deep Dive)**
   - Centralized policy view
   - RBAC for service-to-service (domain-framed)
   - Audit logging and compliance reporting
   - Policy enforcement demonstration

5. **BYO External Authorization (ext-authz)** (if enabled)
   - Overview of generic gRPC ext-authz pattern
   - Deploy the ext-authz server
   - Verify the route works without ext-authz
   - Apply the ext-authz policy
   - Test enforcement (with and without `x-ext-authz: allow` header)
   - View ext-authz logs
   - Cleanup

6. **The Home Run -- End-to-End Scenario**
   - Deploy the chatbot
   - Verify mesh enrollment
   - Deploy ingress gateway
   - Open the chatbot UI
   - Run the demo scenario (domain-appropriate prompts and guardrail tests)
   - Observe the full chain (traces in Gloo UI, metrics in Grafana)
   - The complete picture (summary of all layers)

7. **Without Solo -- The AWS-Native Alternative**
   - Side-by-side comparison: what you'd need to build with AWS services vs Solo
   - Cost and complexity comparison

Each section should include:
- Hands-on commands with correct namespace names, service names, hostnames
- Leadership callouts (business context for non-technical audiences)
- Verification steps

---

### 6o: `CLAUDE.md`

Generate a CLAUDE.md for the output repo following the enrollment-agent's CLAUDE.md structure. Reference `/Users/alexly-solo/Desktop/solo/solo-github/enrollment-agent/CLAUDE.md`.

Must include:

- **What This Is** -- Domain-specific description of the demo
- **Architecture** -- ASCII diagram with correct service/namespace names:
  ```
  {Entity} User
    -> Ingress Gateway (agentgateway-system, LoadBalancer:80, in mesh)
      -> {APP_TITLE} Chatbot (Streamlit, {ns_frontend} namespace)
        -> Agent Gateway (agentgateway-system, ClusterIP:8080) -> LLM Provider (OpenAI)
        -> Data Product API ({ns_backend} namespace, through mesh with mTLS)
          -> Graph DB Mock ({ns_backend} namespace, through waypoint)
    -> Grafana (monitoring namespace)
    -> Gloo UI (agentgateway-system namespace)
  ```
- **Project Structure** -- File tree with descriptions
- **Quick Start** -- Prerequisites, install, /etc/hosts, access URLs
- **Building and Deploying** -- build-and-redeploy.sh usage, Docker build context note
- **Key Design Decisions**:
  - Entity ID format (underscores not dashes, explain why)
  - Waypoint + AuthorizationPolicy (targetRefs, L7 vs L4)
  - Agentgateway NOT in ambient mesh (explain why)
  - Gateway API CRDs (experimental channel, server-side apply)
  - Agentgateway version (v2.3.0+)
  - Blocked messages not saved to chat history
- **Branding / Configuration** -- env var table
- **Adding New Features** -- pages, entities, policies, guardrails, MCP
- **Testing** -- pytest commands for data-product-api and graph-db-mock

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

Should return no output. If any `{{` remains, fix it.

3. **Python syntax check:**
```bash
cd {output_path}
python3 -c "import py_compile; py_compile.compile('demo-ui/Homepage.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/utils/config.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/pages/1_Mesh_Policies.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('demo-ui/pages/2_Multi_Cluster.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/data-product-api/app.py', doraise=True)"
python3 -c "import py_compile; py_compile.compile('services/graph-db-mock/app.py', doraise=True)"
```

All should pass with no errors.

4. **Print next steps:**

> Demo repo generated at `{output_path}`.
>
> **Next steps:**
> 1. `cd {output_path}`
> 2. Review the key files: `Homepage.py`, `config.py`, `graph-db-mock/app.py`, `workshop.md`
> 3. Build container images (see `build-and-redeploy.sh` or CLAUDE.md)
> 4. Run `./install.sh` to deploy
> 5. Add `/etc/hosts` entries as shown by the install script
