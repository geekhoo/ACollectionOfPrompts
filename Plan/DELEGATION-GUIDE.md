# GitHub Copilot Agent Delegation Guide

## Overview

All three prompt engineering and orchestration agents have been updated to use GitHub Copilot's official delegation features instead of custom `@custom-agent` syntax.

## Key Changes

### Before (Custom Syntax)
```markdown
@custom-agent backend-specialist

Task: Build REST API
[prompt details]
```

### After (Official GitHub Copilot Syntax)
```bash
# Method 1: Direct delegation to coding agent
/delegate Build REST API with the following specifications:
[complete prompt with all details]

# Method 2: Invoke specific custom agent
/agent backend-specialist
Task: Build REST API
[prompt details]

# Method 3: Tool-based (for programmatic workflows)
Use the custom-agent tool with task description
```

## Delegation Methods

### 1. `/delegate` Command (Recommended for Most Tasks)

**When to use**: Well-defined tasks that need autonomous implementation

**Syntax**:
```
/delegate [Complete task description]

[Full context and requirements]

Success Criteria:
- [Criterion 1]
- [Criterion 2]
```

**What happens**:
- Creates a new branch
- Opens a draft pull request
- Agent works autonomously
- You can monitor via session logs
- You're added as a reviewer

**Example**:
```
/delegate Implement user authentication system

Requirements:
- JWT-based authentication
- Email/password login
- Password reset functionality
- Rate limiting on login attempts

Stack: Node.js + Express + PostgreSQL
Test coverage: >80%
Timeline: 2-3 days
```

### 2. `/agent` Command (For Specialized Agents)

**When to use**: Need specific custom agent's expertise

**Syntax**:
```
/agent [agent-name]

[Task description tailored to that agent]
```

**Example**:
```
/agent database-specialist

Design and implement database schema for e-commerce platform

Tables needed:
- users
- products
- orders
- order_items
- payments

Requirements:
- Proper foreign key relationships
- Indexing strategy for performance
- Migration scripts
```

### 3. Tool-Based Delegation (For Multi-Agent Workflows)

**When to use**: Complex workflows requiring agent coordination

**In agent markdown file, use the `custom-agent` tool**:
```yaml
tools: ["read", "search", "custom-agent"]
```

**In prompt**:
```markdown
I'll delegate this implementation to the specialized agent.
[Task description that will be passed to the tool]
```

## Monitoring & Steering

### Track Progress
- Navigate to GitHub Agents tab
- View active sessions and logs
- See real-time agent activity

### Steer Mid-Execution
Comment on the PR to guide the agent:
```
@copilot Please add error handling for network timeouts
```

### Request Changes
After agent completes:
```
@copilot The validation logic needs to also check for special characters
```

## Updated Agent Files

### 1. implementation-architect-agent.md
**Changes**:
- Added three delegation methods with examples
- Replaced `@custom-agent` syntax with `/delegate` and `/agent`
- Added handoff template for GitHub Copilot
- Included delegation best practices

**New sections**:
- Handoff Using GitHub Copilot Delegation
- Method 1: Direct Delegation (Recommended)
- Method 2: Agent-Specific Invocation
- Method 3: Using custom-agent Tool
- Delegation Best Practices

### 2. prompt-engineer-agent.md
**Changes**:
- Updated handoff protocol with GitHub Copilot methods
- Modified agent selection matrix to show delegation commands
- Changed quick handoff template to use `/delegate`

**New sections**:
- GitHub Copilot Delegation Methods
- Method 1: Direct Task Delegation (Fastest)
- Method 2: Invoke Specific Custom Agent
- Method 3: Tool-Based Delegation

### 3. spec-to-code-orchestrator.md
**Changes**:
- Complete rewrite of handoff protocols
- Added four delegation patterns
- Updated all response templates
- Added delegation monitoring section
- Updated quick reference card

**New sections**:
- GitHub Copilot Delegation Patterns
- Pattern 1: Standard Task Delegation
- Pattern 2: Agent-Specific Invocation
- Pattern 3: Express Handoff (Quick Tasks)
- Pattern 4: Multi-Agent Collaboration
- Handoff Best Practices
- Delegation Monitoring
- Track Delegated Tasks

## Usage Examples

### Simple Task
```
/delegate Add input validation to the user registration form

Requirements:
- Email format validation
- Password strength requirements (8+ chars, uppercase, number)
- Phone number format
- Display clear error messages

Expected: Updated form component with validation + tests
```

### Complex Multi-Phase Task
```markdown
## Phase 1: Database Setup
/agent database-specialist
[Database schema and setup instructions]

## Phase 2: API Development
/agent backend-specialist
[API implementation using schema from Phase 1]

## Phase 3: Frontend Integration
/agent frontend-specialist
[UI implementation consuming the API from Phase 2]
```

### Quick Fix
```
/delegate Fix the bug where user sessions expire prematurely

Issue: Sessions timing out after 5 minutes instead of 30 minutes
Location: Likely in session middleware configuration
Expected: Configuration fix + test to verify session duration
```

## Best Practices

### 1. Provide Complete Context
The delegated agent doesn't see prior conversation, so include everything:
```
/delegate [task]

Background: [Why this is needed]
Requirements: [What must be done]
Constraints: [Limitations]
Examples: [Concrete examples]
Success Criteria: [How to verify]
```

### 2. Set Clear Success Criteria
Always include checkable completion criteria:
```
Success Criteria:
- [ ] All tests pass
- [ ] Code coverage >80%
- [ ] No TypeScript errors
- [ ] Follows ESLint rules
- [ ] README updated
```

### 3. Specify Technology Stack
Be explicit about versions and frameworks:
```
Stack:
- Node.js 18+
- TypeScript 5.x
- Express 4.x
- PostgreSQL 15
- Jest for testing
```

### 4. Include Error Scenarios
Mention edge cases and error handling:
```
Error Handling:
- Invalid input → 400 with clear message
- Duplicate email → 409 Conflict
- Server error → 500 with logged details
- Rate limit → 429 with retry-after header
```

### 5. Provide Examples
Concrete examples reduce ambiguity:
```javascript
// Example usage:
const user = await userService.create({
    email: 'user@example.com',
    password: 'SecurePass123'
});

// Expected response:
{
    id: '123',
    email: 'user@example.com',
    createdAt: '2025-01-15T10:30:00Z'
}
```

## Migration Notes

If you have existing prompts using `@custom-agent`, update them:

### Before
```markdown
@custom-agent backend-specialist

Build API endpoints for user management
```

### After
```
/agent backend-specialist

Build API endpoints for user management

Endpoints required:
- GET /users
- GET /users/:id
- POST /users
- PUT /users/:id
- DELETE /users/:id

[Additional details...]
```

## Troubleshooting

### Agent Not Starting
- Ensure you have GitHub Copilot Pro/Enterprise
- Check that coding agent is enabled for your account
- Verify repository permissions

### Can't Find Custom Agent
- Check agent file is in `.github/agents/` directory
- Verify YAML frontmatter is correct
- Ensure agent name matches file name (without .md)

### Agent Doesn't Understand Task
- Provide more context and examples
- Break down into smaller, clearer steps
- Use `/delegate` with more detailed prompt

### Want to Change Direction Mid-Task
- Comment on the PR with `@copilot` mention
- Provide steering input without stopping the agent
- Can request complete reset if needed

## Resources

- [GitHub Copilot Agents Documentation](https://docs.github.com/en/copilot/how-tos/use-copilot-agents)
- [Creating Custom Agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/coding-agent/create-custom-agents)
- [Managing Copilot Agents](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/manage-agents)
- [GitHub Copilot CLI](https://github.blog/changelog/2025-10-28-github-copilot-cli-use-custom-agents)

---

**Summary**: All three agents now use GitHub Copilot's official `/delegate` and `/agent` commands for task handoffs, enabling autonomous implementation with progress tracking, steering capabilities, and seamless integration with GitHub's pull request workflow.
