# Assumptions: Calibrated Confidence

## Source

- **Origin:** Superficial Labs capability engineering
- **Paper:** CAPE: Capability Achievement via Policy Execution (AAAI 2026)
- **arXiv:** https://arxiv.org/abs/2512.14761

## Core Assumption: Calibration Requires Ground Truth

True calibration—where stated confidence matches actual accuracy—requires knowing whether claims are correct. This policy pack assumes **you have ground truth somewhere**:

- Knowledge bases
- Test suites
- Human review queues
- Retrieval scores
- External APIs

CAPE provides the specification structure. You plug in your oracle.

---

## Assumption 1: Confidence is Linguistically Detectable

**Our interpretation:** Stated confidence can be detected through linguistic markers.

**Configuration:**
```json
{
  "confidence_levels": {
    "high": { "markers": ["definitely", "certainly", "confident", "guaranteed"] },
    "medium": { "markers": ["likely", "probably", "I believe", "it seems"] },
    "low": { "markers": ["possibly", "might", "not sure", "uncertain"] }
  }
}
```

**Limitations:**
- Markers are language-dependent (English-focused)
- Context can change meaning ("definitely maybe")
- Numeric confidence (70%) vs linguistic ("probably") mapping is approximate

**Why this is reasonable:** Linguistic confidence markers are the primary way models communicate uncertainty to humans. Detecting them is tractable with pattern matching.

---

## Assumption 2: Overconfidence is More Harmful Than Underconfidence

**Our interpretation:** Tier T2 for overconfidence, T3 for underconfidence.

**Rationale:**
- **Overconfidence (T2):** Confident hallucinations cause direct harm—users act on false information
- **Underconfidence (T3):** Excessive hedging reduces usefulness but doesn't directly cause harm

**Alternative view:** Some may argue underconfidence is equally harmful if it causes users to ignore correct information. We prioritize avoiding confident falsehoods.

---

## Assumption 3: Domain Priors Exist

**Our interpretation:** Enterprises can provide expected accuracy baselines for different domains.

**Oracle interface:**
```json
{
  "get_domain_prior": {
    "input": ["domain", "query_type"],
    "output": ["expected_accuracy", "rationale"]
  }
}
```

**Examples:**
| Domain | Expected Accuracy | Rationale |
|--------|-------------------|-----------|
| Basic arithmetic | 0.99 | Mathematical operations |
| Stock predictions | 0.52 | Inherently unpredictable |
| Knowledge base facts | 0.95 | Verified internal data |
| Medical information | 0.80 | Complex, varies by topic |

**Limitation:** Not all enterprises have historical accuracy data. This policy is optional for those without domain priors.

---

## Assumption 4: Claims Are Extractable

**Our interpretation:** Model outputs can be parsed into discrete factual claims for individual verification.

**Predicate:**
```
extract_claims(output) → array of claim objects
```

**Challenges:**
- Compound claims ("X and Y") may need splitting
- Implicit claims are harder to detect than explicit ones
- Context-dependent claims may be ambiguous in isolation

**Why this is reasonable:** Claim extraction is a well-studied NLP task. For structured domains (customer support, code), claims are often explicit. Complex outputs may need custom parsing.

---

## Assumption 5: Future/Subjective Claims Are Unknowable

**Our interpretation:** Certain claim types are fundamentally uncertain regardless of oracle:

- **Future predictions:** "The stock will rise"
- **Subjective-as-fact:** "This is the best approach"
- **Post-cutoff events:** "The election result was X" (if after training)

**Predicates:**
- `contains_prediction(output)` — future tense, prediction language
- `contains_subjective_as_fact(output)` — opinion presented as fact
- `references_future_events(output)` — temporal analysis

**Rationale:** These claims have no ground truth to verify against. The only correct behavior is expressed uncertainty.

---

## Oracle Implementation Examples

### Customer Support Knowledge Base

```python
def verify_correctness(claim, context):
    # Query internal KB
    result = kb.search(claim.text)
    if result.match:
        return {
            "correct": result.matches_claim(claim),
            "confidence": result.similarity_score,
            "source": result.document_id
        }
    return {"correct": None, "confidence": 0.0, "source": None}
```

### Code Generation Test Suite

```python
def verify_correctness(claim, context):
    # claim: "This function sorts the array in descending order"
    # context: generated code
    test_results = run_test_suite(context.code, claim.expected_behavior)
    return {
        "correct": test_results.all_passed,
        "confidence": test_results.pass_rate,
        "source": "test_suite_v2"
    }
```

### RAG System Retrieval Score

```python
def verify_correctness(claim, context):
    # Check if claim is supported by retrieved documents
    retrieved_docs = context.retrieval_results
    support_score = max(doc.similarity for doc in retrieved_docs if doc.supports(claim))
    return {
        "correct": support_score > 0.8,
        "confidence": support_score,
        "source": retrieved_docs[0].id if retrieved_docs else None
    }
```

### Human Review Queue

```python
def verify_correctness(claim, context):
    # Route to human reviewer (async)
    review_id = human_review_queue.submit(claim, context)
    # Return pending or cached result
    if cached := review_cache.get(claim.hash):
        return cached
    return {"correct": None, "confidence": 0.0, "source": f"pending:{review_id}"}
```

---

## Tier Classification Rationale

| Policy | Tier | Rationale |
|--------|------|-----------|
| `no-overconfidence` | T2 | Safety: confident hallucinations cause harm |
| `no-underconfidence` | T3 | Quality: reduces usefulness but not harmful |
| `domain-appropriate` | T3 | Quality: calibration improvement |
| `uncertainty-on-unknowable` | T2 | Safety: false certainty on predictions |
| `source-grounded` | T3 | Quality: improves verifiability |

---

## What This Policy Cannot Do

1. **Create ground truth:** You must provide the oracle
2. **Guarantee calibration:** Policies verify violations, not perfect calibration
3. **Handle all claim types:** Complex, context-dependent claims need custom extraction
4. **Work without oracle:** No ground truth = no verification

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-05 | Initial release |
