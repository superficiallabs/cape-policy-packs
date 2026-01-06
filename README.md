# CAPE Policy Packs

Open-source executable specifications for AI evaluation, verification, and training.

## What's Here

| Pack | Description | Category |
|------|-------------|----------|
| [eu-ai-act-article-14](./eu-ai-act-article-14) | EU AI Act Article 14 Human Oversight requirements as testable policy | Regulatory |
| [calibrated-confidence](./calibrated-confidence) | Train models to express appropriate uncertainty using ground truth oracles | Capability |

## What Are CAPE Policies?

CAPE (Capability Achievement via Policy Execution) turns requirements into executable specifications. Instead of vague guidance like "be helpful" or "express appropriate uncertainty," CAPE policies define testable assertions:

```json
{
  "id": "policy.confidence.no-overconfidence",
  "tier": "T2",
  "assert": [
    { 
      "expr": "NOT (stated_confidence(claim) == 'high' AND oracle.verify_correctness(claim).correct == false)", 
      "msg": "High confidence on incorrect claim" 
    }
  ],
  "on_violation": {
    "action": "CORRECT",
    "correction_hint": "Replace high-confidence language with hedged alternatives."
  }
}
```

## How to Use These

**For Evaluation:**
Run policies against model outputs to measure compliance.

**For Runtime Verification:**
Verify outputs at inference, route violations appropriately.

**For Training:**
Use pass/fail labels to generate preference pairs (DPO) that train capabilities into the model.

## Pack Structure

Each pack includes:

```
/pack-name
├── policy.cpl          # Policy specification (JSON)
├── README.md           # Usage documentation
├── ASSUMPTIONS.md      # Explicit interpretive choices
├── MAPPING.md          # Source → policy traceability
└── examples.json       # Test cases with expected results
```

## Design Principles

These packs follow CAPE design principles:

- **Artifact/output assessment only** — policies verify outputs, not execution
- **No execution harnesses** — static analysis, no runtime dependencies
- **Contains correction data** — every violation includes correction hints
- **Clear assumptions** — interpretive choices are documented
- **JSON format** — machine-readable, schema-validatable
- **No learned verifiers** — deterministic evaluation

## Adding New Packs

We welcome contributions. Each pack should include:

1. `policy.cpl` with kebab-case policy IDs
2. `README.md` explaining usage
3. `ASSUMPTIONS.md` documenting interpretive choices
4. `MAPPING.md` tracing source requirements to predicates
5. `examples.json` with pass, fail, and near-miss test cases

## References

- [CAPE Paper (AAAI 2026)](https://arxiv.org/abs/2512.14761)
- [CAPE Protocol Repository](https://github.com/superficiallabs/cape)
- [Anthropic Skills as CAPE Policies](https://github.com/superficiallabs/skills)
- [Superficial Labs](https://superficiallabs.com)

## License

Apache 2.0
