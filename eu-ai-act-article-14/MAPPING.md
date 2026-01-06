# Mapping: EU AI Act Article 14 Human Oversight

## Source

- **Regulation:** EU AI Act (Regulation (EU) 2024/1689)
- **Article:** Article 14: Human Oversight
- **Version:** Official Journal, June 13, 2024

---

## Constraint Mapping

| # | Regulatory Text | Tier | Predicate | Notes |
|---|-----------------|------|-----------|-------|
| 1 | "remain aware of the possible tendency of automatically relying or over-relying on the output" (14(4)(b)) | T2 | `NOT (high_certainty_language AND low_confidence_topic)` | Linguistic uncertainty as proxy |
| 2 | "decide, in any particular situation, not to use the high-risk AI system or to otherwise disregard, override or reverse" (14(4)(d)) | T2 | `NOT (high_stakes_context AND missing_escalation_signal)` | Explicit escalation signals |
| 3 | "correctly interpret the high-risk AI system's output" (14(4)(c)) | T3 | `has_reasoning_chain OR has_factor_list OR has_evidence_citation` | Reasoning enables interpretation |
| 4 | "properly understand the relevant capacities and limitations" (14(4)(a)) | T2 | `has_limitation_acknowledgment OR has_scope_statement` | Outputs convey limitations |
| 5 | "detecting and addressing anomalies, dysfunctions and unexpected performance" (14(4)(a)) | T2 | `has_anomaly_flag OR has_edge_case_warning` | Model surfaces anomalies |

---

## Non-Verifiable Guidance

| # | Regulatory Text | Reason |
|---|-----------------|--------|
| 1 | "designed and developed in such a way... including with appropriate human-machine interface tools" (14(1)) | System design, not output property |
| 2 | "Human oversight shall aim to prevent or minimise the risks to health, safety or fundamental rights" (14(2)) | Outcome-level goal, not testable per-output |
| 3 | "proportionate to the risks, level of autonomy and context of use" (14(3)) | Requires system-level risk assessment |
| 4 | "verified and confirmed by at least two natural persons" (14(5)) | Process requirement, not output property |

---

## Article 14 Full Text Reference

### 14(1) — Design Requirement
> High-risk AI systems shall be designed and developed in such a way, including with appropriate human-machine interface tools, that they can be effectively overseen by natural persons during the period in which they are in use.

**Mapped to:** Overall policy pack scope

### 14(4)(a) — Understanding Capacities/Limitations + Anomaly Detection
> natural persons to whom human oversight is assigned are enabled... to properly understand the relevant capacities and limitations of the high-risk AI system and be able to duly monitor its operation, including in view of detecting and addressing anomalies, dysfunctions and unexpected performance

**Mapped to:** 
- `policy.article14.capability-boundaries`
- `policy.article14.anomaly-flagging`

### 14(4)(b) — Awareness of Automation Bias
> to remain aware of the possible tendency of automatically relying or over-relying on the output produced by a high-risk AI system (automation bias)

**Mapped to:** `policy.article14.uncertainty-disclosure`

### 14(4)(c) — Output Interpretation
> to correctly interpret the high-risk AI system's output, taking into account, for example, the interpretation tools and methods available

**Mapped to:** `policy.article14.reasoning-transparency`

### 14(4)(d) — Override Capability
> to decide, in any particular situation, not to use the high-risk AI system or to otherwise disregard, override or reverse the output

**Mapped to:** `policy.article14.escalation-routing`

---

## Scope Boundaries

### In Scope (Output Properties)
- Linguistic confidence markers
- Escalation signal presence
- Reasoning chain presence
- Limitation acknowledgments
- Anomaly flags

### Out of Scope (System-Level)
- User interface design
- Logging and audit trails
- Human reviewer assignment
- Process workflows
- Access controls

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-05 | Initial release |
