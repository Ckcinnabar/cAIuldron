<!--
SYNC IMPACT REPORT
==================
Version Change: 1.0.0 → 1.1.0
Modified Principles:
  - Principle I: "inypb-Only Stack" → "Python & Jupyter Stack"
  - Technology Constraints section updated to reflect Python/Jupyter ecosystem
Added Sections:
  - Jupyter Notebook Standards (new subsection under Technology Constraints)
  - Python & Jupyter Technology Stack (detailed technology requirements)
Removed Sections: N/A
Templates Requiring Updates:
  ✅ .specify/templates/plan-template.md - Constitution check updated (Principle I reference)
  ✅ .specify/templates/spec-template.md - Verified, no changes needed (technology-agnostic)
  ✅ .specify/templates/tasks-template.md - Verified, no changes needed (implementation-agnostic)
  ✅ .specify/templates/agent-file-template.md - Not checked (file status unknown)
  ✅ .specify/templates/checklist-template.md - Not checked (file status unknown)
  ✅ .claude/commands/ - Verified empty, no command files to update
Follow-up TODOs: None
Rationale: This is a MINOR version bump because new technology standards and Jupyter-specific
guidance sections were added, which materially expands the constitutional guidance without
breaking backward compatibility.
==================
-->

# cAIuldron Constitution

## Core Principles

### I. Python & Jupyter Stack (NON-NEGOTIABLE)

The entire project MUST use Python and Jupyter notebooks for all implementations, interfaces,
and integrations.

- **Mandatory**: Every component, feature, library, and tool MUST be built using or compatible
  with Python (3.8+) and Jupyter notebook (.ipynb) formats.
- **No exceptions**: Alternative languages or incompatible notebook formats are prohibited
  unless explicitly approved through constitutional amendment.
- **Validation**: All pull requests MUST verify Python/Jupyter compliance before merge.
- **Documentation**: All technical specifications MUST reference Python and Jupyter best
  practices.

**Rationale**: Maintaining Python and Jupyter as the single technology foundation ensures
simplicity, reduces cognitive overhead, eliminates integration complexity, and enables team
members to focus on delivering value. Python's extensive scientific computing ecosystem and
Jupyter's interactive development environment are ideal for data analysis, experimentation,
and reproducible research.

### II. Library-First Architecture

Every feature starts as a standalone library with clear boundaries and independent testability.

- **Self-contained**: Libraries MUST be independently testable without external dependencies
  where possible.
- **Clear purpose**: Each library MUST solve a specific problem; organizational-only groupings
  are prohibited.
- **Documented**: All libraries MUST include comprehensive documentation of their purpose,
  API, and usage.
- **Reusable**: Libraries MUST be designed for potential reuse across different contexts and
  notebooks.

**Rationale**: Library-first architecture promotes modularity, testability, and reusability
while preventing tight coupling and monolithic design patterns. This is especially important
in notebook-based projects where code can easily become scattered and duplicated.

### III. CLI-First Interface

Every library exposes its functionality through a command-line interface following text I/O
protocols.

- **Standard I/O**: Accept input via stdin/arguments, output results to stdout, errors to
  stderr.
- **Format support**: MUST support both JSON (machine-readable) and human-readable text
  formats.
- **Composability**: CLI tools MUST be composable through standard Unix pipelines.
- **Discoverability**: All CLI commands MUST provide built-in help and usage documentation.

**Rationale**: CLI-first design ensures debuggability, scriptability, and interoperability
while maintaining simplicity and transparency. This enables notebook users to invoke
functionality both programmatically (via Python imports) and through shell commands within
notebook cells.

### IV. Test-First Development (NON-NEGOTIABLE)

Test-Driven Development (TDD) is mandatory for all feature implementations.

- **TDD cycle**: Tests MUST be written first, approved by stakeholders, MUST fail initially,
  then implementation proceeds.
- **Red-Green-Refactor**: Strictly enforce the TDD workflow: failing test → passing code →
  refactoring.
- **Coverage gates**: All code MUST have corresponding tests; untested code cannot be merged.
- **Test types**: Contract tests for interfaces, integration tests for workflows, unit tests
  for logic.

**Rationale**: Test-first development prevents defects, documents behavior, enables confident
refactoring, and ensures specifications are testable before implementation begins. In
notebook environments, tests provide critical validation that cells execute correctly and
produce expected outputs.

### V. Simplicity and Minimalism

Favor simplicity over premature optimization and abstraction.

- **YAGNI principle**: Implement only what is needed now; do not build for hypothetical
  future requirements.
- **Minimal dependencies**: Avoid unnecessary libraries and frameworks; prefer standard
  library solutions and well-established packages.
- **Explicit over implicit**: Favor clear, explicit code over clever abstractions.
- **Complexity justification**: Any complexity introduction MUST be documented and justified
  in the Complexity Tracking section of implementation plans.

**Rationale**: Simplicity reduces maintenance burden, accelerates onboarding, minimizes bugs,
and keeps the codebase understandable and maintainable over time. Jupyter notebooks benefit
from clear, linear logic that can be executed and understood cell by cell.

## Technology Constraints

### Python & Jupyter Technology Stack

All components MUST adhere to the following Python and Jupyter-based technology requirements:

- **Primary language**: Python 3.8 or higher (all implementations)
- **Notebook format**: Jupyter notebook (.ipynb) for interactive development and documentation
- **Package management**: pip with requirements.txt or Poetry/pipenv for dependency management
- **Virtual environments**: Use venv, virtualenv, or conda for environment isolation
- **Testing framework**: pytest for Python modules; nbval or papermill for notebook testing
- **Documentation**: Follow Python docstring conventions (PEP 257) and include markdown cells
  in notebooks
- **Code style**: Follow PEP 8 style guide; enforce with black, flake8, or ruff
- **Type hints**: Encourage type annotations for function signatures (PEP 484)

### Jupyter Notebook Standards

All notebooks MUST follow these conventions:

- **Clear structure**: Use markdown cells to organize notebooks into logical sections with
  headings
- **Cell organization**: Keep cells focused on single tasks; avoid overly long cells
- **Output management**: Clear outputs before committing to version control (or use tools
  like nbstripout)
- **Reproducibility**: Notebooks MUST be executable from top to bottom without errors
- **Dependencies**: Document all package requirements and versions at the start of notebooks
- **Kernel specification**: Clearly specify which Python kernel/environment to use

### Prohibited Technologies

The following are explicitly prohibited unless granted a constitutional exception:

- Non-Python languages (except for unavoidable system integration or build tools)
- Non-Jupyter notebook formats incompatible with .ipynb
- Technologies that cannot be invoked from Python or Jupyter environments
- Frameworks that require abandoning the Jupyter development paradigm

### Exception Process

Technology exceptions require:

1. Written justification documenting technical necessity
2. Proof that Python/Jupyter-native solution is impossible or severely inadequate
3. Constitutional amendment proposal with stakeholder approval
4. Migration plan for eventual return to Python/Jupyter if feasible

## Development Workflow

### Feature Development Lifecycle

All features MUST follow the speckit workflow:

1. **Specification** (`/speckit.specify`): Document user scenarios and requirements
2. **Planning** (`/speckit.plan`): Research and design technical approach
3. **Task Generation** (`/speckit.tasks`): Create dependency-ordered implementation tasks
4. **Implementation** (`/speckit.implement`): Execute tasks with continuous testing
5. **Review**: Constitution compliance check before merge

### Code Review Requirements

All pull requests MUST verify:

- ✅ Python & Jupyter compliance (Principle I)
- ✅ Library-first architecture (Principle II)
- ✅ CLI interface implementation (Principle III)
- ✅ Test coverage and TDD adherence (Principle IV)
- ✅ Simplicity justification for complex code (Principle V)

### Quality Gates

Code cannot be merged unless:

- All tests pass (contract, integration, unit, notebook execution)
- Constitutional principles are satisfied
- Documentation is complete and accurate
- Notebooks are reproducible (execute cleanly from top to bottom)
- Code review approval obtained

## Governance

### Constitutional Authority

This constitution supersedes all other development practices, guidelines, and conventions.
When conflicts arise, constitutional principles take precedence.

### Amendment Process

Constitutional amendments require:

1. **Proposal**: Written amendment with rationale and impact analysis
2. **Review**: Technical feasibility assessment and stakeholder consultation
3. **Approval**: Consensus from project maintainers
4. **Migration**: Implementation plan for transitioning existing code if needed
5. **Documentation**: Update all dependent templates and documentation

### Versioning Policy

- **MAJOR** (X.0.0): Backward-incompatible changes, principle removals, fundamental
  redefinitions
- **MINOR** (x.Y.0): New principles added, sections expanded, material guidance updates
- **PATCH** (x.y.Z): Clarifications, wording improvements, non-semantic fixes

### Compliance Review

- Constitution compliance MUST be verified at every code review
- Violations MUST be documented in the Complexity Tracking section of implementation plans
- Unjustified violations block merge approval
- Recurring violations trigger constitutional review

### Runtime Guidance

For detailed development guidance and agent-specific instructions, refer to
`.claude/agent-file.md` (if present) or project-specific documentation in `docs/`.

---

**Version**: 1.1.0 | **Ratified**: 2025-10-16 | **Last Amended**: 2025-10-16
