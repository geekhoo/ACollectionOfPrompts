@workspace Perform a comprehensive code review of this codebase against the provided specification requirements.

- $input: [spec-requirements.md]
- $input: [implementation-plan.md] 
- $input: [execution-summary.md]

# Context Documents
Review the following documents:
- Initial specification requirements: #file:[spec-requirements.md]
- Implementation plan: #file:[implementation-plan.md]  
- Execution summary: #file:[execution-summary.md]
- Codebase: Use #codebase to analyze all implementation files

# Review Objectives
1. Verify that ALL specification requirements have been implemented correctly
2. Validate implementation quality against best practices
3. Test critical functionality using available tools
4. Identify gaps, bugs, security issues, and technical debt
5. Assess code maintainability and documentation quality

# Analysis Instructions
For each specification requirement:
- Map the requirement to specific code implementations in the codebase
- Analyze if the implementation correctly fulfills the requirement
- Use /tests to generate and validate test coverage for the requirement
- Check for edge cases and error handling
- Verify security considerations and input validation

Use available tools to:
- Run existing unit and integration tests
- Execute code to verify functionality
- Check for compilation/syntax errors
- Validate API endpoints and data flows
- Review logging and error handling mechanisms

# Output Format
Generate a comprehensive markdown report with the following structure:

## Executive Summary
- Overall assessment (Pass/Partial Pass/Fail)
- Key findings summary (3-5 bullet points)
- Critical issues requiring immediate attention
- Recommendation for production readiness

## Requirements Validation
For each requirement from the specification:
### Requirement [ID]: [Name]
- **Status**: ✅ Implemented / ⚠️ Partial / ❌ Missing
- **Implementation Location**: File paths and line numbers
- **Validation Results**: Test outcomes and verification
- **Issues Found**: Bugs, gaps, or concerns
- **Recommendations**: Suggested improvements

## Code Quality Analysis
### Functionality
- Requirement coverage analysis
- Edge case handling
- Error scenarios and exception handling

### Code Structure and Design  
- Architectural adherence
- Design pattern usage
- Modularity and separation of concerns
- Code duplication analysis

### Readability and Maintainability
- Naming conventions consistency
- Code organization and formatting
- Comment quality and documentation
- Complexity assessment (functions/classes size)

### Performance and Efficiency
- Performance bottlenecks identified
- Algorithm and data structure appropriateness
- Resource utilization (memory, CPU)
- Database query optimization

### Security
- Security vulnerabilities (injection, XSS, authentication)
- Input validation and sanitization
- Sensitive data handling
- Secure coding practices adherence

### Testing Coverage
- Unit test coverage percentage
- Integration test coverage
- Test quality and edge case coverage
- Missing test scenarios

## Test Validation Results
### Tests Executed
[Table format:]
| Test Name | Type | Status | Issues Found |
|-----------|------|--------|--------------|
| ...       | ...  | ...    | ...          |

### Test Coverage Gaps
- Areas lacking test coverage
- Missing integration tests
- Recommended additional tests

## Issues and Findings
[For each issue, use this format:]
### Issue [ID]: [Title]
- **Severity**: Critical / High / Medium / Low
- **Category**: Bug / Security / Performance / Maintainability / Documentation
- **Location**: File path and line numbers
- **Description**: Detailed explanation
- **Impact**: How this affects the system
- **Recommendation**: How to fix or improve

## Technical Debt
- Code smells identified
- Refactoring opportunities
- Deprecated dependencies or patterns
- Areas requiring improvement

## Documentation Review
- README completeness
- API documentation quality
- Inline code documentation
- Setup and deployment instructions
- Missing or outdated documentation

## Recommendations
### Immediate Actions (Critical Priority)
1. [Action item with specific details]
2. [Action item with specific details]

### Short-term Improvements (High Priority)
1. [Action item with specific details]
2. [Action item with specific details]

### Long-term Enhancements (Medium Priority)
1. [Action item with specific details]
2. [Action item with specific details]

## Conclusion
- Summary of review findings
- Overall code quality assessment
- Production readiness decision
- Next steps and sign-off requirements

Ensure all findings are:
- Specific with file paths and line numbers
- Actionable with clear recommendations
- Prioritized by severity and impact
- Supported by test results or code analysis
