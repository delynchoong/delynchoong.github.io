---
title: "(Part 1) Azure OpenAI GPT-5: AI Vision Manufacturing Defect Agent"
categories:
  - blog
tags:
  - Azure OpenAI
  - Github Copilot
  - App
  - Development
  - AI Vision
---

## AI-Powered Vision QA in Manufacturing: From Prototype to Scalable Pattern

Manufacturing industry are complex organizations with many different moving factory lines. There is a huge potential to unlock with AI-powered vision systems that can streamline quality assurance processes and improve defect detection. Manual human inspection is costly, with mistakes costing manufacturers millions of dollars each year. Here, we explore the transformative impact of AI vision on manufacturing defect detection. This series aims to expand into full Agentic AI Factory line implementation.

## Executive Summary

Manufacturers are under pressure to increase yield, reduce scrap, and accelerate root-cause analysis without ballooning engineering overhead. AI vision systems—once bespoke, brittle, and expensive—are now attainable as lightweight, composable services. This repository demonstrates a minimal, production-inclined pattern: multimodal large language model (LLM) reasoning (Azure OpenAI GPT-4o family) + a thin Streamlit interface + deterministic routing of images into pass / fail buckets. It delivers rapid feedback loops for operators while creating a data exhaust for continuous improvement.

> Outcome: Faster defect triage, lower manual inspection effort, structured reasoning artifacts to feed analytics and retraining.

![alt text](/assets/images/2025-08-14/1.png)

## 1. Manufacturing Value Proposition of AI Vision

Strategic gains:

- Throughput: Parallelize automated pre-screening; escalate only contentious cases.
- Quality Intelligence: Mine "reason" strings to uncover recurring defect modes earlier.
- Traceability: Attach decision metadata to MES / digital thread for full genealogy.
- Cost Optimization: Defer capex-heavy classical systems until scale warrants hybridization.

## 2. Repository Overview & Flow

Interested to test it out? Fork the [Github Repo here](https://github.com/delynchoong/AI-Vision-Manufacturing-Agent)


| File/Folder         | Description                                                        |
|---------------------|--------------------------------------------------------------------|
| `app.py`            | Streamlit UI and orchestration                                     |
| `src/file_ops.py`   | Filesystem utilities (create folders, save, move, list, counts)    |
| `src/vision.py`     | Vision analysis using Azure OpenAI (or mock mode)                  |
| `infra/main.bicep`  | (Optional) Azure resource provisioning                             |
| `proccessing/`      | Staging area (ephemeral) for images awaiting inference             |
| `error/`            | Classified defective images                                        |
| `passed/`           | Classified good images                                             |


### Runtime Flow

1. User uploads images (batch) → saved to `proccessing/`.
2. For each image: `vision.analyze_image()`
   - Loads & normalizes
   - (If real mode) Calls Azure OpenAI Chat Completions with embedded base64 image + structured prompt
   - Receives **JSON**: `{defect, confidence, reason}` (enforced via `response_format`)
3. Decision router applies confidence threshold; moves file to `error/` or `passed/`.
4. UI displays annotated image (green or red border) + per-image metrics.

### Key Design Choices

- Filesystem First: Easiest to reason about; later swappable to Azure Blob Storage.
- Mock Mode: Enables demos & CI without secrets (random but biased by filename tokens like "crack").
- Structured Output: Reduces brittle regex parsing; fosters machine-consumable defect logs.
- Minimal State: Pure functions + side-effectful file moves = straightforward scaling path.

## 3. Deep Dive: Core Files

 `app.py`

UI sections: Upload, Inspect, Galleries.
- After visualizing inference result, immediately moves images (idempotence relies on unique UUID filenames).
- Sidebar surfaces environment sanity (deployment name, API key presence) for rapid debugging.

`src/file_ops.py`

- `ensure_folders` guarantees required directories exist (creates on cold start).
- `save_uploaded_file` writes image with a UUID prefix, preventing collisions and aiding traceability.
- `folder_counts` & `list_images` supply quick operational metrics.

 `src/vision.py`

- Loads environment & config (threshold, deployment, API version).
- `MOCK_MODE` toggled automatically if credentials absent (fail-open design for demos).
- Composes a prompt instructing the model to emit a restricted JSON schema; uses `response_format={'type':'json_object'}` for higher reliability.
- Parsing path: safe JSON load → fallbacks on exceptions → applies decision gating on confidence.
- Visual annotation: adds colored border for immediate operator cognition.

`infra/main.bicep`

- (Not expanded here) scaffolds Azure OpenAI, Storage, and an App Service baseline.
- Intended path: integrate Managed Identity & assign least-privileged roles (Blob Data Contributor, etc.).
  

## 4. Getting Started (Quick)

```bash
python -m venv .venv
source .venv/bin/activate  # (Windows: .\.venv\Scripts\Activate.ps1)
pip install -r requirements.txt
cp .env.example .env  # add your Azure OpenAI values
streamlit run app.py
```

Upload images, observe triage, inspect `error/` vs `passed/`.

## 5. Summary

This project is a springboard: a concise blueprint for defect triage that can evolve into a resilient, observable, policy-aligned inspection service. By embracing multimodal reasoning, structured outputs, and incremental architectural hardening, manufacturers can unlock continuous quality intelligence—turning raw images into strategic process improvements.

 Fork the [Github Repo here](https://github.com/delynchoong/AI-Vision-Manufacturing-Agent)
