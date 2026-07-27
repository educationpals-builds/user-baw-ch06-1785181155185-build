# Clause Walk Prompt Pack

Five standalone prompts for BLOCK framework clause checking. Each prompt is self-contained and can be used in any chat model. Paste your code/config after the prompt.

---

## Prompt 1: BOUNDARIES (B)

```
You are auditing training code for BOUNDARY violations — places where task or domain boundaries are poorly maintained, leading to instruction confusion.

Look for:
- Domain mixing frequency (per-batch switching is risky)
- Task format inconsistency across domains
- Instruction template variations
- Context window fragmentation

I will paste code or config. Return exactly:

```yaml
clause: B (Boundaries)
finding: [BLOCK|CAUTION|CLEAR]
cited_lines: "filename:line_numbers"
evidence: |
  [relevant excerpt]
analysis: |
  [what boundary issue exists or why it's clear]
fix: |
  [specific fix if needed, or "N/A - boundaries well-maintained"]
```

Code/config to audit:

[PASTE HERE]
```

---

## Prompt 2: LEAKAGE (L)

```
You are auditing training code for DATA LEAKAGE — contamination between training and evaluation data that produces false performance signals.

Look for:
- Random splits without deduplication (leaks near-duplicates)
- Missing fuzzy/semantic deduplication
- Eval data from same time period as training
- Benchmark examples in training corpus
- Shared preprocessing that could leak statistics

I will paste code or config. Return exactly:

```yaml
clause: L (Leakage)
finding: [BLOCK|CAUTION|CLEAR]
cited_lines: "filename:line_numbers"
evidence: |
  [relevant excerpt]
analysis: |
  [what leakage path exists or why it's clear]
fix: |
  [specific fix if needed, or "N/A - proper isolation confirmed"]
```

Code/config to audit:

[PASTE HERE]
```

---

## Prompt 3: OVERFIT (O)

```
You are auditing training code for OVERFIT RISK — missing controls that allow the model to memorize training data rather than generalize.

Look for:
- Missing early stopping
- Infrequent or missing evaluation
- No learning rate decay/scheduling
- Missing regularization (dropout, weight decay)
- Training for fixed steps without validation checks
- Checkpoint selection by training loss only

I will paste code or config. Return exactly:

```yaml
clause: O (Overfit)
finding: [BLOCK|CAUTION|CLEAR]
cited_lines: "filename:line_numbers"
evidence: |
  [relevant excerpt]
analysis: |
  [what overfit risk exists or why it's controlled]
fix: |
  [specific fix if needed, or "N/A - overfit controls adequate"]
```

Code/config to audit:

[PASTE HERE]
```

---

## Prompt 4: COLLAPSE (C)

```
You are auditing model architecture for REPRESENTATION COLLAPSE — structural patterns that cause embeddings to lose distinctiveness during training, especially under domain shift.

Look for:
- Normalization AFTER pooling (washes out features)
- Missing per-domain statistics
- Aggressive pooling without residuals
- No embedding space monitoring
- Uniform initialization in late layers

KNOWN FAILURE PATTERN: Post-pooling LayerNorm causes instruction drift at ~2k steps on mixed domain data. The normalization erases domain-distinctive features after they've been pooled together.

I will paste code or config. Return exactly:

```yaml
clause: C (Collapse)
finding: [BLOCK|CAUTION|CLEAR]
cited_lines: "filename:line_numbers"
evidence: |
  [relevant excerpt]
analysis: |
  [what collapse risk exists or why architecture is safe]
fix: |
  [specific fix if needed, or "N/A - representation stability maintained"]
```

Code/config to audit:

[PASTE HERE]
```

---

## Prompt 5: KILL-SWITCH (K)

```
You are auditing training infrastructure for KILL-SWITCH adequacy — the ability to detect failure, halt training, and recover to a good state.

Look for:
- Metric-based halt conditions (not just step count)
- Checkpoint retention (keeping N recent, not just latest)
- Rollback capability to previous checkpoint
- Emergency save on halt
- Alerting/notification on threshold breach
- Graceful shutdown handling

I will paste code or config. Return exactly:

```yaml
clause: K (Kill-switch)
finding: [BLOCK|CAUTION|CLEAR]
cited_lines: "filename:line_numbers"
evidence: |
  [relevant excerpt]
analysis: |
  [what kill-switch gap exists or why recovery is assured]
fix: |
  [specific fix if needed, or "N/A - halt and recovery adequate"]
```

Code/config to audit:

[PASTE HERE]
```

---

## Usage Notes

1. **Copy one prompt** into your chat interface
2. **Replace `[PASTE HERE]`** with your actual code/config
3. **Review the finding** — BLOCK means stop, CAUTION means fix soon, CLEAR means verified safe
4. **Apply fixes** and re-run to confirm resolution

## Full Sweep

To run all five clauses, use each prompt sequentially on the relevant files:
- Boundaries → training config, data loader
- Leakage → data pipeline, split logic
- Overfit → training loop, scheduler
- Collapse → model architecture, forward pass
- Kill-switch → checkpoint manager, training harness

---

*Prompt pack generated for BLOCK framework pre-flight bench*