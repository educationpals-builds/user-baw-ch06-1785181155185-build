{
  "schema_version": "1.0",
  "build_id": "baw-v3-preflight-finetune-check",
  "build_name": "Where will this fine-tune break? pre-flight check",
  "created": "2024",
  
  "field_attribution": {
    "learner_provided": [
      "model_name",
      "break_point",
      "clause_check"
    ],
    "ai_drafted": [
      "README.md",
      "charter.md",
      "blueprints/pre-flight-bench.md",
      "prompts/clause-walk-pack.md",
      "METHOD.md",
      "VERIFY.md",
      ".ep/provenance.json"
    ]
  },
  
  "learner_field_values": {
    "model_name": "open-llama-finetune-candidate",
    "break_point": "instruction drift after 2k steps on mixed domain data",
    "clause_check": "BLOCK: boundaries, leakage, overfit, collapse, kill-switch"
  },
  
  "disclosure": {
    "ai_assisted": true,
    "human_reviewed": false,
    "status": "ai_drafted"
  },
  
  "regulatory_marking": {
    "framework": "EU AI Act",
    "article": "Article 50",
    "requirement": "AI-generated content disclosure",
    "compliance_note": "This repository was drafted by an AI system. All files in ai_drafted array were generated based on learner-provided specifications. Human review recommended before production use."
  },
  
  "usage_rights": {
    "license": "Educational use",
    "attribution_required": true,
    "commercial_use": "Requires human review and sign-off"
  },
  
  "verification": {
    "method": "Stranger verification via VERIFY.md",
    "seeded_specimen": "Post-pooling LayerNorm in InstructionHead.forward()",
    "expected_finding": "BLOCK on Collapse clause with line 212 citation"
  }
}