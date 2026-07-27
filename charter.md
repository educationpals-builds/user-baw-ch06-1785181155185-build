# Pre-Flight Charter: open-llama-finetune-candidate

## Specimen Under Review

**Model**: open-llama-finetune-candidate  
**Planned Training**: 5,000 steps on mixed-domain instruction data  
**Identified Break Point**: Instruction drift after 2k steps on mixed domain data  
**Review Date**: Pre-training gate  
**Reviewer**: Automated BLOCK bench + human sign-off required

---

## Standard Applied

BLOCK Framework v1.0 — Five-clause pre-flight assessment for open model fine-tuning safety and stability.

---

## Clause Findings

### 1. BOUNDARIES (B)

**Finding**: CAUTION  
**Cited Lines**: `training_config.yaml:47-52`  
**Key Evidence**:
```yaml
# Lines 47-52
domain_mixing:
  strategy: uniform_random
  domains: [code, chat, reasoning, creative, factual]
  switch_frequency: per_batch
```
**Analysis**: Per-batch domain switching with uniform random selection creates unstable task boundaries. The model cannot establish consistent instruction-following patterns when domain context shifts every batch.

**Recommendation**: Implement domain clustering — group 50-100 batches per domain before switching.

---

### 2. LEAKAGE (L)

**Finding**: CLEAR (earned)  
**Cited Lines**: `data_pipeline.py:112-118`  
**Key Evidence**:
```python
# Lines 112-118
def split_data(dataset):
    # Deduplication by exact match AND fuzzy hash
    deduped = deduplicate(dataset, method='simhash', threshold=0.92)
    # Temporal split: eval data from later timestamp
    train, eval = temporal_split(deduped, eval_ratio=0.1)
    return train, eval
```
**Analysis**: Proper deduplication and temporal splitting prevents train/eval leakage. SimHash threshold of 0.92 catches near-duplicates.

---

### 3. OVERFIT (O)

**Finding**: CAUTION  
**Cited Lines**: `training_loop.py:78-84`  
**Key Evidence**:
```python
# Lines 78-84
scheduler = CosineAnnealingLR(
    optimizer,
    T_max=5000,
    eta_min=1e-7
)
# No early stopping configured
# No eval frequency specified
```
**Analysis**: No early stopping mechanism. Eval frequency unspecified. At 2k+ steps with mixed domains, overfit to dominant domain likely without intervention checkpoints.

**Recommendation**: Add eval every 250 steps, implement patience-based early stopping with patience=3.

---

### 4. COLLAPSE (C)

**Finding**: BLOCK  
**Cited Lines**: `model_arch.py:203-209`  
**Key Evidence**:
```python
# Lines 203-209
class InstructionHead(nn.Module):
    def forward(self, x):
        # LayerNorm AFTER attention pooling
        pooled = self.attention_pool(x)
        normed = self.layer_norm(pooled)  # <-- normalization placement
        return self.output_proj(normed)
```
**Analysis**: Post-pooling normalization placement creates representation collapse risk under domain shift. When instruction embeddings from different domains are pooled then normalized, distinctive features wash out. This is the primary driver of instruction drift at 2k steps.

**Recommendation**: Move LayerNorm before pooling, or add per-domain normalization statistics.

---

### 5. KILL-SWITCH (K)

**Finding**: CLEAR (earned)  
**Cited Lines**: `checkpoint_manager.py:34-45`  
**Key Evidence**:
```python
# Lines 34-45
class CheckpointManager:
    def __init__(self, config):
        self.rollback_enabled = True
        self.keep_last_n = 5
        self.metric_threshold = config.get('halt_threshold', 0.65)
    
    def should_halt(self, metrics):
        if metrics['instruction_adherence'] < self.metric_threshold:
            self.save_emergency_checkpoint()
            return True
        return False
```
**Analysis**: Kill-switch properly configured with metric-based halt, emergency checkpoint save, and rollback capability.

---

## Severity Story

```
CLAUSE      | STATUS  | SEVERITY | BLOCKS LAUNCH?
------------|---------|----------|---------------
Boundaries  | CAUTION | Medium   | No (fixable)
Leakage     | CLEAR   | —        | No
Overfit     | CAUTION | Medium   | No (fixable)
Collapse    | BLOCK   | High     | YES
Kill-switch | CLEAR   | —        | No
```

**Aggregate**: 1 BLOCK, 2 CAUTION, 2 CLEAR

---

## Launch Call

**DECISION: HOLD**

Do not proceed with fine-tuning until:

1. **[REQUIRED]** Fix normalization placement in `model_arch.py:206` — move LayerNorm before attention pooling
2. **[RECOMMENDED]** Reduce domain switching frequency in `training_config.yaml:51`
3. **[RECOMMENDED]** Add early stopping in `training_loop.py`

---

## Tripwire Configuration

```yaml
tripwire:
  name: instruction_drift_monitor
  model: open-llama-finetune-candidate
  
  primary_metric:
    name: instruction_adherence_score
    source: held_out_single_domain_eval
    
  thresholds:
    warning: 0.78
    critical: 0.72
    
  checkpoints:
    - step: 500
      expected_min: 0.85
    - step: 1000
      expected_min: 0.82
    - step: 2000  # <-- known break point
      expected_min: 0.75
    - step: 3000
      expected_min: 0.73
      
  actions:
    on_warning:
      - log_detailed_metrics
      - save_checkpoint
      - alert_team
    on_critical:
      - halt_training
      - rollback_to_last_good
      - reduce_domain_mix_ratio(0.5)
      
  recheck_interval: 500
  
  post_intervention:
    observation_window: 1000
    success_criteria: metric > 0.75 sustained
```

---

*Charter generated via BLOCK framework pre-flight bench*