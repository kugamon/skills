# sf-message-grammar-audit

A Claude skill that audits every user-facing message in a Salesforce managed package for grammatical issues, then generates a prioritized remediation report with suggested fixes.

## What It Does

This skill extracts all user-facing messages from a managed package (using the same pipeline as [sf-message-catalog](../sf-message-catalog)), then runs each message through a grammar analysis that checks for:

- **Article errors** -- "a Order" should be "an Order", "a Invoice" should be "an Invoice"
- **Subject-verb agreement** -- "One of the Product has" should be "One of the Products has"
- **Plural/singular mismatches** -- "an Inactive Product(s)" contradicts itself
- **Spelling** -- "Can not" should be "Cannot"
- **Capitalization inconsistencies** -- "contact Roles" vs. "Contact Roles" across components
- **Punctuation** -- missing Oxford commas, missing periods
- **Cross-component inconsistencies** -- same validation worded differently in Quote vs. Order handlers

Each issue gets a priority level:

| Priority | When to Fix |
|----------|-------------|
| **High** | Clear grammar error visible to end users. Fix before next release. |
| **Medium** | Agreement mismatch or inconsistency. Fix when touching the component. |
| **Low** | Capitalization or style issue. Fix opportunistically. |

The output is a landscape Word document (.docx) with a summary table and a detailed findings table showing the current message, the specific issue, the suggested correction, and the source component reference.

## Who It's For

This skill is designed for **Salesforce ISV development teams** who want to improve the quality and professionalism of their app's user-facing messages. It's useful for:

- Pre-release quality checks
- Technical debt tracking for message wording
- Onboarding new writers to maintain message consistency
- Tracking grammar quality improvements over time via monthly runs

This is an **internal-facing** skill -- the output is a remediation report for developers, not a customer-facing document.

## Requirements

- **Claude Desktop (Cowork mode)** or **Claude Code**
- A **Salesforce MCP connector** connected to the target org
- The **docx** skill (included with Cowork by default) for document generation
- **Node.js** with the `docx` npm package (installed automatically if missing)

## Usage

Trigger the skill by asking Claude something like:

- "Audit our managed package messages for grammar issues"
- "Check all error messages for grammatical problems"
- "Run a grammar audit on our Salesforce app messages"
- "Find typos and grammar issues in our user-facing messages"
- "Are there any grammatical issues in our error messages?"

Claude will ask you for:

1. Which Salesforce org to connect to
2. The managed package namespace prefix (e.g., `kugo2p`, `SBQQ`, `blng`)
3. Whether a recent message catalog already exists (to skip re-extraction)

## Scheduling

This skill supports monthly scheduling. The default cron is `0 9 1 * *` (1st of each month at 9 AM). If you also schedule `sf-message-catalog`, run that one first -- this skill can reuse its output instead of re-querying Salesforce.

Set it up by asking:

- "Schedule the grammar audit to run monthly"
- "Run this after the message catalog each month"

## Example Output

The generated document includes:

**Summary Table**

| Issue Type | High | Medium | Low | Total |
|------------|------|--------|-----|-------|
| Article Errors (a/an) | 6 | 0 | 0 | 6 |
| Subject-Verb Agreement | 1 | 4 | 0 | 5 |
| Plural/Singular Mismatch | 1 | 3 | 0 | 4 |
| Spelling | 1 | 0 | 0 | 1 |
| Capitalization | 0 | 0 | 3 | 3 |
| Style / Word Order | 0 | 0 | 2 | 2 |
| **TOTAL** | **9** | **7** | **5** | **21** |

*Counts shown are from a real audit of a CPQ managed package with ~490 messages.*

**Detail Table (sample rows)**

| # | Category | Component | Current Message | Suggested Fix | Priority |
|---|----------|-----------|-----------------|---------------|----------|
| 1 | Article Error | SalesOrderTriggerHandler | One of the Product in this Quote is inactive. | One of the **Products** in this Quote is inactive. | High |
| 2 | Article Error | newOrder (LWC) | Please select a Order Source first. | Please select **an** Order Source first. | High |
| 3 | Spelling | ImportLinesController | Can not find the data type for field | **Cannot** find the data type for field | High |

## Related Skills

- **sf-message-catalog** -- Generates a complete customer-facing catalog of all messages. This grammar audit skill builds on its extraction pipeline.

## License

MIT License. See the [LICENSE](../../LICENSE) at the plugin root for details.
