# Comprehensive implementation review prompt

Title: Comprehensive implementation review against spec with tool-assisted validation

## Goal
- Ask VS Code Copilot Chat to perform an end-to-end review of the codebase against the provided Spec, Plan, and Execution Summary.
- Instruct Copilot to use available VS Code tools to validate the implementation (search, open files, run tasks, terminal commands, tests, linters).
- Produce a single comprehensive Markdown review report (structure defined below).

## Inputs (placeholders)
- Spec: (attach or paste SPEC here; include version/date)
- Plan: (attach or paste PLAN here)
- Execution summary: (attach or paste SUMMARY here)
- Codebase: workspace opened in VS Code (ensure key files are open for context)

## Scope
- Trace each requirement in the Spec to code and tests.
- Validate test coverage and test results.
- Run build, linters, and type checks where applicable.
- Inspect CI/task configs, docs, and developer experience (build/run steps).
- Surface security, performance, reliability, and maintainability concerns.

## Instructions to the assistant (must include)
1. Read and synthesize the Spec, Plan, and Execution Summary first. List every requirement (assign or use IDs). If a requirement is ambiguous, call it out with suggested clarifying questions.
2. Search the workspace to locate the implementation for each requirement. For each mapping, provide exact file references and line ranges.
3. Use VS Code context and available tools to validate implementation:
   - Run the project's build and test tasks (use existing tasks/commands). Capture outputs and test results.
   - Run linters/formatters/type-checkers if available. Capture results.
   - If tests are missing for a requirement, propose minimal unit/integration tests and include code snippets for those tests.
4. If test/build commands fail, include command output, diagnose root causes, and suggest concrete fixes. Rerun steps if appropriate.
5. Do not exfiltrate secrets or externalize private code. Keep analysis local to the workspace and the chat session.
6. Cite evidence for every claim (file paths, code excerpts, command outputs). Keep excerpts concise and link to locations.

## Required use of tools
- Use workspace search/symbol navigation to find implementations.
- Use the terminal to run build/test/lint tasks (e.g., `dotnet build/test`, `npm test`, `pytest`, etc.).
- Use available VS Code tasks to run CI-like steps where present.
- If a required tool or config is not present, state the minimal setup to enable validation.

## Output: Comprehensive Markdown report (required structure)
1. Title and metadata
   - Project name, commit/ref (if available), date, reviewer: GitHub Copilot Chat (VS Code)
   - Inputs reviewed (files/paths/versions)
2. TL;DR
   - Overall status: Green / Yellow / Red
   - 3–5 top-level findings
3. Scope & Inputs
   - Exact list of artifacts reviewed and their paths
4. Requirements traceability matrix
   - For each requirement: ID, description, status (Implemented / Partial / Missing), code refs (file:lines), tests (file:lines), evidence (test outputs, logs)
5. Functional validation
   - High-level design mapping to requirements, notable design decisions, edge cases
6. Tests & Coverage
   - Commands used, results summary, coverage % (if available), missing/insufficient tests
7. Static analysis & quality
   - Linter/type/format results and notable issues
8. Security & compliance
   - Input validation, auth, secret handling, vulnerable dependencies, license concerns
9. Performance & reliability
   - Potential bottlenecks and reliability risks; suggested measurements or benchmarks
10. Documentation & developer experience
   - README accuracy, setup steps, CI reproducibility, dev friction
11. Bugs, Risks, Gaps
   - Repro steps for defects (if any), severity, likely root causes
12. Recommendations & next actions
   - Prioritized (P0 / P1 / P2), with concrete implementation or test examples
13. Appendix
   - Commands executed and concise outputs (fenced blocks), environment assumptions, file index

## Constraints & expectations
- Be explicit and evidence-driven; include file:line citations for claims.
- Provide minimal runnable command examples to reproduce checks.
- When proposing code changes, provide small, focused snippets and suggested file paths.
- Prioritize clarifying ambiguous requirements before assuming behavior.

## Acceptance criteria (for this review)
- Every requirement from the Spec is mapped to code and tests or clearly labeled Missing, with evidence.
- Build, test, and lint commands were attempted; results included.
- A prioritized, actionable remediation list is provided.
- The report is self-contained and reproducible.

---

## How to run this prompt in VS Code Copilot Chat
1. Open the Spec, Plan, and Execution Summary files in VS Code. Open key implementation files you think are important.
2. Start a Copilot Chat session and paste the entire "Instructions to the assistant" and the "Output" sections above. Replace the Input placeholders with the actual content or attach them.
3. Allow Copilot to use workspace context. Ask it to begin the review. If necessary, follow up with clarification requests.


*Sources and best-practice references used when crafting this prompt:*
- "Prompt engineering for Copilot Chat" — VS Code docs
- "Prompt engineering for GitHub Copilot Chat" — GitHub docs
- Microsoft/.NET blog posts on prompt files and instructions
- Microsoft Engineering Playbook: Markdown code review recommendations


*Notes:* Paste or attach the Spec/Plan/Summary into the chat before running the prompt to ensure full context is available to Copilot Chat.