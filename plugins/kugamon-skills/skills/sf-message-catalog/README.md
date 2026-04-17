# sf-message-catalog

A Claude skill that generates a comprehensive catalog of every user-facing error, warning, info, and confirmation message across a Salesforce managed package.

## What It Does

This skill connects to a Salesforce org via MCP, queries the Tooling API for all source code in a managed package namespace, and extracts every user-facing message. It covers five component types:

- **Apex Classes & Triggers** -- `addError()`, `ApexPages.addMessage()`, thrown exceptions
- **Lightning Web Components (LWC)** -- `ShowToastEvent`, `setCustomValidity`, `.catch()` handlers
- **Aura Lightning Components** -- `force:showToast`, `component.set("v.errorMessage")`, `ServerActionService`
- **Visualforce Pages** -- `apex:pageMessages`, `apex:pageMessage`, JS `alert()`/`confirm()`
- **Visualforce Components** -- inline validation text, watermarks, conditional output panels

The output is a professional landscape Word document (.docx) with:

- An executive summary table showing message counts by severity and functional area
- Color-coded severity labels (Error, Warning, Info, Confirm)
- Detailed tables for each component group listing every message, its triggering condition, and the method where it lives

## Who It's For

This skill is designed for **Salesforce ISVs and consulting partners** who want to give their customers a complete reference of the messages their app can display. It's useful for:

- Customer onboarding documentation
- Support team reference guides
- AppExchange listing supplementary materials
- Internal QA review of message coverage

## Requirements

- **Claude Desktop (Cowork mode)** or **Claude Code**
- A **Salesforce MCP connector** connected to the target org (the skill prompts you to pick which org at runtime)
- The **docx** skill (included with Cowork by default) for document generation
- **Node.js** with the `docx` npm package (installed automatically if missing)

## Usage

Trigger the skill by asking Claude something like:

- "Generate a message catalog for our managed package"
- "List all error and warning messages in the kugo2p namespace"
- "Create an error message document for our Salesforce app"
- "Catalog every user-facing message in our package"

Claude will ask you for:

1. Which Salesforce org to connect to
2. The managed package namespace prefix (e.g., `kugo2p`, `SBQQ`, `blng`)

Then it extracts messages across all component types and generates the document.

## Scheduling

This skill supports monthly scheduling to keep the catalog current as your package evolves. The default cron is `0 9 1 * *` (1st of each month at 9 AM). You can set this up by asking:

- "Schedule the message catalog to run monthly"
- "Run this on the 1st of every month"

## Example Output

The generated document includes:

**Executive Summary Table**

| Functional Area | Errors | Warnings | Info | Confirm | Total |
|-----------------|--------|----------|------|---------|-------|
| Apex Classes    | 158    | 26       | 17   | 16      | 217   |
| LWC Components  | 42     | 5        | 13   | 13      | 73    |
| Aura Components | 46     | 7        | 22   | 9       | 84    |
| VF Pages        | 62     | 9        | 24   | 5       | 100   |
| VF Components   | 8      | 4        | 4    | 0       | 16    |
| **TOTAL**       | **316**| **51**   | **80**| **43** | **490**|

*Counts shown are from a real run against a CPQ managed package with ~200 components.*

**Detail Table (per component)**

| # | Method | Message | Condition | Severity |
|---|--------|---------|-----------|----------|
| 1 | validatePricebook | Price Book name entered is invalid. Please use the Choose Price Book button to change the Price Book. | PricebookName is null OR pricebook not found | ERROR |
| 2 | handleBeforeUpdate | You cannot change the Quote currency if there are child line items. | Multi-currency enabled, currency changed, AND quote has child lines | ERROR |

## Related Skills

- **sf-message-grammar-audit** -- Builds on this skill's extraction but focuses on finding grammatical issues in messages. Produces an internal remediation report instead of a customer-facing catalog.

## License

MIT License. See the [LICENSE](../../LICENSE) at the plugin root for details.
