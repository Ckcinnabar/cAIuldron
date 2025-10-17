<!--
SYNC IMPACT REPORT
==================
Version Change: [NEW] → 1.0.0
Modified Principles: N/A (initial version)
Added Sections:
  - Core Principles (5 principles: Notebook-First Development, Cell-Based Modularity,
    Reproducibility & Environment Management, Documentation-Driven Notebooks, Python Best Practices)
  - Development Workflow
  - Quality Standards
  - Governance
Removed Sections: N/A
Templates Status:
  ✅ .specify/templates/plan-template.md - updated with notebook project structure and Python/Jupyter specific fields
  ✅ .specify/templates/spec-template.md - reviewed, compatible (no changes needed)
  ✅ .specify/templates/tasks-template.md - updated with notebook-specific task examples and path conventions
Follow-up TODOs: None
-->

# cAIuldron Constitution

## Core Principles

### I. Notebook-First Development

All source code MUST be written in Jupyter notebook (.ipynb) format. This principle ensures:
- Interactive development and immediate feedback
- Inline visualization and data exploration
- Self-documenting code with markdown explanations
- Reproducible analysis workflows

**Rationale**: Jupyter notebooks provide the optimal environment for AI/ML experimentation,
data analysis, and iterative development by combining code, documentation, and visualization
in a single executable document.

### II. Cell-Based Modularity

Notebooks MUST be organized with clear cell-level separation of concerns:
- Each code cell MUST have a single, well-defined purpose
- Related functionality MUST be grouped in logical cell sequences
- Import cells MUST be placed at the beginning of notebooks
- Configuration and constants MUST be defined in dedicated cells
- Reusable functions MUST be extracted to separate utility notebooks when used across multiple notebooks

**Rationale**: Cell-based modularity enables easier debugging, testing, and maintenance.
Clear cell organization allows developers to understand and modify specific functionality
without analyzing the entire notebook.

### III. Reproducibility & Environment Management

All notebooks MUST be fully reproducible:
- Environment dependencies MUST be declared in requirements.txt or environment.yml
- Python version MUST be explicitly specified
- Random seeds MUST be set for all stochastic operations
- Data dependencies and sources MUST be documented in the notebook
- Execution order MUST be clear (cells should run top-to-bottom without errors)
- External data files MUST include version information or checksums where applicable

**Rationale**: Reproducibility is fundamental to scientific computing and AI development.
Other developers and stakeholders must be able to re-run notebooks and obtain identical
results on different machines and at different times.

### IV. Documentation-Driven Notebooks

Every notebook MUST include:
- Markdown title cell with notebook purpose and overview
- Section headers using markdown cells to organize major sections
- Inline markdown cells explaining complex logic, algorithms, or decisions
- Code comments for non-obvious implementation details
- Output cells showing expected results (keep meaningful outputs in version control)
- Summary markdown cell at the end with key findings or next steps

**Rationale**: Notebooks serve as both executable code and living documentation.
Rich documentation enables knowledge transfer, facilitates code review, and
provides context for future modifications.

### V. Python Best Practices

Despite the notebook format, code MUST adhere to Python standards:
- Follow PEP 8 style guidelines (use tools like black, flake8)
- Use type hints for function signatures where beneficial
- Implement error handling for external operations (file I/O, API calls, etc.)
- Avoid global state mutations; prefer functional or class-based approaches
- Use descriptive variable names (avoid single letters except for conventional cases like loop indices)
- Keep cell outputs reasonable in size (avoid printing large datasets)

**Rationale**: Professional Python standards ensure code quality, maintainability,
and team collaboration even in an interactive notebook environment.

## Development Workflow

### Notebook Organization

Projects MUST organize notebooks by purpose:
- **Exploratory notebooks**: Prefix with `explore_` for initial data exploration and experimentation
- **Analysis notebooks**: Prefix with `analyze_` for specific analytical tasks
- **Model notebooks**: Prefix with `model_` for model training and evaluation
- **Pipeline notebooks**: Prefix with `pipeline_` for production workflows
- **Utility notebooks**: Prefix with `utils_` for shared functions and helpers

### Version Control

Notebooks in version control MUST:
- Clear output cells before committing UNLESS the output is essential for documentation
- Use nbstripout or similar tools to manage output cell versioning
- Include a .gitignore that excludes checkpoint files (.ipynb_checkpoints/)
- Commit with descriptive messages indicating what analysis or functionality changed

### Testing Approach

For notebook-based code:
- Critical functions SHOULD be extracted to .py modules with unit tests when stability is required
- Notebook testing can use assertion cells to validate intermediate results
- Integration testing SHOULD execute complete notebooks and verify final outputs
- Use papermill or nbconvert for automated notebook execution in CI/CD

## Quality Standards

### Code Quality Gates

Before merging notebook changes:
- All cells MUST execute successfully in top-to-bottom order
- No unused imports or variables in final version
- Code formatting MUST pass linting checks (black, flake8 compatible)
- Large data outputs MUST be cleared or summarized
- Sensitive information (API keys, passwords) MUST NOT be present

### Performance Considerations

Notebooks SHOULD:
- Document expected execution time for long-running cells
- Use progress bars (tqdm) for loops that take >30 seconds
- Implement checkpointing for expensive computations
- Cache intermediate results when appropriate
- Optimize memory usage to prevent kernel crashes

## Governance

### Amendment Process

Constitution amendments require:
1. Proposed changes documented in a markdown cell or separate proposal document
2. Team review and discussion of impact on existing notebooks
3. Migration plan for existing notebooks if breaking changes introduced
4. Approval from project maintainers
5. Version bump following semantic versioning

### Compliance Review

All notebook pull requests MUST:
- Verify adherence to notebook organization principles
- Check for proper documentation and markdown explanations
- Ensure reproducibility requirements are met
- Validate that Python best practices are followed
- Confirm no sensitive data or credentials are included

### Versioning Policy

- **MAJOR**: Breaking changes to notebook structure, Python version changes, or fundamental principle revisions
- **MINOR**: New principles added, expanded quality requirements, or new workflow sections
- **PATCH**: Clarifications, typo fixes, or non-breaking refinements

### Complexity Justification

Any deviation from these principles MUST be:
- Explicitly documented in the notebook with rationale
- Reviewed and approved during code review
- Recorded in a "Technical Debt" section if temporary

**Version**: 1.0.0 | **Ratified**: 2025-10-16 | **Last Amended**: 2025-10-16
