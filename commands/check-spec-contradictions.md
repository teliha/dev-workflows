# Check Spec Contradictions Command

Analyze specification files for contradictions, inconsistencies, and ambiguities.

Please analyze all specification files in the `specs/` directory and identify:

1. **Direct Contradictions**: Statements that directly conflict with each other
2. **Inconsistencies**: Similar features described differently across specs
3. **Ambiguities**: Unclear or vague requirements that could be interpreted multiple ways
4. **Missing Information**: Critical details that should be specified but aren't
5. **Outdated Information**: Specs that may not match current implementation

For each finding, provide:
- Location (file path and section)
- Description of the contradiction/issue
- Potential impact
- Suggested clarifying questions

Generate a summary report with actionable questions that need to be answered by the team.
