---
name: review-report-markdown
description: Generate a human-readable engineering review report from a Combined Review Schema.
input: combined-review-schema
output: markdown-report
---

# Purpose

Convert a Combined Review Schema into a readable engineering review report.

This skill does not:

- Perform reviews
- Create findings
- Modify findings
- Change statuses
- Validate data

This skill only formats review results.

---

# Input

Input shall be a valid Combined Review Schema.

---

# Output

Generate a Markdown report.

Do not output JSON.

Do not output schemas.

Do not add findings that do not exist in the source schema.

---

# Report Structure

Generate the report using the following structure.

# PACKAGE REVIEW

Status: <overall_status>

## Summary

Drawing Failures: <drawing_failures>

Drawing Unclear: <drawing_unclear>

Package Failures: <package_failures>

Package Unclear: <package_unclear>

ECN Failures: <ecn_failures>

ECN Unclear: <ecn_unclear>

---

## Drawing Results

Generate one section for each drawing.

Format:

### <part_num> | <status>

Description: <description>

Revision: <rev>

Only include checks with status:

- FAIL
- UNCLEAR

Do not include PASS checks.

For every displayed check:

- Include source
- Include check name
- Include affected pages
- Include findings

Example:

### 146703B | FAIL

Description: BAR,TOW,WELDMENT

Revision: 01

**DRAWING - Revision Row Count**

Pages: 1

- Revision 01 requires 2 revision rows but only 1 row was found.

**PACKAGE - BOM Description Match**

Pages: 1

- Assembly 146974B item BN950 description does not match referenced drawing description.

**ECN - Drawing Revision Match**

Pages: 1

- Drawing revision 01 does not match ECN revision 02.

---

### 146974B | PASS

Description: GROUP,BAR,TOW

Revision: 01

No findings.

---

## Remaining Package and ECN Findings

Generate this section only when:

```json
remaining_findings.length > 0
```

For each remaining finding display:

- Source
- Check
- Pages
- Findings

Example:

### PACKAGE - Duplicate Drawings

Pages: 1, 4

- Part number 146703B appears multiple times in the package.

### ECN - Missing Affected Drawing

Pages: 2

- ECN references BN951 but no drawing was found in the package.

---

# Drawing Status Display

Display all drawings.

PASS drawings shall still be included.

For PASS drawings display:

```text
No findings.
```

---

# Ordering Rules

Preserve the drawing order provided by the Combined Review Schema.

Do not reorder drawings.

The report shall display drawings in the same order they appear in:

```json
drawings[]
```

This order is assumed to represent the original drawing package sequence.

Within each drawing:

1. Display FAIL checks first.
2. Display UNCLEAR checks second.
3. Do not display PASS checks.

Preserve the order of findings within each check.

---

# Traceability

Maintain the original package sequence throughout the report.

A drawing's position within the report shall correspond to its position within the input Combined Review Schema.

This preserves traceability to the original drawing package and package page numbering.

---

# Pages Formatting

When pages contain values:

```json
[1]
```

Display:

```text
Pages: 1
```

When pages contain multiple values:

```json
[4,5,6]
```

Display:

```text
Pages: 4, 5, 6
```

When pages are empty:

Do not display a Pages line.

---

# Findings Formatting

Display each finding as a separate bullet.

Example:

```text
- Material specification is missing.
- Coating specification is missing.
```

Do not merge findings together.

Preserve original wording.

---

# Status Formatting

Display statuses exactly as supplied.

Allowed values:

```text
PASS
FAIL
UNCLEAR
```

Do not replace statuses with icons, colors, emojis, or alternate wording.

---

# Formatting Rules

Use Markdown headings.

Use bullet lists for findings.

Use bold text for check titles.

Preserve all findings exactly as supplied.

Do not rewrite findings.

Do not summarize findings.

Do not remove findings.

Do not invent findings.

Do not output JSON.

Do not output schemas.

Do not output additional commentary.

Generate only the formatted report.