# GitHub Copilot Custom Agents - Complete Guide

## Overview

GitHub Copilot Custom Agents allow you to create specialized AI agents with tailored expertise for specific development tasks. These agents are configured through markdown files with YAML frontmatter and can be deployed at repository, organization, or enterprise levels.

## File Structure & Location

### Repository-Level Agents
- **Location**: `.github/agents/` directory
- **Scope**: Available only within the specific repository
- **Example**: `.github/agents/code-reviewer.md`

### Organization/Enterprise-Level Agents  
- **Location**: Root `agents/` directory in `.github-private` repository
- **Scope**: Available across all repositories in the organization/enterprise
- **Example**: `agents/enterprise-reviewer.md` (not `.github/agents/`)

## YAML Frontmatter Syntax

### Basic Structure
```yaml
---
name: agent-unique-name
description: Brief description of agent purpose and capabilities
tools: ["tool1", "tool2", "tool3"]  # Optional, defaults to all tools
---

[Agent prompt and instructions in markdown]
```

### Advanced Structure (Organization/Enterprise Only)
```yaml
---
name: enterprise-agent
description: Enterprise-grade agent with MCP server integration
tools: ["read", "edit", "search", "custom-mcp/*", "github/*"]
mcp-servers:
  custom-mcp:
    type: 'local'  # or 'stdio'
    command: 'mcp-server-command'
    args: ['--arg1', 'value1']
    tools: ["*"]  # or specific tools
    env:
      API_KEY: ${COPILOT_MCP_API_KEY}
      SECRET: ${{ secrets.COPILOT_MCP_SECRET }}
---

[Agent prompt and instructions]
```

## Tools Configuration

### Tool Availability Options

1. **Enable All Tools** (Default)
   ```yaml
   # Option 1: Omit tools property
   ---
   name: my-agent
   description: Agent with all tools
   ---
   
   # Option 2: Explicit wildcard
   ---
   name: my-agent
   description: Agent with all tools
   tools: ["*"]
   ---
   ```

2. **Enable Specific Tools**
   ```yaml
   ---
   name: focused-agent
   description: Agent with limited tools
   tools: ["read", "search", "edit"]
   ---
   ```

3. **Disable All Tools**
   ```yaml
   ---
   name: planning-agent
   description: Planning-only agent without tools
   tools: []
   ---
   ```

### Tool Aliases Reference

| Primary Alias | Compatible Aliases | Purpose |
|--------------|-------------------|---------|
| `shell` | `Bash`, `powershell` | Execute shell commands |
| `read` | `Read`, `NotebookRead` | Read file contents |
| `edit` | `Edit`, `MultiEdit`, `Write` | Edit files |
| `search` | `Grep`, `Glob` | Search files/content |
| `custom-agent` | `Task` | Invoke other custom agents |
| `web` | `WebSearch`, `WebFetch` | Web operations |
| `todo` | `TodoWrite` | Task list management |

### MCP Server Tools

#### Built-in MCP Servers
```yaml
tools: [
  "github/*",           # All GitHub tools
  "github/get-pr",      # Specific GitHub tool
  "playwright/*",       # All Playwright tools
  "playwright/click"    # Specific Playwright tool
]
```

#### Custom MCP Server Tools
```yaml
tools: [
  "security-scanner/*",              # All tools from custom MCP
  "security-scanner/scan-secrets",   # Specific tool
  "sonarqube/analyze-quality"       # Another MCP tool
]
```

## MCP Server Configuration (Enterprise/Organization Only)

### Basic MCP Configuration
```yaml
mcp-servers:
  my-mcp:
    type: 'local'
    command: 'node'
    args: ['./mcp-server.js']
    tools: ["*"]
```

### MCP with Authentication
```yaml
mcp-servers:
  authenticated-mcp:
    type: 'stdio'
    command: 'mcp-server'
    args: ['--config', 'production']
    tools: ["scan", "analyze", "report"]
    env:
      # Different syntax options for secrets
      API_KEY: COPILOT_MCP_API_KEY                    # Direct reference
      TOKEN: $COPILOT_MCP_TOKEN                       # With $
      SECRET: ${COPILOT_MCP_SECRET}                   # With ${}
      GITHUB_TOKEN: ${{ secrets.COPILOT_MCP_GH }}     # GitHub Actions style
      CONFIG_VAR: ${{ var.COPILOT_MCP_CONFIG }}       # Variable style
```

## Best Practices

### 1. Agent Naming Conventions
```yaml
# Good: Clear, specific, lowercase with hyphens
name: security-review-specialist
name: frontend-performance-analyzer
name: database-migration-helper

# Bad: Vague, inconsistent naming
name: MyAgent
name: agent1
name: helper
```

### 2. Comprehensive Descriptions
```yaml
# Good: Detailed and specific
description: Specialized agent for reviewing React/TypeScript applications focusing on performance, accessibility, and code quality standards

# Bad: Too generic
description: Reviews code
```

### 3. Structured Prompts
```markdown
---
name: well-structured-agent
description: Example of well-structured agent prompt
---

# Agent Role & Expertise
[Define the agent's identity and core competencies]

## Responsibilities
[List specific tasks and areas of focus]

## Methodology
### Phase 1: [Analysis Phase]
[Detailed steps]

### Phase 2: [Execution Phase]
[Detailed steps]

## Output Format
[Define expected output structure]

## Quality Standards
[List quality criteria and thresholds]
```

### 4. Tool Selection Strategy
```yaml
# Minimal tools for focused tasks
name: documentation-writer
tools: ["read", "edit", "search"]

# Full tools for comprehensive work
name: full-stack-developer
tools: ["*"]

# Mixed built-in and MCP tools
name: security-auditor
tools: ["read", "search", "security-scanner/*", "github/get-file"]
```

### 5. Sub-Agent Delegation Pattern
```markdown
## Delegation Strategy
When encountering specialized areas, delegate to expert agents:

### Database Review
@custom-agent database-specialist
Focus: Query optimization, indexing, migrations

### Security Audit  
@custom-agent security-auditor
Focus: Vulnerability scanning, compliance checks

### Performance Analysis
@custom-agent performance-expert
Focus: Bottlenecks, optimization opportunities
```

## Tips & Tricks

### 1. Version Control for Agents
```yaml
# Use Git branches/tags for versioning
# agents/reviewer-v1.md (stable)
# agents/reviewer-v2-beta.md (testing)
```

### 2. Environment-Specific Agents
```yaml
# Development environment agent
name: dev-helper
description: Relaxed standards for development

# Production review agent
name: prod-guardian
description: Strict validation for production code
```

### 3. Template Variables (Future Feature)
```yaml
# Potential future syntax for reusable templates
name: team-${TEAM_NAME}-reviewer
description: Reviewer for ${TEAM_NAME} team following ${STANDARDS}
```

### 4. Testing Custom Agents
1. Create agent in feature branch
2. Test with small, controlled PRs
3. Iterate based on results
4. Merge to main when stable

### 5. Monitoring Agent Performance
Track metrics:
- Response time
- Accuracy of findings
- False positive rate
- User satisfaction scores

## Complex SaaS Application Review Example

### Multi-Module Architecture Review
```yaml
---
name: saas-architecture-reviewer
description: Reviews complex SaaS with microservices, APIs, and integrations
tools: ["read", "search", "shell", "github/*", "custom-agent"]
---

# SaaS Architecture Specialist

## Expertise Areas
- Microservices patterns
- API Gateway design
- Event-driven architecture
- Database sharding
- Container orchestration

## Review Process
### Service Discovery
1. Map all services and dependencies
2. Identify API contracts
3. Analyze data flow

### Per-Service Review
For each microservice:
- Validate boundaries
- Check API versioning
- Review error handling
- Assess scalability

### Integration Review
- Message queue patterns
- Service mesh configuration
- Circuit breaker implementation
```

### Security-Focused Agent
```yaml
---
name: security-compliance-auditor
description: Enterprise security and compliance validation
tools: ["read", "search", "security-scanner/*", "github/check-vulnerabilities"]
mcp-servers:
  security-scanner:
    type: 'local'
    command: 'security-cli'
    args: ['--mode', 'deep-scan']
    tools: ["scan-vulnerabilities", "check-secrets", "audit-dependencies"]
    env:
      SCAN_API_KEY: ${COPILOT_MCP_SECURITY_KEY}
---

# Security & Compliance Auditor

## Compliance Frameworks
- SOC 2 Type II
- HIPAA
- GDPR
- PCI-DSS

## Security Checklist
- [ ] OWASP Top 10 coverage
- [ ] Dependency vulnerability scan
- [ ] Secret detection
- [ ] Input validation
- [ ] Authentication/Authorization
```

## Troubleshooting

### Common Issues

1. **Agent Not Appearing**
   - Check file location (`.github/agents/` for repo-level)
   - Verify YAML syntax is valid
   - Ensure agent name is unique
   - Refresh the agents page

2. **Tools Not Working**
   - Verify tool names/aliases are correct
   - Check MCP server configuration
   - Ensure environment variables are set
   - Review tool permissions

3. **MCP Server Errors**
   - Validate JSON/YAML syntax
   - Check command availability
   - Verify secrets in Copilot environment
   - Test MCP server independently

4. **Agent Not Performing as Expected**
   - Review prompt clarity
   - Check tool availability
   - Verify agent has necessary context
   - Test with simpler tasks first

## Advanced Patterns

### Hierarchical Review System
```yaml
# Top-level orchestrator
name: review-orchestrator
tools: ["custom-agent"]

# Delegates to:
# - @custom-agent frontend-reviewer
# - @custom-agent backend-reviewer  
# - @custom-agent security-reviewer
# - @custom-agent performance-reviewer
```

### Context-Aware Agents
```yaml
name: context-aware-reviewer
description: Adapts review based on PR context

# In prompt:
## Context Detection
Analyze PR to determine:
- Feature addition → Focus on functionality
- Bug fix → Verify fix and regression tests
- Refactoring → Check behavior preservation
- Performance → Run benchmarks
```

### Progressive Review Strategy
```yaml
name: progressive-reviewer
description: Multi-pass review with increasing depth

# Pass 1: Quick scan (5 min)
- Critical security issues
- Obvious bugs
- Build failures

# Pass 2: Standard review (30 min)
- Code quality
- Test coverage
- Documentation

# Pass 3: Deep dive (2 hours)
- Architecture implications
- Performance analysis
- Future maintainability
```

## Future Considerations

### Upcoming Features (Speculative)
- Agent composition/inheritance
- Dynamic tool loading
- Cross-repository agent sharing
- Agent performance analytics
- Template variables support
- Agent marketplace

### Integration Possibilities
- IDE integration improvements
- CI/CD pipeline integration
- Third-party tool connections
- Custom metrics collection
- Automated agent optimization

## Resources

- [Official GitHub Docs](https://docs.github.com/en/copilot/reference/custom-agents-configuration)
- [Custom Agents Tutorial](https://docs.github.com/en/copilot/tutorials/customization-library/custom-agents)
- [Awesome Copilot Repository](https://github.com/github/awesome-copilot)
- [MCP Documentation](https://modelcontextprotocol.io/)
- [Community Examples](https://github.com/github/awesome-copilot/tree/main/agents)

## Conclusion

GitHub Copilot Custom Agents provide powerful capabilities for tailoring AI assistance to specific needs. By understanding the configuration syntax, tool ecosystem, and best practices, you can create sophisticated agents that significantly enhance your development workflow, especially for complex tasks like reviewing multi-module SaaS applications.

Key takeaways:
- Start simple and iterate
- Leverage tool combinations effectively
- Use MCP servers for advanced capabilities
- Design clear, structured prompts
- Monitor and refine agent performance
- Consider hierarchical agent systems for complex tasks
