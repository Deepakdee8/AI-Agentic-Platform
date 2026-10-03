# Product Requirements Document (PRD)

## AI-Native GCP Infrastructure Provisioning Platform

**Document Version:** 1.1  
**Status:** Draft  
**Product Type:** AI-native cloud infrastructure control plane  
**Initial Cloud Provider:** Google Cloud Platform (GCP)  
**Primary IaC Engine:** Terraform

---

# 1. Executive Summary

The product is an **AI-native infrastructure provisioning platform for Google Cloud Platform** that allows users to create, modify, understand, and manage cloud infrastructure using natural language.

Instead of requiring users to manually write Terraform, understand every GCP dependency, navigate multiple GCP consoles, and execute infrastructure changes themselves, users describe their desired infrastructure in natural language.

For example:

> "Create a GKE cluster with 3 nodes in us-central1, a PostgreSQL Cloud SQL instance, and a VPC."

The platform converts this request into a structured infrastructure intent, resolves requirements and dependencies, generates an infrastructure blueprint, estimates the expected monthly cost, validates security and organizational policies, and presents the proposed architecture to a human for approval.

After approval, the platform generates Terraform using approved infrastructure modules, validates the Terraform configuration, generates and displays the Terraform plan, executes the provisioning workflow, continuously reports progress, verifies the resulting GCP resources, and records the complete request and infrastructure history.

The platform therefore acts as an **AI control plane over deterministic cloud infrastructure execution**.

---

# 2. Product Vision

> Make cloud infrastructure provisioning as simple as describing the desired outcome in natural language, while maintaining the safety, governance, reproducibility, auditability, and control expected from enterprise infrastructure platforms.

The long-term vision is to allow users to say:

> "Create the infrastructure required to run my application in production."

and have the platform:

1. Understand the requirements.
2. Identify missing information.
3. Design the architecture.
4. Explain its decisions.
5. Estimate cost.
6. Validate security and organizational policies.
7. Ask for human approval.
8. Generate infrastructure-as-code.
9. Validate and plan the changes.
10. Provision the infrastructure.
11. Verify the resulting environment.
12. Maintain an ongoing understanding of the infrastructure.

---

# 3. Product Mission

The platform should reduce the complexity and time required to provision GCP infrastructure while **never sacrificing infrastructure governance and human control**.

The platform should provide:

- Natural-language infrastructure provisioning
- AI-assisted architecture design
- Infrastructure blueprints
- Cost estimation
- Policy and security validation
- Human-in-the-loop approval
- Terraform generation
- Terraform plan/apply execution
- Real-time provisioning status
- Resource inventory
- Complete audit history
- Infrastructure drift detection
- Natural-language infrastructure modifications

---

# 4. Problem Statement

Provisioning cloud infrastructure today typically requires users to understand:

- GCP services
- networking
- IAM
- security
- Terraform
- resource dependencies
- cloud pricing
- regional limitations
- organizational policies
- Terraform state
- deployment pipelines
- operational troubleshooting

A simple requirement can involve many resources.

For example:

> "Create GKE with three nodes and Cloud SQL."

may actually require:

```text
VPC
 ├── Subnet
 │
 ├── Secondary ranges
 │
 └── Routing

GKE
 ├── Cluster
 ├── Node pool
 ├── IAM
 └── Networking

Cloud SQL
 ├── Instance
 ├── Network connectivity
 ├── Database
 ├── User
 └── Backup configuration
```

The user should not need to manually understand or construct all of these components.

---

# 5. Product Principles

## 5.1 AI is not the infrastructure authority

The LLM must never directly control production infrastructure.

The architecture must be:

```text
Natural Language
       ↓
AI
       ↓
Structured Blueprint
       ↓
Policy Validation
       ↓
Human Approval
       ↓
Terraform
       ↓
Terraform Plan
       ↓
Execution
       ↓
GCP
```

---

## 5.2 Blueprint is the central product abstraction

The platform should not treat Terraform as the primary representation of user intent.

The primary representation should be a platform-owned:

> **Infrastructure Blueprint**

Terraform is generated from the approved blueprint.

---

## 5.3 Desired state, approved state, and actual state must be separate

The platform should track:

```text
Desired State
      ↓
Approved State
      ↓
Terraform State
      ↓
Actual GCP State
```

This allows the platform to identify:

- user-request changes
- approval changes
- Terraform implementation differences
- infrastructure drift

---

## 5.4 Human control is mandatory for consequential changes

The platform should support configurable approval policies.

Examples:

```text
Development
→ automatic execution allowed

Staging
→ approval required

Production
→ architecture approval + execution approval
```

---

## 5.5 Deterministic systems enforce safety

Security, IAM, policy, budgets, Terraform validation, state locking, and execution permissions must not depend solely on LLM reasoning.

---

# 6. Target Users

## 6.1 Cloud/Platform Engineers

Needs:

- faster infrastructure provisioning
- standardized architectures
- reusable modules
- governance
- visibility
- automation

## 6.2 DevOps Engineers

Needs:

- reduced Terraform boilerplate
- faster environment creation
- infrastructure troubleshooting
- drift detection

## 6.3 Developers

Needs:

> "Give me the infrastructure I need to run my application."

without becoming GCP/Terraform experts.

## 6.4 Infrastructure Architects

Needs:

- architecture review
- standardization
- dependency visibility
- cost visibility
- policy enforcement

## 6.5 Engineering Managers / Platform Teams

Needs:

- infrastructure governance
- cost control
- auditability
- standardized deployments
- operational visibility

---

# 7. Primary User Journey

## Example

User enters:

> "Provision a GKE cluster with 3 nodes in us-central1, a Cloud SQL PostgreSQL database, and a VPC."

The platform executes:

```text
1. Request received
2. Intent extracted
3. Requirements resolved
4. Architecture designed
5. Blueprint generated
6. Cost estimated
7. Security/policy validated
8. Human approval
9. Terraform generated
10. Terraform validated/planned
11. Terraform applied
12. Resources verified
```

Final result:

```text
Provisioning completed successfully.

Created resources:

VPC
platform-vpc

Subnet
gke-subnet

GKE
platform-gke

Node Pool
default-pool

Cloud SQL
platform-db

Request ID:
REQ-1234
```

---

# 8. Product Scope

## MVP Scope

The first version should support:

### GCP services

- VPC
- Subnets
- GKE
- GKE node pools
- Cloud SQL
- IAM required for supported resources
- Secret Manager integration where required

### Core capabilities

- Natural-language requests
- Intent extraction
- Infrastructure blueprint
- Architecture visualization
- Cost estimation
- Policy validation
- Human approval
- Blueprint modification
- Terraform generation
- Terraform validation
- Terraform plan
- Terraform apply
- Real-time execution status
- Resource verification
- Request history
- Audit trail

---

# 9. Out of Scope for MVP

The following should not be required for the first release:

- Multi-cloud provisioning
- AWS/Azure support
- Autonomous production remediation
- Fully autonomous infrastructure changes
- Hundreds of GCP services
- Complex application deployment
- Kubernetes application deployment
- Advanced FinOps optimization
- Automated migration from existing infrastructure
- Full enterprise ITSM integration
- Self-healing production infrastructure

These can be future capabilities.

---

# 10. High-Level System Architecture

```text
                         ┌───────────────────┐
                         │       User        │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │     Chainlit UI   │
                         │ Chat + Blueprint   │
                         └─────────┬─────────┘
                                   │
                          SSE / WebSockets
                                   │
                                   ▼
                         ┌───────────────────┐
                         │    FastAPI API    │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Request Manager   │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ LangGraph Runtime │
                         │ Workflow / Agents │
                         └─────────┬─────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼
       AI/Agent Layer      Knowledge Layer       Policy Layer
              │                    │                    │
              ▼                    ▼                    ▼
       Intent/Planner       Graph + RAG          Policy Engine
              │
              ▼
       Blueprint Engine
              │
              ▼
       Cost Estimation
              │
              ▼
          HITL
              │
              ▼
      Terraform Engine
              │
              ▼
      Execution Engine
              │
              ▼
             GCP
              │
              ▼
      Verification Engine
              │
              ▼
      Resource Inventory
              │
              ▼
          PostgreSQL
```

---

# 11. Technology Stack

The initial implementation will use the following technology stack.

| Layer | Technology | Purpose |
|---|---|---|
| Frontend / UI | Chainlit | Conversational interface, blueprint/status presentation, HITL |
| Real-time communication | SSE / WebSockets | Streaming agent output and provisioning status |
| Backend | Python | Primary application and AI orchestration language |
| API | FastAPI | REST APIs, request handling, authentication integration |
| AI orchestration | LangGraph | Stateful agent/workflow orchestration |
| Data validation | Pydantic | Intent, Blueprint, event, API, and tool schemas |
| Database | PostgreSQL | Requests, blueprints, executions, resources, audit data |
| Infrastructure as Code | Terraform | Deterministic infrastructure provisioning |
| Cloud Provider | GCP | Initial infrastructure provider |
| Eventing | GCP Pub/Sub | Asynchronous workflow and provisioning events |
| Object storage | Google Cloud Storage | Terraform artifacts, plans, logs, and large artifacts |
| Secrets | Secret Manager | Secure secret storage and references |
| Policy | OPA or equivalent | Deterministic policy and governance enforcement |
| Observability | OpenTelemetry + GCP observability | Logs, metrics, traces |
| Terraform execution | Isolated worker/job environment | Secure Terraform execution |

### Technology decisions

The product will use **Chainlit** rather than a custom React/Next.js frontend for the initial MVP. Chainlit will provide the conversational user experience and HITL interaction surface.

The backend will be implemented in **Python with FastAPI**.

**LangGraph** will be used for stateful AI-agent orchestration. LangGraph should orchestrate reasoning and workflow state, but infrastructure execution remains outside the LLM's direct control.

**Pydantic** will define the contracts for:

- Intent
- Blueprint
- Blueprint versions
- Agent outputs
- Tool inputs/outputs
- Workflow state
- Events
- API requests/responses

**PostgreSQL** will be the system-of-record database.

---

# 12. 12-Stage Provisioning Pipeline

The platform must implement the provisioning workflow as a durable state machine.

## Stage 1 — Request Ingestion

Input:

```text
Natural language request
```

Output:

```text
request_id
conversation_id
request status
```

Example:

```json
{
  "request_id": "REQ-1234",
  "status": "RECEIVED"
}
```

---

## Stage 2 — Intent Extraction

The Intent Agent converts natural language into structured intent.

Example:

```json
{
  "resources": [
    {
      "type": "gke",
      "node_count": 3
    },
    {
      "type": "cloud_sql",
      "engine": "postgres"
    },
    {
      "type": "vpc"
    }
  ],
  "constraints": {
    "region": "us-central1"
  }
}
```

Requirements:

- identify resources
- identify quantities
- identify regions
- identify relationships
- identify explicit constraints
- identify ambiguity

---

## Stage 3 — Requirement Resolution

The system determines what information is missing.

Inputs:

```text
User Intent
+
Organization Defaults
+
Platform Defaults
+
Environment Context
+
Knowledge Graph
+
Policies
```

If a critical decision cannot safely be inferred, the system asks the user.

Example:

> "Should this GKE cluster use Standard or Autopilot mode?"

---

## Stage 4 — Architecture Planning

The Architecture Agent generates a logical infrastructure architecture.

Example:

```text
VPC
 │
 └── GKE Subnet
       │
       └── GKE Cluster
             │
             └── Node Pool

VPC
 │
 └── Private Service Connectivity
       │
       └── Cloud SQL
```

Output:

- Architecture Graph
- Dependencies
- Assumptions

---

## Stage 5 — Blueprint Generation

The Blueprint Engine generates the canonical infrastructure blueprint.

Example:

```yaml
apiVersion: infra.platform/v1

kind: Environment

metadata:
  name: application-platform

spec:

  network:
    type: vpc
    name: platform-vpc

  region:
    name: us-central1

  gke:
    mode: standard
    name: platform-gke

    nodePools:
      - name: default
        count: 3
        machineType: e2-standard-4

  cloudSQL:
    engine: postgres
    version: "16"
    name: platform-db
```

The blueprint must conform to a strict Pydantic/schema definition.

---

## Stage 6 — Cost Estimation

The Cost Service calculates the estimated monthly cost.

Output:

```text
Estimated monthly cost

GKE
$XXX

Cloud SQL
$XXX

Networking
$XX

Total
$XXX/month
```

The system must display assumptions:

- Region
- Machine type
- Storage
- Expected usage
- Pricing model
- Estimated monthly hours

The platform must distinguish:

```text
Estimated cost
```

from:

```text
Actual GCP billing
```

---

## Stage 7 — Policy and Security Validation

The policy engine validates the blueprint.

Example policies:

```text
Production:
- public IP prohibited
- CMEK required
- approved regions only
- deletion protection required
- approved machine types only
- mandatory labels
- maximum monthly budget
```

Policy results:

```text
PASS
WARN
REQUIRES_APPROVAL
DENY
```

Example:

```text
❌ DENIED

Reason:
Cloud SQL public IP is prohibited in production.
```

---

## Stage 8 — Human-in-the-Loop Approval

The platform displays:

### Architecture

Visual infrastructure graph.

### Blueprint

Structured infrastructure specification.

### Cost

Estimated monthly cost.

### Assumptions

Values inferred by the platform.

### Policies

Security and governance checks.

### Changes

For blueprint revisions.

The user can:

```text
Approve
Modify
Reject
```

---

## Natural-Language Blueprint Modification

Example:

> "Change the GKE nodes to e2-standard-8 and increase them to 5."

The platform creates:

```text
Blueprint v1
      ↓
Natural Language Change
      ↓
Blueprint v2
      ↓
Validation
      ↓
Cost recalculation
      ↓
Human approval
```

Blueprint versions must be immutable.

---

## Stage 9 — Terraform Generation

After approval:

```text
Approved Blueprint
        ↓
Module Resolver
        ↓
Terraform Generator
```

The system must prefer approved Terraform modules over arbitrary code generation.

Generated Terraform must be associated with:

```text
request_id
blueprint_version
module_versions
terraform_version
provider_version
```

---

## Stage 10 — Terraform Validation and Planning

The platform runs:

```text
terraform fmt
terraform init
terraform validate
TFLint
security scanning
policy validation
terraform plan
```

The user should see:

```text
Terraform Plan

5 resources to create
0 resources to modify
0 resources to destroy
```

Where applicable, the platform should expose a human-readable change summary.

---

## Stage 11 — Terraform Apply / Provisioning

Execution occurs in an isolated execution environment.

The API server must not execute Terraform directly.

Conceptually:

```text
Control Plane
      ↓
Execution Job
      ↓
Terraform
      ↓
GCP
```

The execution service must support:

- retries
- timeout handling
- state locking
- cancellation
- logs
- partial failure
- execution history

---

## Stage 12 — Verification and Inventory

After Terraform completes, the platform verifies actual GCP resources.

Example:

```text
GKE
✓ Exists
✓ RUNNING
✓ us-central1
✓ 3 nodes

Cloud SQL
✓ Exists
✓ RUNNABLE
✓ PostgreSQL 16

VPC
✓ Exists
✓ Correct subnet
```

The platform then creates/updates the resource inventory.

---

# 13. Request State Machine

The request lifecycle must be represented explicitly.

```text
RECEIVED
   ↓
UNDERSTANDING
   ↓
REQUIREMENTS_RESOLUTION
   ↓
ARCHITECTURE_PLANNING
   ↓
BLUEPRINT_GENERATED
   ↓
COST_ESTIMATED
   ↓
POLICY_VALIDATED
   ↓
WAITING_FOR_APPROVAL
   ↓
APPROVED
   ↓
TERRAFORM_GENERATING
   ↓
TERRAFORM_VALIDATING
   ↓
TERRAFORM_PLANNING
   ↓
PROVISIONING
   ↓
VERIFYING
   ↓
COMPLETED
```

Failure states:

```text
FAILED
CANCELLED
WAITING_FOR_USER
REQUIRES_REMEDIATION
PARTIALLY_PROVISIONED
```

---

# 14. LangGraph Agent Architecture

LangGraph will provide the primary AI orchestration layer.

The graph should separate reasoning nodes from deterministic workflow nodes.

Conceptually:

```text
                         START
                           │
                           ▼
                  ┌─────────────────┐
                  │ Intent Agent    │
                  └────────┬────────┘
                           │
                           ▼
                  ┌─────────────────┐
                  │ Requirement     │
                  │ Resolver        │
                  └────────┬────────┘
                           │
                   Missing information?
                      /            \
                    YES             NO
                    │                │
                    ▼                ▼
                HITL Question   Architecture
                    │                │
                    └───────┬────────┘
                            ▼
                   ┌─────────────────┐
                   │ Blueprint Agent │
                   └────────┬────────┘
                            │
                            ▼
                     Cost Service
                            │
                            ▼
                     Policy Engine
                            │
                            ▼
                      HITL Approval
                            │
                     ┌──────┴──────┐
                     │             │
                  Modify         Approve
                     │             │
                     ▼             ▼
                Blueprint      Terraform
                  Update        Generation
                                   │
                                   ▼
                              Validation
                                   │
                                   ▼
                                Plan
                                   │
                                   ▼
                                Apply
                                   │
                                   ▼
                              Verify
                                   │
                                   ▼
                               Complete
```

LangGraph state should contain only the information required to execute the current workflow and should reference persisted database objects for durable state.

---

# 15. AI Agent Architecture

The MVP should use a small number of specialized agents.

## Intent Agent

Responsibilities:

- natural language understanding
- resource extraction
- constraints
- ambiguity

## Architecture Agent

Responsibilities:

- architecture design
- dependencies
- resource relationships
- module selection suggestions

## Blueprint Agent

Responsibilities:

- convert architecture into platform blueprint
- modify existing blueprints
- explain assumptions

## Terraform Agent

Responsibilities:

- select Terraform modules
- generate Terraform configuration
- explain Terraform changes

The actual Terraform execution remains deterministic.

## Operations Agent

Responsibilities:

- analyze provisioning failures
- explain errors
- identify possible remediation
- suggest blueprint changes

The Operations Agent must not autonomously modify production unless an explicit automation policy allows it.

---

# 16. Skills Architecture

Skills represent reusable capabilities.

Examples:

```text
discover-gcp-project
discover-gcp-region
create-vpc-blueprint
create-gke-blueprint
create-cloudsql-blueprint
estimate-gcp-cost
validate-security-policy
resolve-terraform-module
generate-terraform
terraform-plan
terraform-apply
verify-gke
verify-cloudsql
inventory-resource
```

Each skill should have:

```text
name
version
description
input schema
output schema
permissions
execution policy
```

Skills should be implemented as deterministic or tightly constrained functions wherever possible.

---

# 17. Connector Architecture

Connectors provide external-system integrations.

MVP:

```text
GCP
Terraform
```

Future:

```text
GitHub
GitLab
Slack
Jira
ServiceNow
PagerDuty
Datadog
```

Example:

```text
GCP Connector
 ├── Resource Manager
 ├── IAM
 ├── Compute
 ├── GKE
 ├── Cloud SQL
 ├── Billing
 └── Monitoring
```

---

# 18. Knowledge Platform

The platform should combine:

```text
Knowledge Graph
+
RAG / Vector Search
+
Structured Metadata
```

## Knowledge Graph

Represents:

```text
resources
dependencies
compatibility
regions
services
network relationships
Terraform modules
security requirements
policies
```

Example:

```text
GKE
 ├── requires → VPC
 ├── uses → subnet
 ├── supports → Standard
 ├── supports → Autopilot
 └── available_in → us-central1
```

## RAG Knowledge

RAG should contain:

- GCP documentation
- internal architecture standards
- Terraform module documentation
- security guidelines
- organizational runbooks
- operational procedures
- approved architecture patterns

---

# 19. Terraform Module Registry

The platform must maintain a curated registry of reusable Terraform modules.

Example:

```text
gcp-vpc
gke-standard
gke-autopilot
cloudsql-postgres
cloudsql-mysql
cloud-storage
redis
load-balancer
artifact-registry
```

Each module should contain:

```text
name
version
inputs
outputs
supported regions
dependencies
security properties
cost model
documentation
policy constraints
```

---

# 20. Infrastructure Blueprint Specification

The blueprint is the platform's canonical infrastructure language.

It should contain:

```text
metadata
project
environment
region
network
compute
database
storage
security
IAM
observability
dependencies
policies
cost
assumptions
```

The blueprint must be:

- versioned
- schema validated
- immutable after approval
- machine readable
- human readable
- Terraform-independent

Pydantic models will provide the primary application-level validation layer.

---

# 21. Cost Model

The cost service should expose:

```text
estimated_monthly_cost
estimated_daily_cost
resource_costs
pricing_timestamp
pricing_source
assumptions
confidence
```

Example:

```json
{
  "estimated_monthly_cost": 742.50,
  "currency": "USD",
  "assumptions": [
    "730 hours/month",
    "us-central1",
    "3 GKE nodes",
    "100 GB Cloud SQL storage"
  ]
}
```

---

# 22. Security Requirements

Security is a core product requirement.

## Principle

The AI must not receive unrestricted GCP privileges.

Use separate identities for:

```text
AI planning
Resource discovery
Terraform plan
Terraform apply
Production execution
```

---

# 23. IAM

The platform must implement least-privilege access.

Users should not automatically receive project-owner privileges because they use the platform.

The platform should support:

```text
Viewer
Developer
Platform Engineer
Approver
Administrator
```

with configurable permissions.

---

# 24. Secret Management

Secrets must never be included directly in LLM prompts or blueprint content.

The platform should reference secrets:

```text
secret_ref:
  projects/.../secrets/db-password
```

rather than storing plaintext credentials.

---

# 25. Terraform State

Terraform state must be stored in a secure remote backend.

Requirements:

- encryption
- versioning
- access control
- locking
- auditability
- environment isolation

State must never be stored on the application server's local filesystem as the authoritative state.

---

# 26. Environment Locking

Concurrent infrastructure changes must be controlled.

Example:

```text
Project: production

Current lock:
REQ-1234

Status:
Terraform Apply
```

A second request should become:

```text
QUEUED
```

rather than concurrently modifying the same Terraform state.

---

# 27. Real-Time Progress

The Chainlit UI must receive streaming execution events through **SSE and/or WebSockets**.

Example event:

```json
{
  "request_id": "REQ-1234",
  "stage": "PROVISIONING",
  "resource": "gke_cluster",
  "status": "CREATING",
  "timestamp": "..."
}
```

The UI should display:

```text
✓ VPC created
✓ Subnet created
✓ GKE cluster created
◉ Creating node pool
○ Cloud SQL
○ Verification
```

### Communication requirements

SSE may be used for:

- one-way streaming from backend to UI
- agent reasoning/status events
- Terraform logs
- provisioning progress

WebSockets may be used where bidirectional real-time communication is required.

The implementation should avoid maintaining two parallel event systems. Both SSE and WebSocket endpoints should consume the same backend event stream.

---

# 28. Event Architecture

Important events include:

```text
RequestCreated
IntentExtracted
RequirementsResolved
BlueprintCreated
BlueprintModified
CostCalculated
PolicyValidated
ApprovalRequested
ApprovalGranted
ApprovalRejected
TerraformGenerated
TerraformValidated
TerraformPlanCreated
ProvisioningStarted
ResourceProvisioning
ResourceCreated
ResourceProvisioningFailed
VerificationStarted
VerificationCompleted
RequestCompleted
RequestFailed
```

All events should contain:

```text
request_id
workflow_id
timestamp
actor
stage
event_type
payload
```

---

# 29. Request History

Users must be able to open any historical request.

Request detail page should show:

```text
Request
Conversation
Intent
Architecture
Blueprint versions
Cost estimates
Approvals
Terraform
Terraform plan
Execution logs
Events
Created resources
Failures
Remediation
Audit history
```

---

# 30. Resource Inventory

Every provisioned resource should have a platform identity.

Example:

```json
{
  "platform_resource_id": "res_92831",
  "request_id": "REQ-1234",
  "type": "gke_cluster",
  "name": "platform-gke",
  "gcp_resource_id": "projects/.../clusters/platform-gke",
  "terraform_address": "module.gke.google_container_cluster.primary",
  "blueprint_version": 3,
  "status": "ACTIVE"
}
```

---

# 31. Drift Detection

Post-MVP, the platform should periodically compare:

```text
Approved Blueprint
       ↓
Terraform State
       ↓
Actual GCP State
```

Example:

```text
DRIFT DETECTED

GKE node count

Expected: 3
Actual: 5
```

The platform should:

- explain the drift
- identify when it occurred if possible
- show the source of change where available
- propose remediation

Automatic remediation should require explicit policy authorization.

---

# 32. User Interface Requirements

The MVP UI will be implemented with **Chainlit**.

The primary interface should include the following areas.

## 32.1 Request / Chat

Natural-language input:

```text
What would you like to provision?
```

The conversation should support:

- infrastructure requests
- clarification questions
- blueprint modifications
- approval commands
- status questions
- request history queries

---

## 32.2 Blueprint View

Display:

- resources
- relationships
- configuration
- dependencies
- assumptions

---

## 32.3 Architecture View

Visual graph:

```text
VPC
 ├── GKE
 │    └── Node Pool
 │
 └── Cloud SQL
```

---

## 32.4 Cost View

Display:

```text
Estimated Monthly Cost

GKE          $XXX
Cloud SQL    $XXX
Networking   $XX
----------------
Total        $XXX
```

---

## 32.5 Approval View

Display:

```text
Blueprint
Cost
Security
Policies
Assumptions
Terraform summary
```

Actions:

```text
Approve
Modify
Reject
```

---

## 32.6 Execution View

Display:

```text
Pipeline

✓ Request
✓ Architecture
✓ Blueprint
✓ Cost
✓ Policy
✓ Approval
✓ Terraform
✓ Plan
◉ Apply
○ Verify
```

---

## 32.7 Resource View

Display:

```text
Resource
Type
GCP ID
Region
Status
Blueprint
Terraform
Created At
```

---

## 32.8 Request History

Search/filter:

```text
Request ID
User
Project
Environment
Status
Date
Resource
```

---

# 33. API Requirements

The backend API will be implemented using **FastAPI**.

Example endpoints:

```text
POST /api/v1/requests
GET  /api/v1/requests
GET  /api/v1/requests/{id}

GET  /api/v1/requests/{id}/events
GET  /api/v1/requests/{id}/blueprint
GET  /api/v1/requests/{id}/blueprints

POST /api/v1/requests/{id}/modify
POST /api/v1/requests/{id}/approve
POST /api/v1/requests/{id}/reject

GET  /api/v1/requests/{id}/terraform
GET  /api/v1/requests/{id}/plan

GET  /api/v1/resources
GET  /api/v1/resources/{id}

GET  /api/v1/projects
GET  /api/v1/environments
```

Real-time endpoints:

```text
GET /api/v1/requests/{id}/stream
WS  /api/v1/requests/{id}/ws
```

The exact SSE/WebSocket API may be refined during system design.

---

# 34. Core Data Model

Primary entities:

```text
User
Organization
Project
Environment
Request
Conversation
Message
Intent
Blueprint
BlueprintVersion
Architecture
Approval
PolicyEvaluation
CostEstimate
TerraformRun
TerraformPlan
Execution
Resource
Event
AuditLog
Module
Skill
Connector
```

Relationships:

```text
Organization
   ↓
Project
   ↓
Environment
   ↓
Request
   ├── Conversation
   ├── Intent
   ├── Blueprint Versions
   ├── Approvals
   ├── Terraform Runs
   ├── Executions
   ├── Events
   └── Resources
```

PostgreSQL will be the primary system of record.

---

# 35. Non-Functional Requirements

## Availability

Target:

```text
99.9% control-plane availability
```

Provisioning operations must be durable even if the UI disconnects.

---

## Durability

Workflow state must survive:

- API restart
- worker restart
- browser disconnect
- temporary GCP failures

---

## Security

The platform must support:

- least privilege
- encryption
- audit logs
- identity-based access
- secret isolation
- environment isolation

---

## Scalability

The system should support horizontal scaling of:

- FastAPI instances
- LangGraph workers
- workflow workers
- Terraform execution workers
- event processors

---

## Observability

Every request must be traceable using:

```text
request_id
workflow_id
agent_run_id
blueprint_id
terraform_run_id
execution_id
```

---

# 36. Recommended Deployment Architecture

The initial platform should run on GCP.

Conceptually:

```text
                        Internet
                           │
                           ▼
                     Chainlit UI
                           │
                    SSE / WebSocket
                           │
                           ▼
                      FastAPI
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
         LangGraph                  PostgreSQL
          Workers                  Cloud SQL
              │
              ▼
           Pub/Sub
              │
       ┌──────┴───────┐
       │              │
       ▼              ▼
Terraform Workers   Other Workers
       │
       ▼
  Remote Terraform State
       │
       ▼
      GCP
```

The precise compute services can be selected during the technical design phase, but the control plane and execution plane should remain logically separated.

---

# 37. Repository Architecture

Recommended repository:

```text
infra-ai-platform/

├── apps/
│   └── api/
│
├── ui/
│   └── chainlit/
│
├── services/
│   ├── request/
│   ├── blueprint/
│   ├── architecture/
│   ├── cost/
│   ├── policy/
│   ├── terraform/
│   ├── execution/
│   ├── verification/
│   └── inventory/
│
├── agents/
│   ├── intent/
│   ├── architecture/
│   ├── blueprint/
│   ├── terraform/
│   └── operations/
│
├── graph/
│   └── workflows/
│
├── skills/
│   ├── gcp-discovery/
│   ├── vpc/
│   ├── gke/
│   ├── cloudsql/
│   ├── cost/
│   ├── policy/
│   ├── terraform/
│   └── verification/
│
├── connectors/
│   ├── gcp/
│   ├── terraform/
│   └── github/
│
├── modules/
│   └── terraform/
│       ├── vpc/
│       ├── gke/
│       └── cloudsql/
│
├── knowledge/
│   ├── graph/
│   ├── documents/
│   └── embeddings/
│
├── schemas/
│   ├── intent/
│   ├── blueprint/
│   └── events/
│
├── models/
│
├── repositories/
│
├── policies/
│
└── tests/
    ├── unit/
    ├── integration/
    ├── workflow/
    ├── agent/
    └── evaluation/
```

---

# 38. MVP Release Definition

The MVP is complete when a user can perform the following successfully:

### Input

> "Create a GKE cluster with 3 nodes in us-central1, a VPC, and PostgreSQL Cloud SQL."

### Platform

```text
✓ Understand request
✓ Generate intent
✓ Identify missing requirements
✓ Generate architecture
✓ Generate blueprint
✓ Estimate cost
✓ Validate policies
✓ Show blueprint
✓ Allow natural-language modification
✓ Obtain approval
✓ Generate Terraform
✓ Validate Terraform
✓ Generate Terraform plan
✓ Show plan
✓ Execute Terraform
✓ Display real-time status
✓ Verify resources
✓ Display resource IDs
✓ Persist complete request history
```

This constitutes the first end-to-end product milestone.

---

# 39. MVP Acceptance Criteria

## AC-01 — Natural Language

Given a valid infrastructure request, the platform must create a structured intent.

## AC-02 — Blueprint

The platform must generate a schema-valid infrastructure blueprint.

## AC-03 — Dependencies

The blueprint must represent dependencies between resources.

## AC-04 — Cost

The platform must produce an estimated monthly cost and show its assumptions.

## AC-05 — Policy

The blueprint must pass configured organizational policies before execution.

## AC-06 — Human Approval

The platform must not execute infrastructure changes before required approval is received.

## AC-07 — Modification

The user must be able to modify a blueprint through natural language before execution.

## AC-08 — Versioning

Each blueprint modification must create a new immutable blueprint version.

## AC-09 — Terraform

The approved blueprint must produce valid Terraform.

## AC-10 — Plan

The platform must generate and persist the Terraform plan.

## AC-11 — Provisioning

The platform must execute Terraform in an isolated execution environment.

## AC-12 — Status

The user must see real-time provisioning progress through SSE/WebSockets.

## AC-13 — Verification

The platform must verify that expected resources actually exist after provisioning.

## AC-14 — Inventory

The platform must record the GCP identity of created resources.

## AC-15 — Audit

The platform must maintain a complete audit trail.

---

# 40. Success Metrics

## Product Metrics

### Provisioning completion rate

Percentage of approved requests successfully provisioned.

Initial target:

```text
> 95% for supported resource types
```

### Blueprint acceptance rate

Percentage of generated blueprints approved without major modification.

### Human correction rate

Percentage of blueprints requiring significant modification.

### Time to infrastructure

Measure:

```text
Request submitted
       ↓
Infrastructure ready
```

### Terraform success rate

Percentage of generated Terraform configurations that successfully pass validation and plan.

### Policy violation rate

Number of proposed configurations violating mandatory policies.

### User satisfaction

Measure:

- provisioning experience
- blueprint clarity
- cost clarity
- execution visibility

---

# 41. AI Quality Metrics

The AI layer should be evaluated independently.

Metrics:

```text
Intent accuracy
Blueprint validity
Dependency accuracy
Resource selection accuracy
Module selection accuracy
Policy violation rate
Cost estimation accuracy
Human correction rate
Terraform generation success rate
Failure diagnosis accuracy
```

Create a permanent evaluation dataset containing representative infrastructure requests.

---

# 42. Product Roadmap

## Phase 1 — MVP

```text
Natural language
Intent
Blueprint
GKE
VPC
Cloud SQL
Cost
Policy
HITL
Terraform
Provisioning
Verification
History
```

## Phase 2 — Platform Expansion

Add:

```text
Cloud Storage
Pub/Sub
Redis
Load Balancer
Cloud Run
Artifact Registry
Secret Manager
IAM
```

## Phase 3 — Enterprise

Add:

```text
SSO
RBAC
Organizations
Multiple GCP projects
Multiple environments
Advanced approval workflows
Budget controls
ITSM integration
Git integration
Compliance
```

## Phase 4 — Infrastructure Intelligence

Add:

```text
Drift detection
Cost optimization
Architecture optimization
Failure diagnosis
Remediation suggestions
Infrastructure explainability
```

## Phase 5 — Autonomous Infrastructure Operations

Eventually support:

```text
Observe
   ↓
Detect
   ↓
Understand
   ↓
Propose
   ↓
Approve
   ↓
Remediate
   ↓
Verify
```

with explicit organizational policies controlling the level of autonomy.

---

# 43. Key Risks

## Risk 1 — LLM generates incorrect infrastructure

Mitigation:

```text
Structured schemas
+
Module registry
+
Policy engine
+
Terraform validation
+
Human approval
```

## Risk 2 — Excessive permissions

Mitigation:

```text
Least privilege
+
separate identities
+
approval gates
+
execution isolation
```

## Risk 3 — Incorrect cost estimates

Mitigation:

```text
Dedicated pricing service
+
explicit assumptions
+
timestamped pricing
```

## Risk 4 — Terraform state conflicts

Mitigation:

```text
Environment locks
+
remote state
+
serialized execution
```

## Risk 5 — Partial infrastructure failure

Mitigation:

```text
Execution state
+
verification
+
failure classification
+
remediation workflow
```

## Risk 6 — AI hallucination

Mitigation:

```text
Knowledge graph
+
RAG
+
structured schemas
+
deterministic validation
+
approved modules
```

---

# 44. Fundamental Product Architecture

The platform should ultimately be understood as five layers.

```text
┌──────────────────────────────────────────────┐
│                EXPERIENCE                    │
│ Chainlit / Chat / Blueprint / Cost / Status │
├──────────────────────────────────────────────┤
│                INTELLIGENCE                 │
│ LangGraph / Agents / Skills / Knowledge     │
├──────────────────────────────────────────────┤
│             CONTROL PLANE                   │
│ Workflow / Policy / Approval / Audit        │
├──────────────────────────────────────────────┤
│          INFRASTRUCTURE ENGINE              │
│ Blueprint / Modules / Terraform             │
├──────────────────────────────────────────────┤
│             EXECUTION PLANE                 │
│ Terraform / GCP / Verification              │
└──────────────────────────────────────────────┘
```

The **Control Plane + Blueprint + Terraform execution layers are the foundation**.

The AI layer sits above them rather than replacing them.

---

# 45. Final Product Definition

The product can be summarized as:

> **An AI-native infrastructure control plane that translates natural-language requirements into governed, reviewable, cost-aware infrastructure blueprints and safely provisions them through Terraform on Google Cloud.**

The central product flow is:

```text
                 USER
                   │
                   ▼
          Natural Language
                   │
                   ▼
             AI Intent
                   │
                   ▼
          Architecture Plan
                   │
                   ▼
        Infrastructure Blueprint
                   │
          ┌────────┼────────┐
          │        │        │
          ▼        ▼        ▼
        Cost    Policy   Validation
          │        │        │
          └────────┼────────┘
                   ▼
             HUMAN REVIEW
                   │
             ┌─────┴─────┐
             │           │
          Modify       Approve
             │           │
             └─────┬─────┘
                   ▼
          Terraform Generation
                   │
                   ▼
          Terraform Validation
                   │
                   ▼
            Terraform Plan
                   │
                   ▼
            Approval Gate
                   │
                   ▼
          Terraform Apply
                   │
                   ▼
                       GCP
                   │
                   ▼
             Verification
                   │
                   ▼
          Resource Inventory
                   │
                   ▼
             Request History
```

The most important strategic decision is to make **Infrastructure Blueprint + Module Registry + Policy Engine + Execution Engine** the stable core of the platform.

The AI agents, LangGraph workflows, models, RAG, and knowledge graph should evolve around that core.

This prevents the product from becoming an unreliable "LLM that writes Terraform" and instead makes it a genuine **infrastructure automation/control-plane product with AI as its natural-language intelligence layer**.
