---
name: drawing-information-extraction
description: Extract information from engineering drawings and populate the Drawing schema.
input: PDF file
output: drawing-schema
---

# Drawing Information Extraction

## Purpose

Extract structured information from engineering drawings and populate the Drawing schema.

The goal of this skill is information extraction only.

This skill does not:

- Review drawings
- Evaluate correctness
- Check standards compliance
- Perform engineering analysis
- Determine approval status

This skill only identifies information explicitly present in the drawing and stores it in the schema.

---

# Input

Input may include:

- PDF drawings
- Multi-sheet drawings
- Mechanical drawings
- Fabrication drawings
- Weldments
- Sheet metal drawings
- Assembly drawings

---

# Output

Populate the drawing-schema.

Return valid schema values whenever information is present.

When information cannot be determined, use:

- Empty string ("") for text values
- 0 for numeric values
- false for boolean values
- Empty array ([]) for collections

Never invent values.

---

# Extraction Rules

## General

Extract only information that can be observed directly from the drawing.

Do not infer values from assumptions.

Do not estimate dimensions.

Do not fabricate metadata.

If conflicting information exists, report the most recent revision information and record uncertainty in notes if supported by the schema.

---

# Part Number

## Field

part_num

## Source Priority

1. Title block
2. Drawing title area
3. Drawing metadata
4. File metadata

Extract the primary drawing part number.

Do not extract child component part numbers.

---

# Description

## Field

description

Extract the official part description from:

- Title block
- Drawing header

Preserve original wording.

---

# Weight

## Field

weight

Extract drawing weight if explicitly listed.

Do not calculate weight.

Do not estimate weight.

If multiple weights exist, use the finished part weight.

---

# Revision

## Field

rev

Extract current released revision.

Common locations:

- Title block

---

# Revision Table

## Field

revtable

Extract complete revision history text when available. Each row except header should include:

- Revision levels
- Revision descriptions
- Dates
- Names
- ECN

Preserve original content as closely as possible.

## Revision Table Normalization

When extracting revision table data, normalize revision values before storing them in the schema.

If the latest revision row contains:

```text
**
```

replace it with the next sequential revision value.

### Single Revision Row

If the revision table contains a single row and the revision value is:

```text
**
```

store:

```text
00
```

Example:

```text
**
```

becomes:

```text
00
```

### Multiple Revision Rows

If the latest revision row contains:

```text
**
```

determine the previous revision and increment it by one.

Examples:

```text
00
01
**
```

becomes:

```text
00
01
02
```

```text
00
01
02
**
```

becomes:

```text
00
01
02
03
```

### Formatting

Preserve leading zeros.

Examples:

```text
00 -> 01 -> 02
```

```text
08 -> 09 -> 10
```

The extracted schema shall not contain:

```text
**
```

when a sequential revision can be determined.

Normalize the revision value before populating:

```json
revtable[].rev
```

---

# ECN

## Field

ecn

Extract:

- ECN

numbers associated with the current revision.

Use latest applicable change record.

---

# Material

## Field

material

## Purpose

Extract the complete material specification exactly as shown on the drawing.

Preserve all material codes, commas, dashes, suffixes, prefixes, and specification identifiers.

Material information may consist of multiple codes separated by:

- commas
- semicolons
- spaces
- slashes
- line breaks

These codes collectively define the material specification and shall be captured as a single value.

---

## Rules

Extract the entire material field.

Do not truncate.

Do not simplify.

Do not normalize.

Do not remove codes.

Do not remove prefixes or suffixes.

Do not attempt to interpret the meaning of individual material codes.

Preserve the original sequence.

---

Examples:

- 44-50
- 100,RS313
- 400A
- 450B
- N/A

---

# Coating

## Field

coating

Extract:

- Paint specification
- Plating
- Powder coat
- Galvanizing
- Surface treatment

Do not leave empty unless the field is blank. Extract exactly what the field says.

---

# Part Category

## Field

part_category

Allowed values:

- ASSEMBLY
- SHEET METAL
- OTHER

### ASSEMBLY

Use when:

- BOM exists
- Item balloons exist
- Assembly views exist
- Multiple parts are defined

### SHEET METAL

Use when:

- Flat pattern present
- Bend notes present
- Bend tables present
- Sheet metal callouts exist

### OTHER

All remaining drawing types.

Only return valid enum values.

---

# Pages

## Field

pages

## Purpose

Store the page numbers occupied by this drawing within the complete document package.

These page numbers represent the drawing's position within the entire uploaded package, not the drawing's internal sheet numbering.

---

## Rules

Review the entire package before assigning page numbers.

Assign page numbers sequentially based on the order pages appear in the package.

Each drawing shall contain an array of all package page numbers associated with that drawing.

Use package page numbering starting at 1.

---

## Examples

### Example 1

Package contents:

Drawing A = 2 pages
Drawing B = 1 page
Drawing C = 3 pages

Output:

Drawing A

```json
"pages": [1, 2]
```

Drawing B

```json
"pages": [3]
```

Drawing C

```json
"pages": [4, 5, 6]
```

---

# Sheet Metal Information

## Dimensional Data

All dimensional values shall be stored as decimal inches.

Examples:

1/8       -> 0.125
24        -> 24.0
24.125    -> 24.125
610 mm    -> 24.016

Do not store units.

Convert dimensions to inches when units are clearly indicated.

If units cannot be determined confidently, assume inches.

## Cut List

Extract from cutlist table in top right corner:

- Cutlist thickness
- Cutlist length
- Cutlist width

Only when values are explicitly available.

---

## Flat Pattern

Populate:

- Flat pattern length
- Flat pattern width

Only when dimensions are explicitly present on flat pattern view.

Do not calculate.

---

# Bend Count

## Field

num_bends

Count bends only when bend information is explicitly represented.

Sources:

- Flat pattern tables
- Bend tables
- Sheet metal notes
- Bend callouts

Do not estimate.

---

# Flat Pattern Detection

## Field

fp_exists

Set TRUE when:

- Flat pattern view exists
- Flat pattern table exists
- Flat pattern identified by note

Otherwise FALSE.

---

# Folded View Detection

## Field

folded_exists

Set TRUE only when a folded/folding view is explicitly present.

Otherwise FALSE.

---

# Flat Pattern Note Detection

## Field

fp_note

Set TRUE when flat pattern notes exist.

Examples:

- CUT VIEW
- CUT PROFILE

Otherwise FALSE.

---

# Flat Pattern Bounding Box

## Field

fp_boundingbox

Set TRUE when flat pattern bounding box is explicitly shown.

Otherwise FALSE.

---

# Dangling Dimensions

## Field

dangling_dim

Set TRUE when dimensions are visibly unattached, broken, dangling, or are not in black font in the drawing.

Set FALSE otherwise.

Only use visible evidence.

---

# Assembly Components

## Field

parts

Extract all BOM items.

For each item extract:

- part_num
- description
- qty
- item_no
- weight

---

## Rule

The presence of a Bill of Materials determines whether component information should be extracted. if BOM is present, extract all listed items into the parts array.

Part category does not affect BOM extraction.

Whenever a BOM, Parts List, Weldment Cut List, or Component Table is present, extract all listed items into the parts array.

---

## BOM Extraction Priority

Use:

1. Bill of Materials table

Do not create components that are not listed.

---

# Component Weight

Populate component weight only when explicitly listed.

Do not calculate.

Do not estimate.

---

# Multi-Sheet Drawings

Review every sheet before populating schema.

Information may be distributed across multiple sheets.

Use the released drawing as a whole.

---

# Confidence Rules

If information is unclear:

- Prefer blank values
- Prefer FALSE
- Prefer 0

Never guess.

Missing information is preferable to incorrect information.

---

# Validation Rules

Before returning output:

Verify:

- Valid JSON structure
- Valid enum values
- Numeric fields contain numbers
- Boolean fields contain booleans
- Arrays contain valid records
- No hallucinated values
- If:
  ```json
  "bom_exists": true
  ```
  Then: parts array must contain at least one item.

The extracted schema must represent only information contained within the drawing.