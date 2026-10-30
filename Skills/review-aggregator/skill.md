---
name: review-aggregator
description: Combine Drawing Review, Package Review, and ECN Review outputs into a single review result.
input:
  - drawing-review-schema[]
  - package-review-schema
  - ecn-review-schema
output: review-result-schema
---

# Purpose

Combine review results from multiple review skills into a single normalized review result.

This skill does not:

- Review drawings
- Review packages
- Review ECNs
- Create new findings
- Modify findings

This skill only aggregates existing review results.

---

# Inputs

The following inputs may be provided:

- Drawing Review Schema[]
- Package Review Schema
- ECN Review Schema

Drawing Review results are required.

Package Review and ECN Review may be omitted.

---

# Output

Produce a combined-review-schema.

The Review Result Schema shall serve as the authoritative review result for all report formats.

Future outputs may include:

- Markdown reports
- Excel reports
- PDF reports
- Dashboards
- Release summaries

---

# Aggregation Rules

Do not modify findings.

Do not rewrite findings.

Do not combine findings.

Preserve all original checks.

Preserve all original statuses.

Preserve all original pages.

Preserve all original related parts.

---

# Drawing Collection

Create one drawing entry for each reviewed drawing.

Populate:

```json
{
  "part_num": "",
  "description": "",
  "rev": "",
  "status": "",
  "checks": []
}
```

from available review inputs.

---

# Status Calculation

## Drawing Status

A drawing status is determined from all checks assigned to that drawing.

### FAIL

If any check for that drawing has status:

```text
FAIL
```

drawing status shall be:

```text
FAIL
```

### UNCLEAR

If no checks FAIL and one or more checks are:

```text
UNCLEAR
```

drawing status shall be:

```text
UNCLEAR
```

### PASS

If all checks PASS:

```text
PASS
```

---

# Drawing Review Checks

All Drawing Review checks belong to the drawing being reviewed.

Store them in:

```json
drawings[].checks
```

Set:

```json
source = "DRAWING"
```

---

# Package Review Checks

When a Package Review finding references a specific drawing:

Attach the check to that drawing.

Set:

```json
source = "PACKAGE"
```

Examples:

- Description mismatch
- Missing component drawing
- BOM inconsistency
- Revision mismatch

---

# Package Findings Without Drawing Ownership

If a Package Review finding cannot be associated with a specific drawing:

Store the finding in:

```json
remaining_findings[]
```

Set:

```json
source = "PACKAGE"
```

Examples:

- Duplicate drawing numbers
- Circular references
- Global package issues

---

# ECN Review Checks

When an ECN finding references a specific drawing:

Attach the check to that drawing.

Set:

```json
source = "ECN"
```

Examples:

- Revision mismatch
- Description mismatch
- Quantity mismatch

---

# ECN Findings Without Drawing Ownership

If an ECN finding cannot be associated with a specific drawing:

Store it in:

```json
remaining_findings[]
```

Set:

```json
source = "ECN"
```

Examples:

- Missing ECN item
- Unmatched ECN record

---

# Summary Counts

Calculate:

```json
summary
```

using all supplied review results.

---

## Drawing Failures

Count:

```text
Drawing Review checks
```

with status:

```text
FAIL
```

---

## Drawing Unclear

Count:

```text
Drawing Review checks
```

with status:

```text
UNCLEAR
```

---

## Package Failures

Count:

```text
Package Review checks
```

with status:

```text
FAIL
```

---

## Package Unclear

Count:

```text
Package Review checks
```

with status:

```text
UNCLEAR
```

---

## ECN Failures

Count:

```text
ECN Review checks
```

with status:

```text
FAIL
```

---

## ECN Unclear

Count:

```text
ECN Review checks
```

with status:

```text
UNCLEAR
```

---

# Overall Status

Determine:

```json
overall_status
```

from all checks.

## FAIL

If any check has status:

```text
FAIL
```

Overall status shall be:

```text
FAIL
```

## UNCLEAR

If no checks FAIL and one or more checks are:

```text
UNCLEAR
```

Overall status shall be:

```text
UNCLEAR
```

## PASS

If all checks PASS:

```text
PASS
```

---

# Output Rules

- Preserve every check.
- Preserve every finding.
- Do not invent findings.
- Do not suppress findings.
- Do not create new review conclusions.
- Do not perform additional validation.
- The output must be a valid combined-review-schema.