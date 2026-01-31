# Delivery Hub - Project Walkthrough

## Table of Contents

1. [Overview](#overview)
2. [Technology Stack](#technology-stack)
3. [Project Structure](#project-structure)
4. [Core Features](#core-features)
5. [Apex Classes](#apex-classes)
6. [Lightning Web Components](#lightning-web-components)
7. [Data Model](#data-model)
8. [Integration Architecture](#integration-architecture)
9. [Development Setup](#development-setup)

---

## Overview

**Delivery Hub** is an enterprise-grade Salesforce application designed for project management and delivery tracking. It provides a sophisticated ticket management system with a kanban-style interface, bidirectional Jira integration, and AI-powered enhancements using OpenAI.

### Key Capabilities

- **Ticket Management**: Full lifecycle management of delivery work items through customizable workflow stages
- **Kanban Board**: Drag-and-drop interface with multi-persona views (Client, Consultant, Developer, QA)
- **Jira Integration**: Bidirectional synchronization of issues, comments, and attachments
- **AI Enhancement**: OpenAI-powered ticket descriptions, suggestions, and time estimations
- **Dependency Tracking**: Manage blocking relationships between tickets
- **ETA Calculation**: Dynamic completion date calculation based on team capacity
- **Partner Network**: REST API integration for external vendor/partner systems

---

## Technology Stack

| Layer | Technology |
|-------|------------|
| **Platform** | Salesforce DX (API v65.0) |
| **Backend** | Apex (89 classes, 42 test classes) |
| **Frontend** | Lightning Web Components (22 components) |
| **Integration** | REST APIs, Jira Webhooks, OpenAI API |
| **Build Tools** | Salesforce CLI, ESLint, Prettier, Husky |
| **Testing** | sfdx-lwc-jest, Apex Test Classes |

---

## Project Structure

```
delivery-hub/
├── force-app/main/default/
│   ├── classes/                    # Apex classes (89 total)
│   ├── lwc/                        # Lightning Web Components (22 total)
│   ├── aura/                       # Aura Components
│   ├── objects/                    # Custom object definitions
│   ├── layouts/                    # Record layouts
│   ├── flexipages/                 # Lightning pages
│   ├── tabs/                       # Custom tabs
│   ├── triggers/                   # Apex triggers
│   ├── permissionsets/             # Permission sets
│   └── staticresources/            # Static resources
├── config/
│   └── project-scratch-def.json    # Scratch org configuration
├── manifest/
│   └── package.xml                 # Deployment manifest
├── scripts/
│   ├── apex/                       # Apex scripts
│   └── soql/                       # SOQL queries
├── .vscode/                        # VS Code configuration
├── .husky/                         # Git hooks
├── package.json                    # NPM dependencies
└── sfdx-project.json               # Salesforce DX project config
```

---

## Core Features

### 1. Multi-Stage Kanban Board

The primary interface is a kanban board supporting:

- **Customizable Stages**: Backlog, Open, In Progress, UAT, Closed, etc.
- **Drag-and-Drop**: Move tickets between columns with automatic stage transitions
- **Persona Views**: Client, Consultant, Developer, and QA perspectives
- **Card Sizing**: Equal-sized or variable card display modes
- **Filtering**: Board view, intention, and display mode filters

### 2. Jira Bidirectional Synchronization

Complete integration with Jira Cloud:

- **Issue Sync**: Create, update, and delete issues in both directions
- **Comment Sync**: Bidirectional comment synchronization with ADF format support
- **Attachment Sync**: File uploads synchronized between platforms
- **Webhook Support**: Real-time updates via Jira webhooks
- **Batch Processing**: Scheduled sync jobs for consistency

### 3. AI-Powered Features

OpenAI GPT integration providing:

- **Ticket Enhancement**: Auto-generated descriptions and acceptance criteria
- **Time Estimation**: AI-based effort estimation
- **Smart Suggestions**: Context-aware recommendations for ticket content

### 4. Dependency Management

Track relationships between tickets:

- **Blocker Relationships**: Mark tickets that block others
- **Visual Indicators**: Blocked tickets shown with dependency badges
- **Transition Prevention**: Prevents moving blocked tickets to active development

### 5. ETA Calculation Engine

Dynamic completion date calculation:

- **Priority Weighting**: Higher priority tickets processed first
- **Developer Availability**: Considers team capacity
- **Business Days**: Excludes weekends from calculations
- **UAT Buffer**: Configurable buffer days for testing phases

---

## Apex Classes

### Core Controllers

| Class | Purpose |
|-------|---------|
| `DeliveryHubBoardController` | Main kanban board operations, ticket CRUD, dependency management |
| `TicketController` | Ticket data retrieval, AI enhancement, file linking |
| `DeliveryHubSettingsController` | System settings management (AI, Jira, Partner configs) |
| `DeliveryHubDashboardController` | Budget and metrics dashboard data |

### Jira Integration

| Class | Purpose |
|-------|---------|
| `JiraWebhookReceiver` | REST endpoint receiving Jira webhook payloads with authentication |
| `JiraWebhookProcessor` | Async processing of webhook events via Queueable |
| `JiraIssueHandler` | Creates/updates/deletes Ticket__c from Jira issue events |
| `JiraCommentHandler` | Synchronizes comments between platforms |
| `JiraAttachmentHandler` | Downloads Jira attachments to Salesforce ContentVersion |
| `JiraCallout` | Centralized HTTP client for Jira REST API calls |
| `JiraFieldMappingUtil` | Field mapping and ADF-to-HTML conversion |
| `JiraToSFSyncBatch` | Scheduled batch job for periodic synchronization |
| `JiraAttachmentSyncBatch` | Batch job for attachment synchronization |

### External Integration

| Class | Purpose |
|-------|---------|
| `DeliveryHubIntakeService` | REST endpoint for external systems to create tickets |
| `DeliveryHubSender` | Sends requests to network entities/partners |
| `DeliveryHubCommentIntake` | REST endpoint for external comment submission |
| `DeliveryHubCommentSender` | Posts comments to remote systems |
| `DeliveryHubFileSender` | File transfer to external systems |
| `DeliveryHubFileIntake` | File intake from external systems |

### Services & Utilities

| Class | Purpose |
|-------|---------|
| `TicketETAService` | ETA calculation engine with priority-based scheduling |
| `AttachmentSyncService` | Synchronizes Salesforce files to Jira |
| `JiraWebhookConfigService` | Configuration service for webhook processing |
| `HtmlToAdfConverter` | Converts HTML to Atlassian Document Format |
| `JiraCommentSyncHelper` | Helper for comment sync with HTML-to-ADF conversion |
| `AuditLogger` | Creates audit trail comments for field changes |
| `TriggerControl` | Global flag to enable/disable trigger logic |

### Trigger Handlers

| Class | Purpose |
|-------|---------|
| `TicketTriggerHandler` | Orchestrates Ticket__c trigger logic, network sync, blocking validation |
| `ContentDocumentLinkTriggerHandler` | Handles document link events for attachment sync |

---

## Lightning Web Components

### Main Board Components

| Component | Purpose |
|-----------|---------|
| `deliveryHubBoard` | Primary kanban board with drag-drop, filtering, modals, and AI integration |
| `dragAndDropLwc` | Alternative kanban implementation with different configuration approach |
| `dragAndDropList` | Column container for ticket cards |
| `dragAndDropCard` | Individual draggable ticket card |

### Setup & Configuration

| Component | Purpose |
|-----------|---------|
| `deliveryHubSetup` | Initial onboarding and connection setup wizard |
| `settingsContainer` | Tab-based settings hub (General, AI, OpenAI, Jira) |
| `kanbanSettingsContainer` | Kanban-specific settings management |
| `generalSettingsCard` | General system settings |
| `aiSettingsCard` | AI feature toggles (suggestions, auto-descriptions, estimation) |
| `openAISettingsCard` | OpenAI API key and model configuration |
| `jiraSettingsCard` | Jira instance URL, credentials, and project key setup |
| `partnerSettingsCard` | Partner/vendor integration settings |

### Ticket & Request Management

| Component | Purpose |
|-----------|---------|
| `deliveryTicketRefiner` | Edit ticket details and create linked vendor requests |
| `deliveryTicketChat` | Real-time comment interface for ticket discussions |
| `deliveryCommentStream` | Alternative comment display for requests |
| `manageDeliveryRequest` | Vendor request status and management |
| `deliveryFileSender` | File attachment management and sending |
| `deliveryGhostRecorder` | Global issue reporter with keyboard shortcut (Alt+B) |

### Dashboard

| Component | Purpose |
|-----------|---------|
| `deliveryBudgetSummary` | Budget metrics widget (hours, spend, active requests) |

---

## Data Model

### Primary Objects

#### Ticket__c (Custom Object)

The core work item object with fields including:

| Field | Type | Purpose |
|-------|------|---------|
| `BriefDescriptionTxt__c` | Text | Short ticket description |
| `DetailsTxt__c` | Long Text | Detailed content and requirements |
| `StageNamePk__c` | Picklist | Workflow stage (Backlog, Open, In Progress, etc.) |
| `PriorityPk__c` | Picklist | Priority level (High, Medium, Low) |
| `WorkItemTypeTxt__c` | Text | Type of work (Bug, Feature, Task, etc.) |
| `TotalLoggedHoursNumber__c` | Number | Actual hours spent |
| `EstimatedHoursNumber__c` | Number | Estimated effort |
| `DeveloperDaysSizeNumber__c` | Number | Size in developer days |
| `CalculatedETADate__c` | Date | Projected completion date |
| `ProjectedUATReadyDate__c` | Date | UAT target date |
| `JiraKeyTxt__c` | Text | Linked Jira issue key |
| `Developer__c` | Lookup(User) | Assigned developer |
| `Epic__c` | Lookup | Epic association |
| `SortOrderNumber__c` | Number | Manual sort order within stage |

#### Ticket_Comment__c (Custom Object)

Comments associated with tickets:

| Field | Type | Purpose |
|-------|------|---------|
| `Ticket__c` | Lookup | Parent ticket |
| `BodyTxt__c` | Long Text | Comment content |
| `AuthorTxt__c` | Text | Comment author |
| `JiraCommentIdTxt__c` | Text | Linked Jira comment ID |
| `SourcePk__c` | Picklist | Origin (Salesforce, Jira, External) |

#### Ticket_Dependency__c (Custom Object)

Blocking relationships between tickets:

| Field | Type | Purpose |
|-------|------|---------|
| `Ticket__c` | Lookup | The blocked ticket |
| `BlockingTicket__c` | Lookup | The blocking ticket |

#### Request__c (Custom Object)

Vendor/partner requests:

| Field | Type | Purpose |
|-------|------|---------|
| `Ticket__c` | Lookup | Associated ticket |
| `StatusPk__c` | Picklist | Request status |
| `PreApprovedHoursNumber__c` | Number | Pre-approved hours |
| `HourlyRateCurrency__c` | Currency | Hourly rate |
| `RemoteTicketIdTxt__c` | Text | Remote system ticket ID |
| `DeliveryEntityId__c` | Lookup | Network entity |

### Custom Settings

#### Delivery_Hub_Settings__c

System configuration including:
- Jira credentials (instance URL, username, API token)
- OpenAI API key and model selection
- AI feature toggles
- Network entity settings

#### Sync_In_Propgress__c

Synchronization state management to prevent infinite loops during bidirectional sync.

### Custom Metadata

#### Jira_Webhook_Config__mdt

Webhook processing configuration:
- Event type enablement
- Field mappings (JSON)
- Retry attempts
- Sync direction

#### Kanban_Configuration__mdt

Kanban board configuration:
- Stage definitions
- Required fields per stage
- Persona mappings

---

## Integration Architecture

### Jira Integration Flow

```
JIRA → SALESFORCE (Inbound):
┌─────────────────────────────────────────────────────────────────┐
│ Jira Cloud                                                      │
│   ├── Issue Event ─────┐                                        │
│   ├── Comment Event ───┼──► Webhook POST                        │
│   └── Attachment Event─┘                                        │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ JiraWebhookReceiver (REST Endpoint)                             │
│   ├── Validates X-Jira-Secret header                            │
│   ├── Parses JSON payload                                       │
│   ├── Checks for duplicates                                     │
│   └── Enqueues JiraWebhookProcessor                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ JiraWebhookProcessor (Queueable)                                │
│   ├── Routes to JiraIssueHandler ──► Ticket__c CRUD             │
│   ├── Routes to JiraCommentHandler ──► Ticket_Comment__c CRUD   │
│   └── Routes to JiraAttachmentHandler ──► ContentVersion CRUD   │
└─────────────────────────────────────────────────────────────────┘

SALESFORCE → JIRA (Outbound):
┌─────────────────────────────────────────────────────────────────┐
│ TicketTriggerHandler / JiraCommentSyncHelper                    │
│   └── Enqueues Queueable jobs                                   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ JiraCallout                                                     │
│   ├── Sets Sync_In_Propgress__c flag                            │
│   ├── Makes HTTP callouts to Jira REST API                      │
│   └── Unsets flag via UnsetSyncFlagQueueable                    │
└─────────────────────────────────────────────────────────────────┘
```

### Partner Network Integration

```
EXTERNAL SYSTEM → SALESFORCE:
┌─────────────────────────────────────────────────────────────────┐
│ DeliveryHubIntakeService (REST: /deliveryhub/v1/intake)         │
│   └── Creates Ticket__c and Request__c from JSON payload        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ DeliveryHubCommentIntake (REST: /deliveryhub/v1/comments/{id})  │
│   └── GET/POST comments for tickets                             │
└─────────────────────────────────────────────────────────────────┘

SALESFORCE → EXTERNAL SYSTEM:
┌─────────────────────────────────────────────────────────────────┐
│ DeliveryHubSender                                               │
│   └── POST requests to partner endpoints                        │
└─────────────────────────────────────────────────────────────────┘
```

### OpenAI Integration

```
┌─────────────────────────────────────────────────────────────────┐
│ TicketController.getAiEnhancedTicketDetails()                   │
│   ├── Builds prompt with ticket context                         │
│   ├── Calls OpenAI GPT-4o-mini API                              │
│   ├── Parses response to AISuggestionWrapper                    │
│   └── Returns enhanced title, description, acceptance criteria  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Development Setup

### Prerequisites

- Salesforce CLI installed
- Node.js and npm installed
- VS Code with Salesforce Extension Pack

### Installation

1. Clone the repository
2. Install npm dependencies:
   ```bash
   npm install
   ```
3. Authorize a Salesforce org:
   ```bash
   sf org login web -a DevHub
   ```
4. Create a scratch org:
   ```bash
   sf org create scratch -f config/project-scratch-def.json -a delivery-hub
   ```
5. Push source to scratch org:
   ```bash
   sf project deploy start
   ```
6. Open the org:
   ```bash
   sf org open
   ```

### Configuration

1. Navigate to **Delivery Hub Settings** in Salesforce
2. Configure Jira integration:
   - Instance URL (e.g., `https://yourcompany.atlassian.net`)
   - Username (email)
   - API Token (from Atlassian account settings)
   - Project Key
3. Configure OpenAI (optional):
   - API Key
   - Model selection (GPT-4o recommended)
4. Enable AI features as desired

### Testing

Run Apex tests:
```bash
sf apex run test --code-coverage --result-format human
```

Run LWC tests:
```bash
npm run test:unit
```

### Linting

```bash
npm run lint
```

---

## Security Considerations

- **CRUD/FLS Compliance**: Controllers use `with sharing` and FLS checks
- **HMAC Authentication**: Jira webhooks validated with secret header
- **Credential Storage**: API keys stored in protected custom settings
- **Input Validation**: Webhook payloads validated before processing
- **Sync Loop Prevention**: `Sync_In_Propgress__c` flag prevents infinite loops

---

## Key Files Reference

| File | Location |
|------|----------|
| Main Board Component | `force-app/main/default/lwc/deliveryHubBoard/` |
| Jira Webhook Receiver | `force-app/main/default/classes/JiraWebhookReceiver.cls` |
| Field Mapping Utility | `force-app/main/default/classes/JiraFieldMappingUtil.cls` |
| ETA Service | `force-app/main/default/classes/TicketETAService.cls` |
| Settings Controller | `force-app/main/default/classes/DeliveryHubSettingsController.cls` |
| Project Config | `sfdx-project.json` |
| Scratch Org Config | `config/project-scratch-def.json` |

---

*Generated with Claude Code*
