Here’s a compact “Art of War → AI prompting” guide you can use when designing agent prompts.

### High‑level frame: treat each prompt as a campaign plan
- **Clarify**: objective (victory), constraints (resources, time, risk), and theater (domain, tools, data) before asking for output.
- **Ask the agent to think in phases**: assess → plan → act → review, instead of “answer in one shot”.
- **Make the agent explicitly choose strategies rather than just dump information.**

1. Achieving victory without fighting → prompts for leverage, not brute force
Goal: get maximal impact with minimal conflict, rework, or token burn.

   - *Prompt design strategies:*
     - Ask for ways to “win without direct confrontation”:
       - “Given [goal], list 3 options that make competitors/objections irrelevant rather than fighting them directly.”
       - “Focus on solutions that reuse existing assets, code, and workflows instead of replacing them.”
     - Embed “preserve the army” thinking:
       - “Propose a solution that minimizes organizational change, legal risk, and engineering disruption.”
       - “For each idea, explain how it avoids unnecessary conflict with existing systems and stakeholders.”
     - Ask for “blue ocean” thinking:
       - “Suggest niche opportunities where there is little or no competition, and explain why they are strategically under‑contested.”

2. Deception & psychological dislocation → prompts for framing, signaling, and scenario play
(Use this ethically: scenario analysis, negotiation prep, messaging, not disinformation.)

    - *Prompt design strategies:*
      - Have the agent explore perception, not just facts:
        - “How will [stakeholder types] likely perceive this move? What signals does it send?”
        - “List ways our current behavior might be misread by competitors, and how to adjust messaging.”
      - Use role‑play / posture simulations:
        - “Act as our toughest competitor’s strategist. Given this announcement, what weaknesses do you think we are revealing?”
        - “Act as a skeptical investor and write 5 hard questions this plan would trigger.”
      - Design “decoy stress tests” for your own plan:
        - “Assume our rivals misinterpret our actions in the worst possible way. What counter‑moves might they attempt, and how do we pre‑empt them?”

3. Comprehensive assessment & foreknowledge → prompts for deep context and pre‑mortems
Goal: ensure the agent does serious “temple calculations” before recommending action.

    - *Prompt design strategies:*   
      - Make the agent explicitly assess the five factors (you can keep them implicit or name them):
        - “Before proposing a plan, briefly analyze:
          - Moral alignment: who needs to believe in this?
          - Timing: why now? what time constraints?
          - Terrain: platforms, markets, tech stack, regulations.
          - Leadership: decision‑makers and their incentives.
          - Law/discipline: policies, constraints, budget, SLAs.”
      - Use pre‑mortems and scenario planning:
        - “Run a pre‑mortem: assume this project failed in 12 months. List the top 10 plausible causes and early warning signs.”
        - “Provide 3 scenarios (optimistic / base / pessimistic) and how our strategy should adapt in each.”
      - Ask for information gaps and intelligence needs:
        - “List the 10 most critical unknowns that could invalidate this plan, and what data/research would reduce that uncertainty.”

4. Speed and rapid decision → prompts for fast, iterative action
Goal: avoid prolonged, token‑heavy “wars of attrition” with the model and with the project.

    - *Prompt design strategies:*
      - Force an MVP‑first mindset:
        - “Give a ‘minimum viable plan’ we can execute in 1 week with existing resources, before any large refactors.”
        - “For each idea, include: (a) fastest version we can try in 1–2 days, (b) 4–6 week expansion if it works.”
      - Ask for prioritized, not exhaustive, outputs:
        - “List only the top 3 actions by impact/effort ratio, with 2–3 sentences each. Do not generate long lists.”
        - “If you’re unsure, propose a small, fast experiment to learn more rather than a large commitment.”
      - Include explicit iteration hooks:
        - “First: produce a concise plan.
          Second: ask me 3–5 clarifying questions that would improve it.
          Wait for answers before refining.”

5. Striking emptiness, avoiding solidity → prompts for leverage points and weak spots
Goal: focus limited effort on “emptiness” (vulnerabilities, gaps, neglected edges) rather than fighting strengths.

    - *Prompt design strategies:*
      - Ask the agent to map strengths vs weaknesses for you and others:
        - “Given this ecosystem, identify:
          - Our strengths we should avoid over‑defending.
          - Our weaknesses we must protect.
          - Rivals’ weak points where a small move creates outsized disruption.”
      - Target customer pain points instead of feature parity:
        - “Ignore matching competitors’ features. Instead, list the 5 most painful unresolved customer problems and how we can over‑serve them.”
      - For engineering work, focus on leverageful changes:
        - “From this codebase or system description, identify small, local changes that would unlock large benefits (performance, maintainability, UX). Rank by leverage.”

6. Intensive use of spies & intelligence → prompts for research, sensing, and feedback loops
Here “spies” = data, logs, user feedback, competitor info, prior docs, and other tools the agent can use.

    - *Prompt design strategies:*
      - Make the agent a director of research, not just a writer:
        - “Given [goal], specify the 10 most useful artifacts you would like to see (documents, logs, metrics, competitor info).”
        - “You have access to: [list tools/data]. Design a research plan using them before you recommend strategy.”
      - Ask for intelligence‑driven plans:
        - “Base your recommendations only on the attached sources. First, summarize what these sources imply about reality; then propose actions.”
        - “For each proposal, state explicitly: which evidence in the context supports this, and what evidence is missing.”
      - Build secrecy/operational security into prompts when relevant:
        - “Flag any parts of this plan that should not be broadly shared inside the organization and explain why (e.g., legal, negotiation, competitive risk).”

Putting it together: a generic “Art of War” agent prompt skeleton
You can adapt this skeleton for different agents:

- “You are a strategic AI agent helping me win in [domain] with minimal conflict and maximal leverage.
  1) Analyze the situation:
     - Objective, constraints, stakeholders, timing, and terrain.
  2) Propose ways to achieve the objective by:
     - Avoiding direct confrontation where possible.
     - Targeting neglected opportunities and pain points.
  3) Use intelligence:
     - Identify key unknowns and the data we should gather.
     - Base your advice on the provided context, and call out assumptions.
  4) Favor speed and iteration:
     - Give a minimal, fast plan first (1–2 weeks), then a longer‑term expansion.
     - Prioritize actions by impact/effort.
  5) Anticipate counter‑moves:
     - Briefly simulate how competitors/critics might respond and how we can pre‑empt or redirect them.

  Respond in a concise, structured format: [Assessment] → [Opportunities] → [Plan] → [Risks & Counters].”

