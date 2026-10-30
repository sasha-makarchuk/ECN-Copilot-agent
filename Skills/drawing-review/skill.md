---
name: drawing-review
description: Review a single drawing represented by a Drawing Schema and identify documentation, revision, and sheet metal drawing issues.
input: drawing-schema

output: drawing-review-schema
---

# Purpose

Review a single drawing using information contained within a Drawing Schema.

This skill evaluates extracted drawing data and identifies documentation issues that can be verified using the schema.

This skill does not:

- Read PDF files directly
- Inspect drawing graphics
- Analyze page layouts
- Verify balloons
- Compare package contents
- Compare multiple drawings
- Compare drawings to ECNs
- Review manufacturing intent

This skill operates exclusively on the supplied Drawing Schema.

---

# Output

Populate the drawing-review-schema.

Return valid schema values whenever information is present.

When information cannot be determined, use:

- Empty string ("") for text values
- 0 for numeric values
- false for boolean values
- Empty array ([]) for collections

Never invent values.

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

Evaluate every applicable check independently.

Each check shall return:

- check
- status
- pages
- findings

Each finding must clearly describe the issue.

---

# Check: Revision Table Exists

## Purpose

Verify revision history exists.

## Evaluation

PASS when:

- revtable contains one or more revision records.

FAIL when:

- revtable is empty.

UNCLEAR when:

- revision information cannot be determined.

## Pages

```json
[1]
```

## Findings

Example:

```text
No revision table entries were found.
```

---

# Check: Revision Row Count

## Purpose

Verify that the revision table contains the expected number of revision rows.

## Evaluation

Determine the numeric revision from:

```json
rev
```

Required row count:

```text
Required Rows = Revision Number + 1
```

Examples:

```text
00 → 1 row
01 → 2 rows
02 → 3 rows
05 → 6 rows
```

Compare against:

```json
revtable.length
```

PASS when:

- Actual row count equals required row count.

FAIL when:

- Actual row count differs from required row count.

UNCLEAR when:

- Revision cannot be interpreted numerically.

## Pages

```json
[1]
```

## Findings

Example:

```text
Revision 02 requires 3 revision rows but only 2 rows were found.
```

---

# Check: Current Revision Present In Revision Table

## Purpose

Verify that the current revision exists in the revision history.

## Evaluation

Compare:

```json
rev
```

against:

```json
revtable[*].rev
```

PASS when:

- Current revision exists in the revision table.

FAIL when:

- Current revision is not found in the revision table.

UNCLEAR when:

- Revision information is incomplete.

## Pages

```json
[1]
```

## Findings

Example:

```text
Current revision 01 is not present in the revision table.
```

---

# Check: Material Specified

## Purpose

Verify material information exists.

## Evaluation
PASS when:

```json
material != ""
```

FAIL when:

```json
material == ""
```

UNCLEAR when:

- Material information cannot be determined

## Pages

Use:

```json
pages
```

## Findings

Example:

```text Material specification is missing.```

---

#Check: Coating Specified

## Purpose

Verify coating information exists.

## Evaluation

PASS when:

```json
coating != ""
```

FAIL when:

```json
coating == ""
```

UNCLEAR when:

- Coating information cannot be determined.

## Pages

Use:

```json
pages
```

## Findings
Example:

```text
Coating specification is missing.
```

---

# Check: Sheet Metal Cut List

## Applies To

Only when

```json
part_category == "SHEET METAL"
```

## Purpose

Verify cut list information exists.

## Evaluation

PASS when:

```json
cl_thickness > 0
cl_length > 0
cl_width > 0
```

FAIL when:

One or more values are missing or equal to zero.

UNCLEAR when:

The existence of cut list information cannot be determined.

## Pages
Use:

```json
pages
```

## Findings

Example:

```text
Cut list width is missing.
```

---

# Check: Flat Pattern Exists

## Applies To
Only when:

```json
part_category == "SHEET METAL"
```

## Purpose

Verify a flat pattern exists.

## Evaluation
PASS when:

```json
fp_exists == true
```

FAIL when:

```json
fp_exists == false
```

UNCLEAR when:

The existence of a flat pattern cannot be determined.

## Pages

Use:

```json
pages
```

## Findings

Example:

```text
Flat pattern view was not identified.
```
---

# Check: Folded View Exists when Bends Exist

## Applies To

Only when:

```json
part_category == "SHEET METAL"
```

and

```json
num_bends > 0
```

## Purpose
Verify a folded view exists when bends are present.

## Evaluation
PASS when:

```json
folder_exists == true ```

FAIL when:

```json
folder_exists == false
```

UNCLEAR when:

Bend information is unavailable.

## Pages
Use:

```json
pages
```

## Findings

Example:

```text
Part contains bends but no folded view was identified.
```

---

# Check: Flat Pattern Bounding Box

## Applies To

Only when:

```json
part_category == "SHEET METAL"
```

## Purpose

Verify the flat pattern includes a bounding box.

## Evaluation

PASS when:

```json
fp_boundingbox == true
```

FAIL when:
```json
fp_boundingbox == false
```

UNCLEAR when:

The presence of a bounding box cannot be determined.

## Pages

Use:

```json
pages
```
## Findings

Example:

```text
Flat pattern bounding box was not identified.
```

---

# Check: Flat Pattern Dimensions Match Cut List

## Applies To

Only when:

```json
part_category == "SHEET METAL"
```

## Purpose

Verify flat pattern dimensions match cut list dimensions.

## Evaluation

Compare:

```json
cl_length
cl_width
```

to

```json
fp_length
fp_width
```

PASS when:

- Length values match
- Width values match

FAIL when:

- Length or width values differ

UNCLEAR when:

- Any required dimension is missing

## Pages

Use:

```json
pages
```

## Findings

Example:

```text
Cut list dimensions are 24.000 x 18.000 while flat pattern dimensions are 24.250 x 18.000.
```

---

# Check: Dangling Dimensions

## Purpose

Verify dangling dimensions were not detected.

## Evaluation

PASS when:

```json
dangling_dim == false
```

FAIL when:

```json
dangling_dim == true
```

UNCLEAR when:

Dimension status cannot be determined.

## Pages

Use:

```json
pages
```

## Findings

Example:

```text
Dangling dimensions were detected.
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
- Do not invent findings.
- Findings must be specific and actionable.
- Pages must identify where the issue occurs whenever possible.
- Use only information present in the supplied Drawing Schema.
- Never assume information that is not available in the schema.