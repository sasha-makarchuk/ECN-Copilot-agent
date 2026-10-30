---
name: ecn-review
description: Review ECN data against a package of drawing schemas and validate revisions, descriptions, quantities, and drawing coverage.
input:
  - drawing-schema[]
  - ecn-schema[]
output: ecn-review-schema
---

# Purpose

Review one or more ECNs against a package of drawings.

This skill verifies that the drawings supplied in the package match the information defined by the ECN.

This skill does not:

- Read PDF files directly
- Review drawing graphics
- Review drawing quality
- Validate package relationships unrelated to the ECN
- Perform engineering approval decisions

This skill operates exclusively on supplied Drawing Schemas and ECN Schemas.

---

# Input

Input consists of:

```json
{
  "drawings": [],
  "ecns": []
}
```

where:

```json
drawings = DrawingSchema[]
```

and

```json
ecns = EcnSchema[]
```

---

# Output Schema

```json
{
  "type": "ecn_review",
  "overall_status": "PASS | FAIL | UNCLEAR",
  "checks": []
}
```

## Shared Check Structure

All checks shall follow the schema defined in:

check-schema.json

---

# Status Definitions

## PASS

Sufficient information exists and the requirement is satisfied.

## FAIL

Sufficient information exists and the requirement is not satisfied.

## UNCLEAR

Insufficient information exists to determine compliance.

Never return PASS when required information is unavailable.

---

# Review Process

Review all supplied ECNs and all supplied drawings before generating findings.

Use exact part number matching unless otherwise specified.

Compare ECN information against drawing information.

Each discrepancy shall generate a finding.

Multiple findings may be returned within the same check.

---

# Check: ECN Drawing Exists

## Purpose

Verify every ECN item has a matching drawing within the package.

## Evaluation

For every ECN:

Locate a drawing where:

```json
drawing.part_num == ecn.part_num
```

PASS when:

All ECN items have matching drawings.

FAIL when:

One or more ECN items do not have matching drawings.

UNCLEAR when:

Part numbers are missing or unreadable.

## Findings

Example:

```text
ECN item 146703B does not have a corresponding drawing in the package.
```

---

# Check: Drawing Description Matches ECN

## Purpose

Verify drawing descriptions match ECN descriptions.

## Evaluation

For each ECN item:

Compare:

```json
ecn.description
```

to

```json
drawing.description
```

PASS when:

Descriptions match exactly.

FAIL when:

Descriptions differ.

UNCLEAR when:

A matching drawing cannot be located.

## Findings

Example:

```text
Drawing description 'BAR,TOW,WELDMENT' does not match ECN description 'BAR,TOW'.
```

---

# Check: Drawing Revision Matches ECN

## Purpose

Verify drawing revisions match ECN revisions.

## Evaluation

For each ECN item:

Compare:

```json
ecn.rev
```

to

```json
drawing.rev
```

PASS when:

Revisions match.

FAIL when:

Revisions differ.

UNCLEAR when:

Revision information is unavailable.

## Findings

Example:

```text
Drawing revision 01 does not match ECN revision 02.
```

---

# Check: ECN Component Exists In Package

## Purpose

Verify components referenced by an ECN are present within the drawing package.

## Evaluation

For each ECN:

Review:

```json
ecn.parts[]
```

Locate matching drawings for all referenced part numbers.

PASS when:

All referenced parts have drawings in the package.

FAIL when:

One or more referenced parts do not have drawings.

UNCLEAR when:

Part number information is incomplete.

## Findings

Example:

```text
ECN references part BN950 but no matching drawing exists in the package.
```

---

# Check: ECN Component Description Match

## Purpose

Verify component descriptions in the ECN match drawing descriptions.

## Evaluation

For every ECN component:

Compare:

```json
ecn.parts[].description
```

against:

```json
drawing.description
```

PASS when:

Descriptions match.

FAIL when:

Descriptions do not match.

UNCLEAR when:

Referenced drawing is missing.

## Findings

Example:

```text
ECN description 'BUMPER,RUBBER' does not match drawing description 'BUMPER RUBBER 16 INCH'.
```

---

# Check: ECN Quantity Matches Assembly BOM

## Purpose

Verify quantities defined by the ECN match quantities found in assembly and weldment BOMs.

## Evaluation

For each ECN component:

Locate matching BOM entries within assemblies and weldments.

Compare:

```json
ecn.parts[].qty
```

against:

```json
drawing.parts[].qty
```

PASS when:

Quantities match.

FAIL when:

Quantities differ.

UNCLEAR when:

Quantity information is unavailable.

## Findings

Example:

```text
ECN quantity 2 does not match assembly BOM quantity 4 for part BN950.
```

---

# Check: ECN Revision Included In Drawing Revision History

## Purpose

Verify the drawing revision history contains the ECN number associated with the current revision.

## Evaluation

For each matching drawing:

Review:

```json
drawing.revtable[]
```

Determine whether:

```json
ecn.ecn
```

appears within the revision table.

PASS when:

The ECN exists in the revision history.

FAIL when:

The ECN is not present in the revision history.

UNCLEAR when:

Revision history is unavailable.

## Findings

Example:

```text
ECN A82419 was not found in the drawing revision history.
```

---

# Check: ECN References Current Drawing Revision

## Purpose

Verify the ECN corresponds to the current revision listed on the drawing.

## Evaluation

Locate the revision table row associated with the ECN.

Compare:

```json
drawing.rev
```

against the revision associated with:

```json
ecn.ecn
```

PASS when:

The ECN represents the current drawing revision.

FAIL when:

The ECN references an older revision.

UNCLEAR when:

Revision history cannot be determined.

## Findings

Example:

```text
ECN A82419 is associated with revision 01 but the drawing is currently revision 02.
```

---

# Overall Status

Determine overall status using the following rules.

## FAIL

If one or more checks return:

```text
FAIL
```

Overall status shall be:

```text
FAIL
```

## UNCLEAR

If no checks return FAIL and one or more checks return:

```text
UNCLEAR
```

Overall status shall be:

```text
UNCLEAR
```

## PASS

If all applicable checks return:

```text
PASS
```

Overall status shall be:

```text
PASS
```

---

# Output Rules

- Return every applicable check.
- Do not omit failed checks.
- Do not invent relationships.
- Use only information contained in supplied Drawing Schemas and ECN Schemas.
- Preserve part numbers exactly.
- Preserve descriptions exactly.
- Preserve revisions exactly.
- Preserve ECN numbers exactly.
- Report each discrepancy as an individual finding.
- When multiple discrepancies occur within one check, include all findings in the same check result.
- Use page numbers from the source schemas whenever available.