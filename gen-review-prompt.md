# Code Review Agent Prompts for GitHub Copilot

## Overview
This directory contains comprehensive prompts and custom agent definitions for conducting thorough code reviews using GitHub Copilot and VSCode Copilot Chat.

## Generated Files

### 1. VSCode Copilot Review Prompt
**File**: [`code-review-agent-prompt.md`](./Review/code-review-agent-prompt.md)
- Comprehensive prompt for VSCode Copilot Chat
- Structured review methodology with phases
- Detailed markdown report template
- Requirements validation matrices
- Security, performance, and quality checklists

### 2. SaaS Application Review Agent
**File**: [`saas-code-review-agent.md`](./Review/saas-code-review-agent.md)
- GitHub Copilot custom agent definition
- Specialized for complex SaaS applications
- Multi-module and microservices expertise
- Integration with GitHub and Playwright tools
- Delegation to specialized sub-agents

### 3. Enterprise Review Orchestrator
**File**: [`enterprise-review-orchestrator.md`](./Review/enterprise-review-orchestrator.md)
- Enterprise-grade custom agent with MCP servers
- Advanced security scanning integration
- SonarQube quality analysis
- Jira issue tracking integration
- Multi-team coordination capabilities

### 4. Custom Agents Complete Guide
**File**: [`github-copilot-custom-agents-guide.md`](./Review/github-copilot-custom-agents-guide.md)
- Comprehensive documentation on custom agents
- YAML frontmatter syntax reference
- MCP server configuration examples
- Best practices and tips
- Troubleshooting guide

## Key Features

### Review Methodology
- **Phase-based approach**: Context → Discovery → Validation → Analysis → Testing
- **Tool integration**: Leverages all available VSCode and GitHub tools
- **Requirements traceability**: Maps specs to implementation
- **Comprehensive validation**: Static analysis, dynamic testing, security scanning

### Report Structure
- Executive summary with risk assessment
- Requirements validation matrices
- Categorized findings (Critical/High/Medium/Low)
- Code quality metrics and analysis
- Security assessment with OWASP compliance
- Performance review with bottleneck identification
- Actionable recommendations with effort estimates

### Custom Agent Capabilities
- **Tool control**: Selective enabling of specific tools
- **MCP integration**: Extended capabilities via Model Context Protocol
- **Sub-agent delegation**: Specialized agents for specific domains
- **Enterprise features**: Organization-wide deployment and configuration

## Usage

### For VSCode Copilot Chat
1. Open the [`code-review-agent-prompt.md`](./Review/code-review-agent-prompt.md) file
2. Copy the entire content
3. Paste into VSCode Copilot Chat when conducting reviews
4. Provide the codebase, spec, plan, and execution summary

### For GitHub Copilot Custom Agents
1. Choose the appropriate agent file:
   - [`saas-code-review-agent.md`](./Review/saas-code-review-agent.md) for SaaS applications
   - [`enterprise-review-orchestrator.md`](./Review/enterprise-review-orchestrator.md) for enterprise setups
2. Deploy to your repository:
   - Repository-level: `.github/agents/[agent-file].md`
   - Organization-level: `agents/[agent-file].md` in `.github-private` repo
3. Select the agent from GitHub Copilot interface

## DevExtreme jQuery UI Component Generators

### 5. DevExtreme jQuery Generator Agent
**File**: [`devextreme-jquery-generator-agent.md`](./Review/devextreme-jquery-generator-agent.md)
- Expert agent for generating DevExtreme jQuery UI components
- Standards-compliant HTML, CSS, and JavaScript (no frameworks)
- MCP server integration for DevExpress documentation
- Strict CSS scoping with unique component identifiers
- BEM methodology and namespace patterns
- Production-ready component templates

### 6. DX Component Builder
**File**: [`dx-component-builder.md`](./Review/dx-component-builder.md)
- Rapid DevExtreme component builder with intelligent defaults
- Pre-built patterns for DataGrid, Form, Chart components
- Universal component wrapper with scoped styles
- Data source helpers and validation patterns
- Quick-start templates and minimal examples
- MCP documentation integration syntax

## Prompt Engineering & Implementation Agents

### 7. Implementation Architect Agent
**File**: [`implementation-architect-agent.md`](./Plan/implementation-architect-agent.md)
- Analyzes user requirements and researches optimal solutions
- Uses web search and tools to find best practices
- Generates comprehensive implementation prompts using PRECISE framework
- Creates detailed step-by-step coding instructions
- Includes error handling, testing, and edge cases
- Routes to appropriate specialized coding agents

### 8. Prompt Engineer Agent
**File**: [`prompt-engineer-agent.md`](./Plan/prompt-engineer-agent.md)
- Rapid prompt generation with intelligent defaults
- Pattern recognition for common request types
- Quick prompt templates for various scenarios
- Automatic technology stack selection
- Emergency patterns for vague requirements
- Quality scoring for generated prompts

### 9. Spec-to-Code Orchestrator
**File**: [`spec-to-code-orchestrator.md`](./Plan/spec-to-code-orchestrator.md)
- Master orchestrator for requirement-to-code transformation
- 10-second requirement assessment
- Adaptive research depth based on complexity
- Progressive prompt system (Express/Standard/Comprehensive)
- Intelligent agent routing algorithm
- Progress tracking and blocker resolution

### GitHub Copilot Delegation Guide
**File**: [`DELEGATION-GUIDE.md`](./Plan/DELEGATION-GUIDE.md)
- Complete guide to GitHub Copilot's `/delegate` and `/agent` commands
- Three delegation methods with examples
- Monitoring and steering delegated tasks
- Migration from custom syntax to official syntax
- Best practices and troubleshooting
- Real-world usage examples

## Requirements Addressed
✅ Clear and complete prompt for LLM code review
✅ Analysis of spec requirements, plan, and execution  
✅ Tool utilization for testing and validation
✅ Comprehensive markdown report format
✅ Best practices from industry research
✅ Support for complex multi-module applications
✅ Enterprise-grade review capabilities
✅ DevExtreme jQuery UI component generation
✅ MCP server integration for documentation
✅ Standards-compliant HTML/CSS/JS with proper scoping
✅ Requirement-to-implementation prompt generation
✅ Web search integration for best practices research
✅ Intelligent agent routing and delegation
✅ Progressive prompt complexity levels
✅ Automated handoff to specialized coding agents
✅ GitHub Copilot `/delegate` and `/agent` commands
✅ Task monitoring with session logs and PR tracking
✅ Mid-execution steering via `@copilot` mentions
✅ Multi-agent collaboration workflows

```
old notes [**IGNORE**]:
Compare this snippet from Review/github-copilot-custom-agents-template.md:search for vscode copilot chat best practices on prompting LLM

identify how to generate a clear and complete prompt for LLM to review codebase, the LLM will be provided with the codebase, the initial spec requirements, the plan and execution summary

the prompt MUST instruct vscode copilot to clearly analyse those documents and codebase, and to use tools available to test and validate implementation of spec requirements. 

the prompt also MUST request the output of the review to be a comprehensive markdown report (please search web for best practices of such review report)

websearch for references and syntax of custom agent file for github copilot, it must be the latest and find examples of agents and task specific subagents, and understand how custom agents definition markdown files are written, the best practices, tips and tricks. read this https://docs.github.com/en/copilot/reference/custom-agents-configuration read this https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents read this https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents ultrathink about how to create a custom agent for code review of a complex saas application that includes multiple modules, services, and integrations.
```