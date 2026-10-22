---
name: ecn-information-extraction
description: Extract information from Engineering Change Notices and populate the ECN schema.
---

# Purpose

Extract structured information from Engineering Change Notices (ECNs/ECOs) and populate the ECN schema.

This skill performs information extraction only.

This skill does not:

- Review ECNs
- Evaluate technical correctness
- Evaluate approval status
- Assess change quality
- Assess compliance

Only extract information explicitly contained within the ECN.

---

# Output Schema

Populate the ecn-schema.

Return one schema object for each ECN item identified within the document.

---

# Extraction Rules

Extract only information that is explicitly visible.

Do not infer missing values.

Do not create parts that are not present.

Do not modify engineering identifiers.

Preserve engineering identifiers exactly as shown.

---

# ECN Number

## Field

ecn

Extract the ECN number.

Example:

A82419

Store exactly as shown.

---

# Part Number

## Field

part_num

Extract the affected item number for the current ECN section.

Examples:

146971B
146974B
BN948
BN950

Use the item currently being described on the page.

---

# Description

## Field

description

Extract the item description exactly as shown.

Examples:

GROUP,BUMPER,6450

BAR,TOW,WELDMENT

BUMPER,RUBBER,16L X 2.5W X 2D

Preserve commas and formatting.

Do not truncate.

---

# Revision

## Field

rev

Extract the New Revision value.

If revision information cannot be determined, return an empty string.

Do not infer revisions.

---

# Pages

## Field

pages

Store package page numbers occupied by the ECN item.

Page numbering shall correspond to the complete uploaded document.

Examples:

Page 1

```json
"pages": [1]
```

Page 5

```json
"pages": [5]
```

Multi-page item

```json
"pages": [2,3]
```

Do not store printed page labels.

Store package page positions only.

---

# Parts

## Field

parts

Extract affected child items listed in the table.

For each part extract:

- item_no
- part_num
- description
- qty

---

# Quantity

## Field

qty

Extract quantity from the table.

Use numeric values only.

Do not estimate quantities.

---

# Item Number

## Field

item_no

Extract item number from BOM position when available.

Return 0 if not available.

---

# Identifier Preservation Rule

For:

- ecn
- part_num
- rev
- description

preserve values exactly as displayed.

Do not:

- abbreviate
- remove commas
- remove suffixes
- remove prefixes
- normalize values

---

# Validation

Before returning results:

Verify:

- Valid JSON structure
- Correct data types
- Part numbers preserved exactly
- Descriptions preserved exactly
- All package page numbers assigned correctly
- All BOM rows extracted

If information is unclear:

- Use empty strings
- Use 0
- Use empty arrays

Never guess.