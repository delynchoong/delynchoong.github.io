---
title: "AI Vision Manufacturing Defect Agent"
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
| app.py            | Streamlit UI and orchestration                                     |
| `src/file_ops.py`   | Filesystem utilities (create folders, save, move, list, counts)    |
| `src/vision.py`     | Vision analysis using Azure OpenAI (or mock mode)                  |
| `infra/main.bicep`  | (Optional) Azure resource provisioning                             |
| `proccessing/`      | Staging area (ephemeral) for images awaiting inference             |
| `error/`            | Classified defective images                                        |
| `passed/`           | Classified good images                                             |
