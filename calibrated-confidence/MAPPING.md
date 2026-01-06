# Mapping: Calibrated Confidence

## Source

- **Origin:** Superficial Labs capability engineering
- **Paper:** CAPE: Capability Achievement via Policy Execution (AAAI 2026)
- **Concept:** Verification-based training for confidence calibration

---

## Constraint Mapping

| # | Capability Requirement | Tier | Predicate | Notes |
|---|------------------------|------|-----------|-------|
| 1 | No confident hallucinations | T2 | `NOT (stated_confidence == 'high' AND oracle.correct == false)` | Oracle verifies correctness |
| 2 | No excessive hedging on correct claims | T3 | `NOT (stated_confidence == 'low' AND oracle.correct == true AND oracle.confidence > threshold)` | Oracle confirms correctness |
| 3 | Domain-appropriate confidence | T3 | `abs(stated - domain_prior) <= tolerance` | Oracle provides domain priors |
| 4 | Uncertainty on unknowable claims | T2 | `NOT (high_confidence AND (prediction OR subjective_as_fact))` | Claim type detection |
| 5 | Source grounding for high confidence | T3 | `high_confidence → (has_source OR has_reasoning)` | Pattern matching |

---

## Predicate Implementation

### CAPE-Provided (Standard NLP)

| Predicate | Implementation | Complexity |
|-----------|---------------|------------|
| `stated_confidence(text)` | Keyword matching against configured markers | Low |
| `extract_claims(text)` | Sentence splitting + claim identification | Medium |
| `contains_prediction(text)` | Future tense detection, prediction keywords | Low |
| `contains_subjective_as_fact(text)` | Opinion markers without hedging | Medium |
| `has_source_attribution(text)` | Citation patterns, "according to" | Low |
| `has_reasoning_chain(text)` | Causal connectors: "because", "therefore" | Low |
| `has_uncertainty_acknowledgment(text)` | Hedging language presence | Low |

### Enterprise-Provided (Oracle)

| Predicate | Implementation | Notes |
|-----------|---------------|-------|
| `oracle.verify_correctness(claim, ctx)` | Domain-specific ground truth lookup | Required |
| `oracle.get_domain_prior(domain, type)` | Historical accuracy or expert estimate | Optional |

---

## Oracle Interface Specification

### verify_correctness

**Input:**
```json
{
  "claim": {
    "text": "Your subscription renews on March 15th",
    "type": "factual",
    "entities": ["subscription", "March 15th"]
  },
  "context": {
    "user_id": "12345",
    "domain": "customer_support"
  }
}
```

**Output:**
```json
{
  "correct": true,
  "confidence": 0.95,
  "source": "billing_system.subscriptions"
}
```

**Return values:**
- `correct: true` — claim verified as correct
- `correct: false` — claim verified as incorrect
- `correct: null` — unable to verify (no ground truth)

### get_domain_prior

**Input:**
```json
{
  "domain": "stock_predictions",
  "query_type": "predictive"
}
```

**Output:**
```json
{
  "expected_accuracy": 0.52,
  "rationale": "Historical prediction accuracy on financial forecasts"
}
```

---

## Failure Modes by Domain

| Domain | Primary Failure Mode | Oracle Type |
|--------|---------------------|-------------|
| Customer Support | Confident wrong account info | KB lookup |
| Code Generation | "This works" but tests fail | Test suite |
| RAG Systems | Confident unsupported claim | Retrieval score |
| Medical Info | Confident incorrect dosage | Expert review |
| Legal Info | Confident wrong statute | Document verification |
| Predictions | Any confidence on unknowable | N/A (always uncertain) |

---

## Configuration Reference

```json
{
  "configuration": {
    "confidence_levels": {
      "high": { "min": 0.8, "max": 1.0 },
      "medium": { "min": 0.5, "max": 0.8 },
      "low": { "min": 0.0, "max": 0.5 }
    },
    "correctness_threshold": 0.7,
    "calibration_tolerance": 0.2
  }
}
```

| Parameter | Default | Description |
|-----------|---------|-------------|
| `correctness_threshold` | 0.7 | Oracle confidence needed to consider claim "verified" |
| `calibration_tolerance` | 0.2 | Allowed deviation from domain prior |

---

## Training Signal Mapping

| Policy | Violation Label | Preference Direction | Severity |
|--------|-----------------|---------------------|----------|
| `no-overconfidence` | `overconfident_incorrect` | reject | high |
| `no-underconfidence` | `underconfident_correct` | reject | medium |
| `domain-appropriate` | `domain_miscalibrated` | reject | medium |
| `uncertainty-on-unknowable` | `certain_on_unknowable` | reject | high |
| `source-grounded` | `ungrounded_confidence` | reject | low |

Use these labels to generate DPO preference pairs:
- **Chosen:** Outputs passing all policies
- **Rejected:** Outputs violating policies (weighted by severity)

---

## Scope Boundaries

### In Scope
- Linguistic confidence detection
- Claim extraction (standard patterns)
- Oracle integration interface
- Calibration tolerance configuration

### Out of Scope
- Ground truth generation (enterprise provides)
- Complex claim parsing (custom implementation)
- Real-time oracle latency management
- Multi-language support (English-focused)

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-05 | Initial release |
