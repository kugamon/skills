---
name: sf-message-grammar-audit
description: "Audit user-facing messages in a Salesforce managed package for grammatical issues and generate a prioritized remediation report. Extracts messages from Apex classes, Apex triggers, LWC, Aura Components, Visualforce Pages, and Visualforce Components — same extraction as sf-message-catalog — then runs each through grammar analysis to flag article errors, subject-verb agreement, plural/singular mismatches, spelling, capitalization, punctuation, and cross-component wording inconsistencies. Produces a .docx report with current message, issue, suggested fix, component reference, and priority. Use when the user mentions 'grammar audit', 'message quality', 'fix error messages', 'grammatical issues', 'proofread messages', 'message consistency', 'clean up error messages', 'improve error messages', or asks to review messages for grammar, spelling, or consistency. Also triggers on 'typos in our messages' or 'message style review'. Internal-facing; can be scheduled monthly."
---

# Salesforce Managed Package — Message Grammar Audit

Generate an internal remediation report identifying grammatical issues in user-facing messages across a Salesforce managed package. This skill builds on the same extraction pipeline as `sf-message-catalog` but adds a grammar analysis pass and produces a different output format focused on actionable fixes.

## Before You Begin

1. **Read the docx skill** — locate and read the `docx` SKILL.md in your available skills. You will generate a .docx using docx-js.
2. **Check if a recent message catalog exists.** If the user has already run `sf-message-catalog` and the output is available, you can read that file to extract messages instead of re-querying Salesforce. Ask the user.
3. **If no catalog exists**, follow the same extraction process as `sf-message-catalog` (query Tooling API across all five component types). The extraction instructions are identical — refer to that skill for the full Tooling API query patterns and message-detection regex.
4. **Identify the Salesforce org connector.** Ask the user which connected Salesforce org to query. Look for available MCP tools matching `mcp__salesforce-*__tooling_execute`. If multiple orgs are connected, ask the user to pick one.
5. **Identify the managed package namespace prefix** (e.g., `kugo2p`).

## Phase 1: Extract All Messages

Follow the same five-type extraction as `sf-message-catalog`:

1. **Apex Classes + Triggers** — `addError()`, `ApexPages.addMessage()`, `throw new ...Exception()`
2. **LWC** — `ShowToastEvent`, `setCustomValidity`, `.catch()` handlers, custom error components
3. **Aura** — `force:showToast`, `component.set("v.errorMessage")`, `ServerActionService`, `c:Alert`/`c:Toast`
4. **VF Pages** — `apex:pageMessages`, `apex:pageMessage`, JS `alert()`/`confirm()`, conditional panels
5. **VF Components** — inline validation text, watermarks, conditional `apex:outputText`

Parallelize extraction across component types using subagents when possible.

For each message, record: component name, method/context, message text, condition, severity, and source type (Apex/LWC/Aura/VF Page/VF Component).

## Phase 2: Grammar Analysis

Analyze every extracted message for the following issue categories. This is the core value of this skill — be thorough and precise.

### 2.1 Article Errors (a/an)

Check every instance of "a" or "an" followed by a word:
- "a" before a vowel sound → should be "an" (e.g., "a Order" → "an Order", "a Invoice" → "an Invoice")
- "an" before a consonant sound → should be "a" (e.g., "an Product" → "a Product")
- Watch for acronyms where pronunciation matters (e.g., "an SQL" is correct, "a URL" is correct)

### 2.2 Subject-Verb Agreement

Check that subjects and verbs agree in number:
- "One of the Product **has**" — "Product" should be "Products" (object of "of" is plural)
- "One or more of the product/service **do** not belong" — singular noun with plural verb
- "line **exist**" — singular subject needs "exists"
- "There **is** ... Charge(s)" — "(s)" suffix implies plural, verb should be "are"

### 2.3 Plural/Singular Mismatches

Check for contradictions between articles and plural markers:
- "an Inactive Product(s)" — "an" contradicts "(s)"
- "has an ... Line(s)" — same pattern
- Fix by either dropping the article or removing the "(s)"

### 2.4 Spelling and Compound Words

- "Can not" → "Cannot" (one word in standard English)
- "Pricebook" vs "Price Book" — check for consistency within the package
- Common misspellings in technical messages

### 2.5 Capitalization Consistency

Check for inconsistent mid-sentence capitalization across related messages:
- "contact Roles" vs "Contact Roles" — pick one and flag the inconsistent ones
- Field names used as UI labels should follow a consistent pattern
- Compare across components: if `newQuote` uses "Contact Roles" but `newOrder` uses "contact Roles", flag it

### 2.6 Punctuation

- Missing periods at the end of complete sentences
- Double periods
- Missing Oxford commas in three-item lists (e.g., "Buying, Shipping and Billing" → "Buying, Shipping, and Billing")

### 2.7 Cross-Component Inconsistencies

Compare messages that convey the same meaning across different components:
- Quote vs. Order vs. Invoice versions of the same validation should use parallel wording
- "products/services" in one handler but "product/service" in the equivalent handler for another object
- "do not belong to this Pricebook" vs. "do not belong to the Quote Pricebook" — standardize

### Priority Assignment

Assign each issue a priority:

| Priority | Criteria |
|----------|----------|
| **High** | Clear grammar error visible to end users (wrong article, misspelling, broken subject-verb agreement). Fix before next release. |
| **Medium** | Agreement mismatch or cross-component inconsistency. Fix when touching the component. |
| **Low** | Capitalization or style inconsistency. Fix opportunistically. |

## Phase 3: Generate the Report

Create a landscape .docx using docx-js (`npm install docx` if needed).

### Title Page
- Title: `{Package Name} — Grammatical Issue Report`
- Subtitle: `Generated {date}`
- Tag line: `Internal — For Development Team Use`

### Summary Section
- Paragraph: total messages scanned, total issues found
- Summary table with columns: **Issue Type**, **High**, **Medium**, **Low**, **Total**
- Rows for each category (Article Errors, Subject-Verb Agreement, Plural/Singular, Spelling, Capitalization, Style)
- Bold **TOTAL** row

### Priority Legend
- **High** (red `#C00000`) — Fix before next release
- **Medium** (amber `#BF8F00`) — Fix when touching the component
- **Low** (blue `#2E75B6`) — Fix opportunistically

### Detailed Findings Table

An 8-column landscape table:

| # | Category | Component | Source Ref | Current Message | Suggested Fix | Priority | Issue Description |
|---|----------|-----------|------------|-----------------|---------------|----------|-------------------|

Formatting rules:
- **Component** column: monospace font (`Courier New`)
- **Source Ref**: italic gray text showing component type + line reference (e.g., "Apex — SalesOrderTriggerHandler", "LWC — newOrder")
- **Current Message**: normal font, exact text as it appears in source
- **Suggested Fix**: bold green text (`#548235`) showing the corrected message
- **Priority**: color-coded (High=red, Medium=amber, Low=blue), bold, centered
- **Issue Description**: italic text explaining the specific problem

### Document Formatting
- Landscape orientation (15840 x 12240 DXA)
- 0.75" margins (1080 DXA)
- Arial font throughout
- Header: report title right-aligned in gray
- Footer: centered page numbers
- Alternating row shading (`#F2F7FB` / `#FFFFFF`)
- Dark header rows (`#1F4E79` with white text)

### Validation
After generating, validate:
```bash
python <path-to-docx-skill>/scripts/office/validate.py output.docx
```

## Phase 4: Deliver

Save the file as `{PackageName} Grammatical Issue Report.docx` and provide the user a link.

## Scheduling

This skill is designed to run monthly so the development team can track message quality improvements over time. When scheduling:
- Use cron expression `0 9 1 * *` (1st of each month at 9 AM)
- The scheduled prompt should include the namespace prefix and org connector name
- Each run overwrites the previous report
- Consider running this after `sf-message-catalog` so the catalog is fresh

## Relationship to sf-message-catalog

This skill shares the same Phase 1 extraction pipeline as `sf-message-catalog`. The difference:

| | sf-message-catalog | sf-message-grammar-audit |
|---|---|---|
| **Audience** | Customers, end users | Internal development team |
| **Focus** | Complete inventory of all messages | Only messages with grammatical issues |
| **Output** | Full catalog organized by component | Prioritized remediation report |
| **Action** | Reference document | Fix list with suggested corrections |

If both skills are scheduled, run `sf-message-catalog` first, then this skill can optionally read the catalog instead of re-querying Salesforce.
