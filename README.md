# Where Will This Fine-Tune Break? Pre-Flight Check

## The Specimen

A candidate open model checkpoint (`open-llama-finetune-candidate`) staged for fine-tuning on mixed-domain instruction data. The training run is planned for 5k steps with heterogeneous task batches.

## The Verdict

**HOLD** — Instruction drift risk identified at ~2k steps on mixed domain data. The model's task-boundary representations destabilize when domain-switching frequency exceeds the effective context window of learned instruction patterns.

## The Tripwire

```yaml
tripwire:
  metric: instruction_adherence_score
  threshold: < 0.72 on held-out single-domain eval after step 2000
  action: halt training, checkpoint rollback, reduce domain mixing ratio
  recheck: every 500 steps post-intervention
```

## One-Paste Rebuild Block

```bash
# Clone and initialize pre-flight bench
git clone <this-repo>
cd pre-flight-finetune-check

# Run BLOCK clause sweep on your model config
cat your_training_config.yaml | python scripts/clause_check.py --all

# Or run individual clause checks
python scripts/clause_check.py --clause boundaries < model_arch.py
python scripts/clause_check.py --clause leakage < data_pipeline.py
python scripts/clause_check.py --clause overfit < training_loop.py
python scripts/clause_check.py --clause collapse < eval_harness.py
python scripts/clause_check.py --clause kill-switch < checkpoint_manager.py

# Generate pre-flight report
python scripts/generate_report.py --model open-llama-finetune-candidate
```

---

**Build**: baw.v3 workshop  
**Status**: ai_drafted  
**Method**: See METHOD.md for BLOCK framework

<!-- educationpals-build-verified -->