---
name: engineering-review-workflow
description: Execute the complete engineering drawing and ECN review workflow and produce a Combined Review Schema.

when_to_use:
  - User requests drawing review
  - User requests drawing package review
  - User requests package review
  - User requests ECN review
  - User requests ECN package review
  - User requests release validation
  - User requests drawing validation
  - User requests engineering package validation
---

# Purpose

Execute the complete engineering review workflow.

This skill is the primary entry point for all drawing package and ECN package reviews.

This skill coordinates extraction, review, and aggregation activities and produces the final Combined Review Schema.

---

# Skill Priority

This is the preferred review skill.

When a user requests:

- Review drawing
- Review drawing package
- Review assembly package
- Review weldment package
- Review fabrication package
- Review ECN package
- Validate release package
- Validate drawings
- Check drawings for errors
- Verify package completeness
- Verify ECN implementation

Use this skill.

Do not execute Drawing Review, Package Review, ECN Review, or Review Aggregator directly unless specifically requested by the user.

---

# Accepted Inputs

Inputs may include:

- Single drawing PDF
- Drawing package PDF
- Multiple drawing PDFs
- Single ECN PDF
- Multiple ECN PDFs
- Drawing package and ECN package
- Merged PDF containing drawings and ECNs

Packages may contain:

- Drawings only
- Drawings and ECNs

---

# Workflow Selection

Determine whether ECNs are present.

If ECNs are not present:

Execute the Drawing Package Review Workflow.

If ECNs are present:

Execute the ECN Package Review Workflow.

---

# Drawing Package Review Workflow

Execute the following skills in order:

1. Drawing Information Extraction
2. Drawing Review
3. Package Review
4. Review Aggregator

Detailed execution:

1. Execute Drawing Information Extraction.
2. Generate Drawing Schema objects for all drawings.
3. Execute Drawing Review for each Drawing Schema.
4. Execute Package Review using all Drawing Schemas.
5. Execute Review Aggregator using all review outputs.
6. Generate a Combined Review Schema.

---

# ECN Package Review Workflow

Execute the following skills in order:

1. Drawing Information Extraction
2. ECN Information Extraction
3. Drawing Review
4. Package Review
5. ECN Review
6. Review Aggregator

Detailed execution:

1. Execute Drawing Information Extraction.
2. Execute ECN Information Extraction.
3. Generate Drawing Schema[] and ECN Schema[].
4. Execute Drawing Review for each Drawing Schema.
5. Execute Package Review using all Drawing Schemas.
6. Execute ECN Review using Drawing Schema[] and ECN Schema[].
7. Execute Review Aggregator using all review outputs.
8. Generate a Combined Review Schema.

---

# Output

Always return:

Combined Review Schema

Do not return:

- Drawing Schema
- ECN Schema
- Drawing Review Schema
- Package Review Schema
- ECN Review Schema

Intermediate schemas are internal workflow artifacts.

Only the Combined Review Schema shall be returned.

---

# Processing Rules

Follow the workflow exactly.

Execute all required extraction skills before review skills.

Execute all applicable review skills before Review Aggregator.

Do not skip workflow steps.

Do not perform review logic outside the designated review skills.

Do not generate findings directly.

Use review skill outputs when generating the Combined Review Schema.

---

# Data Integrity Rules

Always:

- Preserve engineering identifiers
- Preserve part numbers
- Preserve descriptions
- Preserve revisions
- Preserve page mappings
- Preserve findings
- Preserve statuses

Never:

- Guess values
- Invent values
- Modify review findings
- Remove findings
- Create new findings

Missing information is preferable to incorrect information.

---

# Success Criteria

Success is achieved when:

- Correct document types are identified.
- Correct workflow is selected.
- Required extraction skills are executed.
- Required review skills are executed.
- Review Aggregator is executed.
- Combined Review Schema is produced.
- No information is invented.
- Traceability is maintained.
- All reviews are based on extracted schema data.