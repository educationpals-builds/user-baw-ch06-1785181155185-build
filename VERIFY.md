# Stranger Verification Protocol

## Purpose

This document enables any third party to verify that the pre-flight bench correctly identifies the seeded failure mode.

## The Seeded Specimen

The following code contains a known defect — post-pooling normalization that causes representation collapse under mixed-domain training:

```python
# model_arch.py — Seeded specimen for verification
# Lines 203-215

class InstructionHead(nn.Module):
    def __init__(self, hidden_size, output_size):
        super().__init__()
        self.attention_pool = AttentionPooling(hidden_size)
        self.layer_norm = nn.LayerNorm(hidden_size)
        self.output_proj = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        # DEFECT: LayerNorm AFTER attention pooling
        # This washes out domain-distinctive features
        pooled = self.attention_pool(x)      # Line 211
        normed = self.layer_norm(pooled)     # Line 212 <-- THE BUG
        return self.output_proj(normed)      # Line 213
```

## Expected Finding

When this specimen is audited, the tool must surface:

| Field | Expected Value |
|-------|----------------|
| Clause | C (Collapse) |
| Finding | BLOCK |
| Cited Line | `model_arch.py:212` or equivalent reference to the `layer_norm(pooled)` line |
| Root Cause | Post-pooling normalization erases domain-distinctive features |
| Connection to Break Point | This causes instruction drift after ~2k steps on mixed domain data |

## Verification Steps

### Step 1: Copy the Specimen

Copy the seeded specimen code block above.

### Step 2: Invoke the Auditor

Paste into any chat model with the Collapse clause prompt from `prompts/clause-walk-pack.md`:

```
[Paste Prompt 4: COLLAPSE (C) from clause-walk-pack.md]

Code/config to audit:

[Paste the seeded specimen]
```

### Step 3: Confirm the Finding

The response must include:

- [ ] `finding: BLOCK` (not CAUTION, not CLEAR)
- [ ] Reference to line 212 or the `normed = self.layer_norm(pooled)` statement
- [ ] Explanation that normalization after pooling causes feature washout
- [ ] Recommended fix: move LayerNorm before pooling

### Step 4: Verify the Fix

Paste the corrected version:

```python
class InstructionHead(nn.Module):
    def __init__(self, hidden_size, output_size):
        super().__init__()
        self.layer_norm = nn.LayerNorm(hidden_size)  # Moved up
        self.attention_pool = AttentionPooling(hidden_size)
        self.output_proj = nn.Linear(hidden_size, output_size)
    
    def forward(self, x):
        # FIXED: LayerNorm BEFORE attention pooling
        normed = self.layer_norm(x)          # Normalize first
        pooled = self.attention_pool(normed) # Then pool
        return self.output_proj(pooled)
```

The auditor should now return:
- [ ] `finding: CLEAR`
- [ ] Confirmation that pre-pooling normalization preserves distinctive features

## Verification Record

After completing verification, record:

```yaml
verification:
  date: [DATE]
  verifier: [NAME/HANDLE]
  chat_model_used: [MODEL NAME - no vendor names]
  specimen_finding_correct: [yes/no]
  line_citation_present: [yes/no]
  fix_verification_correct: [yes/no]
  notes: |
    [Any observations]
```

## Why This Matters

Stranger verification ensures:
1. The bench isn't overfitted to one model's quirks
2. The seeded defect is genuinely detectable
3. The prompts are portable across chat interfaces
4. The framework produces reproducible results

---

*Verification protocol for BLOCK framework pre-flight bench*