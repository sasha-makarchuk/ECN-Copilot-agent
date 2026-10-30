---
name: package-review
description: Review a package of drawings and validate relationships between drawings, BOMs, assemblies, and weldments.
input: drawing-schema[]
output: package-review-schema
---

# Purpose

Review a package of drawings using Drawing Schemas.

This skill validates relationships between drawings within the package.

This skill does not:

- Read PDF files directly
- Review drawing graphics
- Compare drawings to ECNs
- Perform single-drawing checks already covered by Drawing Review
- Evaluate engineering correctness

This skill only reviews consistency, completeness, and relationships within the drawing package.

---

# Input

Input is an array of Drawing Schemas.

Example:

```json
[
  {
    "part_num": "146974B",
    "part_category": "ASSEMBLY",
    "parts": []
  },
  {
    "part_num": "146703B",
    "part_category": "WELDMENT"
  }
]
```

---

# Output Schema

```json
{
  "type": "package_review",
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

Never return PASS when required information is missing.

---

# Review Process

Review the entire package before generating findings.

Use all supplied Drawing Schemas.

Package checks may compare drawings against each other.

---

# Check: Duplicate Part Numbers

## Purpose

Verify each drawing number appears only once within the package.

## Evaluation

PASS when:

Every drawing part number is unique.

FAIL when:

The same part number appears multiple times.

UNCLEAR when:

Part numbers are missing.

## Findings

List all duplicate part numbers.

Example:

```text
Part number 146703B appears multiple times in the package.
```

---

# Check: Conflicting Revisions

## Purpose

Verify the package does not contain multiple revisions of the same part.

## Evaluation

PASS when:

Each part number exists at only one revision level.

FAIL when:

The same part number exists with different revisions.

UNCLEAR when:

Revision information is unavailable.

## Findings

Example:

```text
Part number 146703B exists as revision 00 and revision 01.
```

---

# Check: Referenced Component Drawing Exists

## Purpose

Verify all referenced BOM components requiring drawings have a matching drawing in the package.

## Applies To

ASSEMBLY

WELDMENT

## Evaluation

For every item in:

```json
parts[]
```

attempt to find a Drawing Schema with a matching:

```json
part_num
```

PASS when:

All required component drawings exist.

FAIL when:

One or more required component drawings are missing.

UNCLEAR when:

The requirement cannot be determined.

## Exclusions

Do not require drawings for components clearly identified as:

- Commercial hardware
- Standard hardware
- Reference items

If uncertain, return UNCLEAR.

## Findings

Example:

```text
Assembly 146974B references part BN950 but no matching drawing exists in the package.
```

---

# Check: BOM Description Match

## Purpose

Verify BOM descriptions match title block descriptions on referenced drawings.

## Applies To

ASSEMBLY

WELDMENT

## Evaluation

For every BOM item:

1. Find matching drawing by part number.
2. Compare:

```json
parts[].description
```

to

```json
drawing.description
```

PASS when:

Descriptions match.

FAIL when:

Descriptions differ.

UNCLEAR when:

Matching drawing cannot be located.

## Findings

Example:

```text
Assembly 146974B item BN950 description 'BUMPER,RUBBER' does not match drawing description 'BUMPER RUBBER 16 INCH'.
```

---

# Check: Empty Assembly BOM

## Purpose

Verify assemblies and weldments contain component items.

## Applies To

ASSEMBLY

WELDMENT

## Evaluation

PASS when:

```json
bom_exists == false
```

or

```json
parts.length > 0
```

FAIL when:

```json
bom_exists == true
```

and

```json
parts.length == 0
```

UNCLEAR when:

BOM status cannot be determined.

## Findings

Example:

```text
Assembly drawing contains a BOM but no BOM items were extracted.
```

---

# Check: Orphan Drawings

## Purpose

Identify drawings that are not referenced by any assembly or weldment.

## Evaluation

PASS when:

Every drawing is referenced by at least one higher-level assembly.

FAIL when:

Drawings exist that are not referenced anywhere.

UNCLEAR when:

Package structure cannot be determined.

## Exclusions

Exclude:

- Top-level assemblies
- Standalone manufactured parts
- Standalone purchased items

If drawing purpose cannot be determined, mark UNCLEAR.

## Findings

Example:

```text
Drawing BN950 was not referenced by any assembly in the package.
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
- Use only information contained in Drawing Schemas.
- Preserve part numbers exactly.
- Preserve descriptions exactly.
- Preserve revisions exactly.
- Report each discrepancy as an individual finding.
- When multiple discrepancies occur within one check, include all findings in the same check result.