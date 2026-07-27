# METHOD: The BLOCK Framework

## Framework Name

**BLOCK** — A five-clause pre-flight assessment framework for open model fine-tuning.

## Acronym Expansion

| Letter | Clause | Question Answered |
|--------|--------|-------------------|
| **B** | **B**oundaries | Are task and domain boundaries maintained during training? |
| **L** | **L**eakage | Is train/eval data contamination prevented? |
| **O** | **O**verfit | Are overfit controls in place and calibrated? |
| **C** | **C**ollapse | Will representations remain distinctive under training pressure? |
| **K** | **K**ill-switch | Can training be halted and rolled back safely? |

## Framework Philosophy

Fine-tuning failures are predictable. They follow patterns. BLOCK codifies the five most common failure modes into a pre-flight checklist that catches problems before GPU hours are wasted.

## Clause Details

### B — Boundaries

**Failure Mode**: The model loses track of what task it's performing.

**Mechanism**: When domains switch too frequently during training, the model cannot establish stable instruction-following patterns. Each batch pulls representations in a different direction.

**Detection**: Look for per-batch domain switching, inconsistent instruction formats, context window fragmentation.

**Prevention**: Cluster batches by domain (50-100 consecutive), standardize instruction templates, maintain format consistency.

---

### L — Leakage

**Failure Mode**: Eval metrics lie because eval data leaked into training.

**Mechanism**: Near-duplicate examples, shared preprocessing statistics, or temporal overlap between train and eval sets cause inflated performance that doesn't generalize.

**Detection**: Check for fuzzy deduplication, temporal splits, benchmark isolation.

**Prevention**: SimHash or semantic deduplication at 0.90+ threshold, temporal or semantic splits (not random), held-out benchmark quarantine.

---

### O — Overfit

**Failure Mode**: The model memorizes training data instead of learning patterns.

**Mechanism**: Without early stopping, regularization, or frequent evaluation, the model will eventually fit noise in the training data.

**Detection**: Look for missing early stopping, infrequent eval, no regularization, fixed-step training without validation.

**Prevention**: Eval every 250-500 steps, patience-based early stopping, appropriate weight decay, checkpoint selection by validation metric.

---

### C — Collapse

**Failure Mode**: Representations become uniform and lose task-relevant information.

**Mechanism**: Certain architectural patterns — especially post-pooling normalization — cause embeddings to converge to similar values, erasing the distinctive features needed for task discrimination.

**Detection**: Look for normalization after pooling, aggressive mean pooling, missing residual connections in late layers.

**Prevention**: Pre-pooling normalization, per-domain batch statistics, embedding space monitoring, residual connections.

---

### K — Kill-switch

**Failure Mode**: Training degrades but continues, wasting resources and producing a bad model.

**Mechanism**: Without metric-based halting and rollback capability, a training run that goes off the rails will run to completion, and recovery requires starting over.

**Detection**: Look for step-count-only termination, single checkpoint retention, no metric thresholds, missing alerts.

**Prevention**: Metric-based halt conditions, retain last N checkpoints, implement rollback, emergency save on halt, alerting.

---

## Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| **BLOCK** | Critical issue that will cause training failure | Must fix before proceeding |
| **CAUTION** | Significant risk that may cause problems | Should fix, or accept risk explicitly |
| **CLEAR** | Clause requirements satisfied | Verified safe, document evidence |

## Application Protocol

1. **Gather artifacts**: training config, data pipeline, model architecture, checkpoint manager
2. **Run each clause**: Apply clause-specific checks to relevant artifacts
3. **Cite evidence**: Every finding must reference specific lines/keys
4. **Aggregate severity**: Any BLOCK = hold launch; CAUTION-only = proceed with monitoring
5. **Design tripwire**: Configure runtime monitoring based on identified risks
6. **Document**: Charter captures full audit trail

## Framework Scope

BLOCK is designed for:
- Open model fine-tuning (not pre-training)
- Instruction tuning and task adaptation
- Runs of 1k-50k steps
- Single-node to small-cluster training

BLOCK does not cover:
- Large-scale pre-training
- Reinforcement learning from feedback
- Inference deployment
- Model merging/editing

---

*BLOCK framework v1.0 — Letters appear only in this document*