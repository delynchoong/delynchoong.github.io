---
title: "Github Copilot GPT-5: Supercharged Application Development"
categories:
  - blog
tags:
  - Github
  - Github Copilot
  - App
  - Development
  - AI
---

It's been one week since the grand release of [OpenAI's GPT-5 model](https://openai.com/gpt-5/). What does this mean for their partner Microsoft? Similar to the past few releases, GPT-5 is available same day in [Microsoft products like M365 Copilot, AI Foundry and Github Copilot](https://news.microsoft.com/source/features/ai/openai-gpt-5/), enhancing productivity and efficiency.

I've been testing the capabilities out over the past week and here are some interesting projects that I've coded in my trusty Github Copilot GPT-5 Agent mode.

This post gives a concise, practical take on the GPT-5 launch: what’s improved, how it stacks up conceptually against Anthropic’s Claude family, and three hands-on use cases you can try right now!

Want to try GPT-5 quickly? Use [Copilot Chat](https://m365.cloud.microsoft/chat/?fromcode=m365copilot) to switch to the newest model
![alt text](/assets/images/2025-08-12/13.png)

## What’s new in GPT-5 (at a glance)

Based on publicly shared themes for next‑gen models, GPT‑5 emphasizes:

- Stronger planning and tool-use orchestration for multi-step tasks
- More robust code generation and refactoring support
- Long-context understanding for working across larger specs and documents
- Multimodal inputs/outputs with tighter grounding and analysis
- With Microsoft, the model is available via API and can be orchestrated by the model router, which evaluates each prompt and decides the optimal model based on the complexity, performance needs, and cost efficiency of each task

## GPT-5 vs Anthropic (Claude family): a pragmatic view

Both model families have become excellent at:

- Reasoning across multi-step instructions and using tools/functions
- Generating and fixing code with tests and incremental improvements
- Handling long-context inputs (specs, transcripts, multi-file summaries)
- Multimodal understanding (text + images; structured outputs)

Where you might notice differences in practice:

- Planning and tool orchestration: You may find GPT‑5 slightly more assertive at breaking down tasks and using tools in parallel; Claude tends to be conservative and precise.
- Coding ergonomics: Both are strong. If you rely on rapid iteration and agentic refactoring, GPT‑5’s planner behavior can feel productive. Claude often excels at careful edits and polished prose in code comments and docs.
- Safety/controls: Both ship strong guardrails. For regulated scenarios, evaluate each model’s policy controls and auditing hooks in your environment.

The takeaway: choose per task. It’s common to mix models—one for planning, another for polish—behind a consistent tool interface.

I've been using [Claude Sonnet 3.7](https://www.anthropic.com/claude/sonnet) to power my GH Copilot previously, and now, I've switch to Gpt-5(preview) instead. While I won't go into a one-on-one comparison in this blog, nor have I fully benchmarked the difference, the new GPT-5 model impressed me with it's capability to handle complex project code and iterative debugging 

![alt text](/assets/images/2025-08-12/12.png)

---

## Use Case 0.1 Everyone's favourite test

Firstly, I did a basic litmus test for all LLM models - the infamous B test in [Azure AI Foundry](https://azure.microsoft.com/en-us/blog/gpt-5-in-azure-ai-foundry-the-future-of-ai-apps-and-agents-starts-here/) for brevity's sake.
![alt text](/assets/images/2025-08-12/11.png)

No problems here. Now on to more challenging tasks.

## Use Case 1. Start small: run a Tetris game

Goal: validate fast code generation + iterative fixes on a small, visual project.

Suggested prompt

> Generate a minimal Tetris in one HTML file using JavaScript and Canvas. Keep it under ~200 lines, with arrow keys, rotation, pause function, and game over state.

![alt text](/assets/images/2025-08-12/tetris.gif)

- GPT-5 coded a single `index.html` you can drop into the repo, or just use the existing `tetris.html` in this site (see: {{ site.baseurl }}/assets/data/tetris/html).
- Iterate: request smaller functions, clearer collision logic, and tweak gravity/levels.

Why it’s useful

- Quick visual feedback to assess model quality, or for benchmarking
- Great for testing “explain, refactor, improve” loops

### Use Case 2. 3D city building simulator: code-from-prompt + planner capability

Goal: test the model’s ability to decompose a specification, pick libraries, and scaffold a working 3D scene.

Suggested prompt

> Code me a city building simulator that is procedurally generated, I want to view in the browser from a HTML file. The buildings should be realistic and I can move the camera to explore the city.

Let's see how the agent is able to build from a simple 2 line prompt.

![alt text](/assets/images/2025-08-12/city1.gif)

The first iteration is not too bad. Also note the plan the model generates before executing coding tasks:
![alt text](/assets/images/2025-08-12/10.png)

However, there are some enhancements I'd like to add to my project, also if you noticed, the cars are positioned incorrectly. Let's fix that and also add more parks, weather options and light source simulation.

### Generating weather, building density and parks in the building simulator

The cars are currently moving sideways, as seen here.

![alt text](/assets/images/2025-08-12/8.png)

Let's take a look at the code to figure out how to rotate the car by 90 degrees.

![alt text](/assets/images/2025-08-12/7.png)
Added formula to fix the rotation along x-axis

``` tmp.rotation.y = car.lane.dir > 0 ? -Math.PI/2 : Math.PI/2; ```
Cars now look good

![alt text](/assets/images/2025-08-12/9.png)

What to look for

- Does the model propose a plan before coding? Can it explain trade-offs? how often does it cKheck in?
- How well does it refactor when you request “extract building generator into a backend module”?
- Does it keep changes minimal and stable across iterations?

After a few iterations, include adding weather features. This is the final result:
![alt text](/assets/images/2025-08-12/city2.gif)

I kept my requirements for the simulation to be lightweight, hence GPT-5 only leveraged simple threeJS libaries for the asset creations. Check out the [City Simulator Github Repo](https://github.com/delynchoong/3D-City-Building-Simulator) to have some fun yourself.

Why it’s useful
- Potential to combine with other engines for a full fledged 3D modeling product, in real estate and urban planning industry.

---

### Use Case 3. Practical demo: create interactive demos for customers

Goal: combine GPT‑5’s planner with  MCP servers to create GenAI applications for customer demos to drive value proposition.

High-level flow

1. Agent-to-Agent MCP servers (best‑practices, linting, docs search, cloud SDK, etc.)
2. Provide a brief customer scenario (domain, constraints, success criteria)
3. Let the model plan steps, call MCP tools, necessary OpenAI API models and assemble a demo package.

Example prompt

> You are an AI solution engineer. Using available MCP tools, generate a minimal customer demo for a manufacturing AI Envisioning session. Use Python FastAPI service, AI Vision capabilities at lowest cost, and a README with prerequisites and run steps. Create a IaC Bicep file for repeatable deployment. Enforce best-practice checks and stop if any critical check fails; report the failure and remediation.

GPT-5 immediately reasons the steps required, and calls the [Azure MCP server to retrieve best practice recommendations](https://learn.microsoft.com/en-us/azure/developer/azure-mcp-server/tools/azure-best-practices):

![alt text](/assets/images/2025-08-12/6.png)

The demo was capable of quickly detecting defects in images using the vision capability. Since I instructed the model to be cost effective, it utilized a simple GPT-4o model. Further enhancement should implement Azure AI Vision with it's own classication datasets.

![alt text](/assets/images/2025-08-12/14.png)

What to look for

- Can GPT‑5 self-check with best-practice tools before finalizing?
- Does it produce a clean, runnable demo with clear README instructions?
- How quickly can you iterate to a version you’d share with a customer?

A few iterations and paired mode debugging was needed to refine the demo, but overall it was a good enough to demonstrate the key concepts of multimodal AI capability to bring to a customer demo. 

Explore the [real-time manufacturing defect detection repo](https://github.com/delynchoong/real-time-manufacturing-defect-detection)

### Future Enhancements

Looking ahead, I plan to evolve this POC by integrating advanced agentic capabilities. This includes adding dedicated storage solutions to persist data and state across sessions, which will enable more complex workflows and richer user experiences. Additionally, I aim to extend the application into a multi-agent system, where multiple AI agents can collaborate or specialize in different tasks to unlock new possibilities for intelligent automation and decision support within the app.

## Final tips

- Treat prompts as specifications: ask for a plan, modular code, and success criteria.
- Use small “test benches” (like Tetris) to measure iteration quality quickly.
- Mix models when it helps: one for planning, another for polish, behind the same tools.

Enjoy experimenting with the new GPT-5 features!
