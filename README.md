# Structure Fingerprint Evidence Extensions v0.1

A minimal evidence-object layer for comparing Structure Fingerprints, expressing lineage relations, and preparing evidence for allocation review.

This repository defines a step-by-step bridge from structural fingerprinting to downstream review workflows.

It does **not** determine authorship, origin, plagiarism, infringement, royalty shares, or payment execution.

---

## Overview

Structure Fingerprint records describe evidence about a text or work.

This repository adds the next evidence layers:

```text
Structure-Fingerprint
  ↓
Comparison-Result
  ↓
Lineage-Relation
  ↓
Allocation-Readiness
  ↓
Allocation / Royalty Layer

The central principle is simple:

Similarity is not authorship.
Lineage is not judgment.
Allocation-readiness is not allocation.

Each layer keeps uncertainty explicit and prevents premature enforcement.

Core Principle

This repository preserves a strict separation between:

proof-oriented evidence
inference-oriented evidence
lineage interpretation
allocation review readiness
final allocation or legal decision

In other words, this project defines evidence objects, not verdicts.

Specifications
1. Comparison Result v0.1

comparison-result-v0.1 defines a machine-readable object for comparing two Structure Fingerprint records.

It separates:

proof comparison
inference comparison
uncertainty
review requirements
allowed and prohibited downstream uses

Status:

Implemented as YAML spec, JSON Schema, sample, pass vector, and fail vectors.

Main files:

spec/comparison-result-v0.1.yaml
schemas/comparison-result-v0.1.schema.json
examples/comparison-result.sample.json
tests/vectors/pass/comparison-result.valid.json
tests/vectors/fail/comparison-result.missing-confidence.json
tests/vectors/fail/comparison-result.auto-royalty-assignment-missing-prohibition.json
2. Lineage Relation v0.1

lineage-relation-v0.1 defines a machine-readable object for expressing a relationship between fingerprinted works or records.

It may describe relations such as:

derived_from
adapted_from
influenced_by
structurally_related
independent_convergence
insufficient_evidence

It does not prove authorship, origin, infringement, or allocation entitlement.

Status:

YAML specification draft.

Main file:

spec/lineage-relation-v0.1.yaml
3. Allocation Readiness v0.1

allocation-readiness-v0.1 defines a machine-readable object for deciding whether an evidence package is ready to enter allocation review.

It does not assign royalties or execute payments.

It only answers:

Is this evidence package ready for allocation review?

Status:

YAML specification draft.

Main file:

spec/allocation-readiness-v0.1.yaml
Repository Structure
.
├── spec/
│   ├── comparison-result-v0.1.yaml
│   ├── lineage-relation-v0.1.yaml
│   └── allocation-readiness-v0.1.yaml
│
├── schemas/
│   └── comparison-result-v0.1.schema.json
│
├── examples/
│   └── comparison-result.sample.json
│
├── tests/
│   └── vectors/
│       ├── pass/
│       │   └── comparison-result.valid.json
│       └── fail/
│           ├── comparison-result.missing-confidence.json
│           └── comparison-result.auto-royalty-assignment-missing-prohibition.json
│
└── .github/
    └── workflows/
        └── validate-specs.yml
Start Here

If you are new to this repository, read the files in this order:

1. spec/comparison-result-v0.1.yaml
2. schemas/comparison-result-v0.1.schema.json
3. examples/comparison-result.sample.json
4. tests/vectors/pass/comparison-result.valid.json
5. tests/vectors/fail/comparison-result.missing-confidence.json
6. tests/vectors/fail/comparison-result.auto-royalty-assignment-missing-prohibition.json
7. spec/lineage-relation-v0.1.yaml
8. spec/allocation-readiness-v0.1.yaml

This order follows the evidence flow:

compare → relate → prepare for review
Schema Usage

The current JSON Schema implementation covers:

schemas/comparison-result-v0.1.schema.json

It validates the shape of a Comparison Result object using JSON Schema Draft 2020-12.

The schema checks that:

proof comparison and inference comparison are separated
uncertainty confidence is present
ambiguity level is explicit
automatic authorship claims are prohibited
automatic origin claims are prohibited
automatic royalty assignment is prohibited
automatic legal claims are prohibited
unresolved references cannot have low ambiguity
sensitive comparison classes require human review
Local Validation

Install dependencies:

python -m pip install --upgrade pip
pip install jsonschema

Validate the sample, pass vector, and fail vectors:

python - <<'PY'
import json
import sys
from pathlib import Path

from jsonschema import Draft202012Validator, FormatChecker

schema_path = Path("schemas/comparison-result-v0.1.schema.json")
sample_path = Path("examples/comparison-result.sample.json")
pass_dir = Path("tests/vectors/pass")
fail_dir = Path("tests/vectors/fail")

def load_json(path: Path):
    with path.open("r", encoding="utf-8") as f:
        return json.load(f)

schema = load_json(schema_path)

validator = Draft202012Validator(
    schema,
    format_checker=FormatChecker()
)

def validate_should_pass(path: Path):
    instance = load_json(path)
    errors = sorted(
        validator.iter_errors(instance),
        key=lambda e: list(e.path)
    )

    if errors:
        print(f"ERROR: Expected PASS but validation failed: {path}")
        for error in errors:
            location = ".".join(str(p) for p in error.path) or "<root>"
            print(f"  - {location}: {error.message}")
        return False

    print(f"OK: {path} is valid.")
    return True

def validate_should_fail(path: Path):
    instance = load_json(path)
    errors = sorted(
        validator.iter_errors(instance),
        key=lambda e: list(e.path)
    )

    if not errors:
        print(f"ERROR: Expected FAIL but validation passed: {path}")
        return False

    print(f"OK: {path} failed as expected.")
    for error in errors[:5]:
        location = ".".join(str(p) for p in error.path) or "<root>"
        print(f"  - {location}: {error.message}")
    return True

all_ok = True

print("=== Validating sample ===")
all_ok = validate_should_pass(sample_path) and all_ok

print("\n=== Validating pass vectors ===")
for path in sorted(pass_dir.glob("comparison-result*.json")):
    all_ok = validate_should_pass(path) and all_ok

print("\n=== Validating fail vectors ===")
for path in sorted(fail_dir.glob("comparison-result*.json")):
    all_ok = validate_should_fail(path) and all_ok

if all_ok:
    print("\nValidation succeeded.")
    sys.exit(0)

print("\nValidation failed.")
sys.exit(1)
PY
GitHub Actions Validation

This repository includes a validation workflow:

.github/workflows/validate-specs.yml

The workflow validates:

schemas/comparison-result-v0.1.schema.json
examples/comparison-result.sample.json
tests/vectors/pass/comparison-result.valid.json
tests/vectors/fail/comparison-result.missing-confidence.json
tests/vectors/fail/comparison-result.auto-royalty-assignment-missing-prohibition.json

Expected behavior:

sample file must pass
pass vectors must pass
fail vectors must fail

This ensures that the schema validates both valid and invalid boundary cases.

What JSON Schema Validates

The current schema validates structural requirements such as:

required namespaces
required fields
allowed enum values
score ranges from 0 to 1
URI formats
date-time formats
prohibited downstream use requirements
human review requirement for sensitive comparison classes
What Requires Semantic Validation

Some rules are intentionally left for semantic validation rather than JSON Schema.

Examples:

weighting_profile values should sum to 1.0
overall_similarity should be consistent with component scores
comparison_class should match the configured score thresholds
matched_fields and conflicting_fields should refer to real field paths
confidence should reflect upstream uncertainty

These checks should be implemented later in a semantic validator.

Recommended future file:

tools/validate_semantics.py
Policy Boundary

This repository intentionally prohibits automatic use of these objects for:

authorship claims
origin claims
plagiarism claims
royalty assignment
payment execution
legal claims
irreversible enforcement

The objects may support review, audit, dispute handling, and allocation-readiness assessment.

They must not replace human, institutional, legal, or governance review.

Design Philosophy

The project follows five design principles:

1. Compare before claiming lineage.
2. Preserve uncertainty across layers.
3. Separate evidence from judgment.
4. Keep allocation outside evidence objects.
5. Prevent automatic enforcement from weak evidence.

This creates a safer path from structural similarity to downstream review.

Current Status
Comparison Result v0.1:
  YAML spec: implemented
  JSON Schema: implemented
  sample JSON: implemented
  pass vector: implemented
  fail vectors: implemented
  GitHub Actions validation: implemented

Lineage Relation v0.1:
  YAML spec: draft
  JSON Schema: planned
  examples: planned
  validation vectors: planned

Allocation Readiness v0.1:
  YAML spec: draft
  JSON Schema: planned
  examples: planned
  validation vectors: planned
Roadmap

Recommended next steps:

1. Add schemas/lineage-relation-v0.1.schema.json
2. Add examples/lineage-relation.sample.json
3. Add lineage pass/fail vectors
4. Extend validate-specs.yml for lineage-relation
5. Add schemas/allocation-readiness-v0.1.schema.json
6. Add examples/allocation-readiness.sample.json
7. Add allocation-readiness pass/fail vectors
8. Add semantic validation scripts
Non-Goals

This repository does not:

determine absolute authorship
determine absolute origin
prove infringement
prove plagiarism
assign royalty shares
execute payments
replace legal review
replace institutional governance

It defines evidence objects for structured review.

License

Specification text is released under:

CC-BY-4.0

Code and validation scripts may be released under a separate software license if added later.

Citation

If you reference this repository, cite it as an early draft specification for evidence-object layers built on Structure Fingerprint records.

Suggested citation label:

Structure Fingerprint Evidence Extensions v0.1
