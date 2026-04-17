---
name: sf-message-catalog
description: "Generate a comprehensive catalog of every user-facing error, warning, info, and confirmation message across a Salesforce managed package. Extracts messages from Apex classes, Apex triggers, Lightning Web Components (LWC), Aura Lightning Components, Visualforce Pages, and Visualforce Components. Produces a professional .docx document organized by component type with an executive summary table. Use this skill when the user mentions 'error catalog', 'message catalog', 'list all errors', 'list all warnings', 'user-facing messages', 'error messages document', 'warning messages report', 'managed package messages', 'Q2C messages', or asks to catalog, inventory, or document the error and warning messages in any Salesforce app. Also triggers on 'what messages does the app show', 'customer-facing errors', or 'generate message documentation'. This skill can be scheduled to run monthly to keep the catalog current across releases."
---

# Salesforce Managed Package — Message Catalog Generator

Generate a complete, customer-facing catalog of every user-facing message in a Salesforce managed package. The output is a professional landscape .docx with color-coded severity, organized by component type.

## Before You Begin

1. **Read the docx skill** — locate and read the `docx` SKILL.md in your available skills. You will generate a .docx using docx-js.
2. **Identify the Salesforce org connector.** Ask the user which connected Salesforce org to query. Look for available MCP tools matching patterns like `mcp__salesforce-*__tooling_execute` or `mcp__Salesforce-*__tooling_execute`. If multiple orgs are connected, ask the user to pick one. If only one is available, confirm it.
3. **Identify the managed package namespace prefix.** Ask the user for the namespace (e.g., `kugo2p`). This is used to filter Tooling API queries to only components belonging to that package.

## Phase 1: Extract Messages from All Component Types

Query the Salesforce Tooling API for source code across five component types. For each, extract every user-facing message string along with its method/context, triggering condition, and severity.

### 1.1 Apex Classes and Triggers

Query all Apex classes and triggers in the namespace:

```
SELECT Id, Name, Body FROM ApexClass WHERE NamespacePrefix = '{namespace}'
SELECT Id, Name, Body FROM ApexTrigger WHERE NamespacePrefix = '{namespace}'
```

Search each body for these patterns (in priority order):

| Pattern | Severity |
|---------|----------|
| `.addError('...')` | ERROR |
| `ApexPages.addMessage(...)` with `ApexPages.Severity.ERROR` | ERROR |
| `ApexPages.addMessage(...)` with `ApexPages.Severity.WARNING` | WARNING |
| `ApexPages.addMessage(...)` with `ApexPages.Severity.INFO` | INFO |
| `ApexPages.addMessage(...)` with `ApexPages.Severity.CONFIRM` | CONFIRM |
| `throw new ...Exception('...')` | ERROR |
| String variables assigned then passed to `addError` or `addMessage` | Varies |

For each message found, record:
- **Method**: The Apex method name where the message lives
- **Message**: The exact string literal (preserve `{merge fields}`)
- **Condition**: The `if` / guard clause that triggers it (summarize in plain English)
- **Severity**: ERROR, WARNING, INFO, or CONFIRM

### 1.2 Lightning Web Components (LWC)

Query LWC bundles and their resources:

```
SELECT Id, DeveloperName FROM LightningComponentBundle WHERE NamespacePrefix = '{namespace}'
```

Then for each bundle, fetch JS resources:

```
SELECT Id, Source, FilePath FROM LightningComponentResource
WHERE LightningComponentBundleId = '{bundleId}'
AND FilePath LIKE '%.js'
```

Search each JS source for these patterns:

| Pattern | Severity |
|---------|----------|
| `ShowToastEvent({ ... variant: 'error' })` | ERROR |
| `ShowToastEvent({ ... variant: 'warning' })` | WARNING |
| `ShowToastEvent({ ... variant: 'info' })` | INFO |
| `ShowToastEvent({ ... variant: 'success' })` | CONFIRM |
| Custom `c-toast-lwc` or `c-error-panel` component usage | Varies |
| `setCustomValidity('...')` | ERROR |
| `.catch(error => ...)` with error display | ERROR |
| String literals in `this.errorMessage = '...'` assignments | ERROR |

### 1.3 Aura Lightning Components

Query Aura bundles and their definitions:

```
SELECT Id, DeveloperName FROM AuraDefinitionBundle WHERE NamespacePrefix = '{namespace}'
```

Then for each bundle, fetch controller, helper, and component markup:

```
SELECT Id, Source, DefType FROM AuraDefinition
WHERE AuraDefinitionBundleId = '{bundleId}'
AND DefType IN ('CONTROLLER', 'HELPER', 'COMPONENT')
```

Search for these patterns:

| Pattern | Severity |
|---------|----------|
| `$A.get("e.force:showToast")` with `type: 'error'` | ERROR |
| `$A.get("e.force:showToast")` with `type: 'warning'` | WARNING |
| `$A.get("e.force:showToast")` with `type: 'info'` | INFO |
| `$A.get("e.force:showToast")` with `type: 'success'` | CONFIRM |
| `component.set("v.errorMessage", '...')` | ERROR |
| `ServerActionService` centralized error handlers | ERROR |
| Custom `c:Alert` or `c:Toast` event firing | Varies |
| `aura:if` with `isTrue="{!v.showError}"` in markup | Varies |

### 1.4 Visualforce Pages

```
SELECT Id, Name, Markup FROM ApexPage WHERE NamespacePrefix = '{namespace}'
```

Search each markup for:

| Pattern | Severity |
|---------|----------|
| `<apex:pageMessages/>` (renders controller messages) | Varies |
| `<apex:pageMessage ... severity="error">` | ERROR |
| `<apex:pageMessage ... severity="warning">` | WARNING |
| `<apex:pageMessage ... severity="info">` | INFO |
| `<apex:pageMessage ... severity="confirm">` | CONFIRM |
| JavaScript `alert('...')` | ERROR |
| JavaScript `confirm('...')` | WARNING |
| Conditionally rendered `<div>` or `<apex:outputPanel>` with error text | Varies |
| SLDS toast markup (`slds-notify--toast`) | Varies |
| `c:Alert` VF component references | Varies |

### 1.5 Visualforce Components

```
SELECT Id, Name, Markup FROM ApexComponent WHERE NamespacePrefix = '{namespace}'
```

Search for the same patterns as VF Pages, plus:
- Inline validation text (e.g., `Error: Invalid Quantity`)
- Watermark images with status text (e.g., "Cancelled", "Restricted")
- `<apex:outputText>` with conditional rendering showing error/status messages

## Phase 2: Parallelize Extraction

The extraction across five component types is independent work. Use subagents to process all five types in parallel when possible:

- **Agent 1**: Apex Classes + Triggers
- **Agent 2**: LWC Bundles
- **Agent 3**: Aura Bundles
- **Agent 4**: VF Pages + VF Components

Each agent should return its findings as structured arrays in this format:

```
[sequenceNumber, methodOrContext, messageText, triggerCondition, severity]
```

## Phase 3: Generate the Document

Create a landscape .docx using docx-js (`npm install docx` if needed). The document structure:

### Title Page
- Title: `{Package Name} — Error & Warning Message Catalog`
- Subtitle: `Generated {date}`

### Executive Summary
- Brief paragraph explaining the scope of the catalog
- Summary table with columns: **Functional Area**, **Errors**, **Warnings**, **Info**, **Confirm**, **Total**
- One row per logical section, plus a bold **TOTAL** row

### Severity Legend
- **ERROR** (red `#C00000`) — Blocks the operation
- **WARNING** (amber `#BF8F00`) — Alerts but may allow proceeding
- **INFO** (blue `#2E75B6`) — Informational, does not block
- **CONFIRM** (green `#548235`) — Success confirmation

### Message Sections (one per component group)
Each section gets:
- An `H1` heading with section number and group name
- An italic description paragraph
- Subsections (`H2`) per class/component with a 5-column table:

| # | Method/Context | Message | Condition | Severity |
|---|----------------|---------|-----------|----------|

Color-code the Severity cell text using the severity colors above.

### Document Formatting
- Landscape orientation (15840 x 12240 DXA)
- 0.75" margins (1080 DXA)
- Arial font throughout
- Header: catalog title right-aligned in gray
- Footer: centered page numbers
- Alternating row shading (`#F2F7FB` / `#FFFFFF`)
- Dark header rows (`#1F4E79` with white text)

### Section Ordering
1. Apex (group by functional area: Triggers, Controllers, Helpers, etc.)
2. Lightning Web Components (LWC)
3. Aura Lightning Components
4. Visualforce Pages
5. Visualforce Components

### Validation
After generating, validate the docx:
```bash
python <path-to-docx-skill>/scripts/office/validate.py output.docx
```

## Phase 4: Deliver

Save the file with a descriptive name like `{PackageName} Message Catalog.docx` and provide the user a link.

## Scheduling

This skill is designed to be run monthly to keep the catalog current as the managed package evolves. When scheduling:
- Use cron expression `0 9 1 * *` (1st of each month at 9 AM)
- The scheduled prompt should include the namespace prefix and org connector name so it runs unattended
- Each run overwrites the previous catalog file

## Tips for Large Packages

- Tooling API has governor limits. If a package has hundreds of classes, batch your queries (e.g., 50 classes per query using `Id IN (...)` filters).
- Some Apex classes may exceed the Tooling API response size. For very large classes, you may need to use the `tooling_execute` endpoint with a REST query and paginate.
- LWC and Aura bundles require two-step queries (bundle list, then resources per bundle). Plan for this in your parallelization strategy.
