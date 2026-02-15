---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments: ['_bmad-output/planning-artifacts/prd.md']
workflowType: 'architecture'
project_name: 'smarttask'
user_name: 'zunaisha'
date: '2026-02-14T19:53:05+05:00'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
Smart Task Vault requires 21 functional capabilities organized into 5 core areas:

1. **Task Management (FR1-FR5)**: Basic CRUD operations for tasks with priority levels (high/medium/low) and JSON-based persistence across sessions
2. **Priority & Sorting (FR6-FR8)**: Anti-gravity sorting algorithm that automatically positions tasks by priority (high floats to top, low sinks to bottom) and maintains order dynamically
3. **Visual Display (FR9-FR13)**: Terminal-based UI with color-coded priority visualization (bold/highlighted for high, standard for medium, dimmed for low)
4. **Data Persistence (FR14-FR17)**: JSON file storage with graceful error handling for missing/corrupted files, supporting 50+ tasks without degradation
5. **User Experience (FR18-FR21)**: Sub-5-second task addition, instant display, zero configuration, intuitive command syntax

**Non-Functional Requirements:**
Critical NFRs that will drive architectural decisions:

- **Performance (NFR1-NFR5)**: Task addition < 5s, display < 2s, command execution < 100ms, startup < 500ms, efficient sorting for 50+ tasks
- **Reliability (NFR6-NFR10)**: Atomic JSON writes, graceful error recovery, cross-platform terminal compatibility (Windows/macOS/Linux)
- **Usability (NFR11-NFR14)**: Zero configuration, intuitive syntax, user-friendly error messages (no stack traces), immediately recognizable visual indicators

**Scale & Complexity:**

- **Primary domain**: CLI Application / Developer Productivity Tool
- **Complexity level**: Low-Medium (single-user, local-only, file-based storage)
- **Estimated architectural components**: 4-5 core modules
  - CLI command parser
  - Task management engine
  - Anti-gravity sorting algorithm
  - JSON persistence layer
  - Terminal output formatter

### Technical Constraints & Dependencies

**Language & Platform:**
- Python-based CLI tool
- Cross-platform compatibility required (Windows, macOS, Linux)
- Lightweight CLI library (argparse, click, or typer)
- Terminal color library (colorama, rich, or termcolor)

**Storage:**
- JSON file storage (`tasks.json`)
- No database, no networking
- Atomic write operations required
- File location: current directory or user home directory

**Performance Constraints:**
- Command execution must be near-instant (< 100ms for view)
- Sorting algorithm must handle 50+ tasks efficiently
- Minimal startup time (< 500ms)

**Deployment:**
- Zero configuration requirement
- No setup scripts or initialization needed
- Works immediately after installation

### Cross-Cutting Concerns Identified

1. **Anti-Gravity Sorting Algorithm**
   - Core product differentiator
   - Must be fast (< 100ms) and efficient for 50+ tasks
   - Affects all task views and operations
   - Needs to maintain sort order dynamically as tasks are added/completed

2. **Terminal Output Formatting**
   - Color-coding and text styling (bold/dimmed)
   - Cross-platform terminal compatibility
   - Graceful degradation if colors not supported
   - Visual hierarchy reinforcing physics metaphor

3. **Error Handling & Recovery**
   - Graceful handling of missing/corrupted JSON files
   - User-friendly error messages (no technical stack traces)
   - Data integrity protection during crashes

4. **Data Integrity**
   - Atomic file writes to prevent corruption
   - Backup/recovery mechanisms for corrupted data
   - Validation of JSON structure on load

5. **Performance Optimization**
   - Efficient sorting algorithm selection
   - Minimal file I/O overhead
   - Fast startup and command execution

## Starter Template Evaluation

### Primary Technology Domain

**CLI Application** - Python-based developer productivity tool

### Technology Stack Selection

After researching current Python CLI best practices (2026), the following modern stack is recommended:

**Core Framework:**
- **Typer** (latest: install via `pip install "typer[all]"`)
  - Modern CLI framework using Python type hints
  - Auto-generates help text and validates input
  - Includes `rich` and `shellingham` with `[all]` installation
  - Built on Click but more Pythonic

**Terminal Output:**
- **Rich** (latest: v14.3.2, released Feb 2026)
  - Beautiful terminal formatting with colors, tables, progress bars
  - Perfect for visualizing the "anti-gravity" metaphor
  - Cross-platform terminal compatibility
  - Enhanced Unicode/emoji support, nested live objects

**Testing:**
- **Pytest** - Industry standard testing framework
  - Works seamlessly with Typer's `CliRunner`
  - Fixtures for efficient test setup
  - Coverage integration with `pytest-cov`

**Code Quality:**
- **Ruff** - Fast modern linter and formatter (combines Black, Flake8, isort)
- **MyPy** - Static type checking
- **Pre-commit hooks** - Automated quality checks

### Project Structure Decision

**Modern `src` Layout** (2026 best practice):

```
smarttask/
├── pyproject.toml          # Single source of truth for config
├── README.md
├── LICENSE
├── .gitignore
├── src/
│   └── smarttask/
│       ├── __init__.py
│       ├── cli.py          # Typer CLI entry point
│       ├── core/
│       │   ├── task.py     # Task model
│       │   └── sorting.py  # Anti-gravity algorithm
│       ├── storage/
│       │   └── json_store.py  # JSON persistence
│       └── ui/
│           └── formatter.py   # Rich terminal output
└── tests/
    ├── test_cli.py
    ├── test_sorting.py
    └── test_storage.py
```

### Rationale for "No Starter Template"

For this project, building **from scratch** rather than using a cookiecutter template because:

1. **Simplicity**: Project is intentionally lean (4 commands, MVP focus)
2. **Learning Goal**: Mastering GitHub workflow - building from scratch provides more practice
3. **Zero Dependencies**: Starter templates often include unnecessary dependencies
4. **Custom Structure**: Anti-gravity sorting is unique and doesn't fit standard templates

### Initial Setup Commands

**Project Initialization:**

```bash
# Create project structure
mkdir smarttask
cd smarttask

# Create src layout
mkdir -p src/smarttask/{core,storage,ui}
mkdir tests

# Initialize pyproject.toml
# (Manual creation recommended for learning)

# Install dependencies
pip install "typer[all]" pytest pytest-cov ruff mypy

# Initialize git
git init
```

### Architectural Decisions Established

**Language & Runtime:**
- Python 3.10+ (for modern type hints)
- Type hints throughout for Typer integration
- MyPy for static type checking

**CLI Framework:**
- Typer for command parsing and validation
- Rich included via `typer[all]` for terminal output
- No shell completion in MVP (deferred to Phase 2)

**Project Organization:**
- `src/` layout for clean separation
- Modular structure: `core/`, `storage/`, `ui/`
- Tests separate from source code

**Development Tools:**
- Ruff for linting and formatting
- Pytest for testing with CLI runner
- Pre-commit hooks for quality gates
- `pyproject.toml` as single config file

**Build & Packaging:**
- Modern `pyproject.toml` (no `setup.py`)
- Entry point: `smarttask.cli:main`
- Installable via `pip install -e .` for development

**Note:** Project initialization should be the first implementation task, setting up the structure before implementing features.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
1. ✅ Data Model: Pydantic BaseModel with type safety
2. ✅ Anti-Gravity Algorithm: Physics-inspired gravity score calculation
3. ✅ JSON Storage: Atomic writes with temp file pattern
4. ✅ Error Handling: Custom exceptions with Rich console output
5. ✅ Testing Strategy: Comprehensive pytest suite with 80%+ coverage

**Deferred Decisions (Post-MVP):**
- Deployment/packaging strategy (Phase 2: Django web dashboard)
- CI/CD pipeline setup (can add incrementally)
- Performance profiling (optimize if needed after MVP)

### Data Architecture

**Task Data Model:**
- **Technology**: Pydantic v2.x (latest stable)
- **Structure**: 
  ```python
  class Priority(str, Enum):
      HIGH = "high"
      MEDIUM = "medium"
      LOW = "low"
  
  class Task(BaseModel):
      id: str  # UUID4
      description: str
      priority: Priority
      completed: bool = False
      created_at: datetime
  ```
- **Rationale**: Type safety, automatic validation, JSON serialization built-in
- **Validation**: Pydantic handles validation automatically (empty descriptions rejected, priority enum enforced)

**Storage Strategy:**
- **Format**: JSON file (`tasks.json`)
- **Location**: User home directory (`~/.smarttask/tasks.json`) for persistence
- **Atomic Writes**: Temp file + `os.replace()` pattern prevents corruption
- **Implementation**:
  ```python
  def save_tasks_atomic(tasks: List[Task], filepath: Path):
      temp_fd, temp_path = tempfile.mkstemp(dir=filepath.parent)
      try:
          with os.fdopen(temp_fd, 'w') as f:
              json.dump([t.model_dump() for t in tasks], f, indent=2)
          os.replace(temp_path, filepath)  # Atomic operation
      except Exception:
          os.unlink(temp_path)
          raise
  ```
- **Meets**: NFR6 (atomic writes), NFR7 (graceful handling of missing files)

### Core Algorithm: Anti-Gravity Sorting

**Implementation Approach:**
- **Algorithm**: Physics-inspired gravity score calculation
- **Formula**:
  ```python
  def calculate_gravity_score(task: Task) -> float:
      base_weight = {
          Priority.HIGH: 0.0,    # zero gravity - floats
          Priority.MEDIUM: 5.0,
          Priority.LOW: 10.0     # heavy - sinks
      }
      
      # Optional: Time decay (older tasks get slightly heavier)
      age_hours = (datetime.now() - task.created_at).total_seconds() / 3600
      time_factor = age_hours * 0.01
      
      return base_weight[task.priority] + time_factor
  
  # Sort: lower score = higher position (floats to top)
  tasks.sort(key=calculate_gravity_score)
  ```
- **Performance**: O(n log n), handles 50+ tasks in < 100ms
- **Extensibility**: Time decay factor allows future enhancements (urgency, deadlines)
- **Meets**: FR6, FR7, FR8 (anti-gravity sorting logic)

### Error Handling & User Experience

**Exception Hierarchy:**
```python
class TaskVaultError(Exception):
    """Base exception for all vault errors"""
    pass

class CorruptedDataError(TaskVaultError):
    """Raised when JSON is corrupted"""
    pass

class InvalidTaskError(TaskVaultError):
    """Raised when task validation fails"""
    pass
```

**User-Friendly Output:**
- **Technology**: Rich Console for colored, formatted output
- **Strategy**: No stack traces shown to users (logged internally)
- **Examples**:
  - Missing file: `[yellow]No vault found. Creating new task vault...[/yellow]`
  - Corrupted data: `[red]Error: Task vault is corrupted. Creating backup...[/red]`
  - Success: `[green]✓ Task added successfully![/green]`
- **Meets**: NFR9, NFR12 (clear, actionable, user-friendly errors)

### Testing Strategy

**Test Structure:**
```
tests/
├── test_task_model.py      # Pydantic validation, Task creation
├── test_sorting.py          # Anti-gravity algorithm correctness
├── test_storage.py          # Atomic writes, file recovery, corruption handling
└── test_cli.py              # End-to-end CLI command testing
```

**Testing Approach:**
- **Unit Tests**: Core logic (sorting, model validation)
- **Integration Tests**: Storage operations (atomic writes, recovery)
- **CLI Tests**: Typer's `CliRunner` for command simulation
- **Coverage Target**: 80%+ for MVP (focus on critical paths)
- **Framework**: Pytest with fixtures for test data

**Test Priorities:**
1. Anti-gravity sorting correctness (all priority combinations)
2. Atomic write operations (simulate crashes)
3. Corrupted file recovery
4. CLI command parsing and execution

### Decision Impact Analysis

**Implementation Sequence:**
1. **First**: Set up project structure (`src/` layout, `pyproject.toml`)
2. **Second**: Implement Task model (Pydantic) + unit tests
3. **Third**: Implement anti-gravity sorting algorithm + tests
4. **Fourth**: Implement JSON storage with atomic writes + tests
5. **Fifth**: Implement CLI commands (Typer) with Rich output
6. **Sixth**: Integration testing and error handling

**Cross-Component Dependencies:**
- Task model → Used by sorting, storage, and CLI
- Sorting algorithm → Depends on Task model, used by CLI `view` command
- Storage layer → Depends on Task model, used by all CLI commands
- Error handling → Used across all components

**Technology Dependencies:**
```toml
[project]
dependencies = [
    "typer[all]>=0.9.0",  # Includes rich, shellingham
    "pydantic>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-cov>=4.0.0",
    "ruff>=0.1.0",
    "mypy>=1.0.0",
]
```

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 6 areas where AI agents could make different implementation choices

**Applicable Patterns:**
1. ✅ Naming Conventions (Python code, CLI commands, tests)
2. ✅ Project Structure (module organization, file placement)
3. ✅ Error Handling (exception hierarchy, user messages)
4. ✅ Data Formats (JSON structure, field naming)
5. ✅ Testing Patterns (test structure, fixture organization)
6. ✅ CLI Command Patterns (Typer conventions)

**Not Applicable (CLI Tool):**
- ❌ API endpoints (no web API)
- ❌ Database schemas (using JSON files)
- ❌ Frontend components (terminal-only)
- ❌ State management (stateless CLI)

### Naming Patterns

**Python Code Naming Conventions:**

- **Modules/Files**: `snake_case`
  - Examples: `task_model.py`, `json_store.py`, `gravity_sorting.py`
- **Classes**: `PascalCase`
  - Examples: `Task`, `TaskVault`, `CorruptedDataError`, `Priority`
- **Functions/Methods**: `snake_case`
  - Examples: `calculate_gravity_score()`, `save_tasks_atomic()`, `load_vault()`
- **Variables**: `snake_case`
  - Examples: `task_list`, `gravity_score`, `file_path`, `created_at`
- **Constants**: `UPPER_SNAKE_CASE`
  - Examples: `PRIORITY_WEIGHT`, `DEFAULT_VAULT_PATH`, `MAX_TASKS`
- **Private Methods**: `_leading_underscore`
  - Examples: `_validate_task()`, `_backup_corrupted_file()`

**CLI Command Naming Conventions:**

- **Commands**: lowercase, single word
  - Examples: `add`, `view`, `complete`, `delete`
- **Options/Flags**: kebab-case with double dash
  - Examples: `--priority`, `--task-id`, `--show-completed`
- **Short Flags**: single letter with single dash
  - Examples: `-p` (priority), `-a` (all), `-h` (help)
- **Command Format**: `smarttask <command> <args> [options]`
  - Example: `smarttask add "Fix bug" --priority high`

**Test Naming Conventions:**

- **Test Files**: `test_<module>.py`
  - Examples: `test_sorting.py`, `test_storage.py`, `test_cli.py`
- **Test Functions**: `test_<component>_<scenario>_<expected>`
  - Examples: `test_sorting_high_priority_floats_to_top()`, `test_storage_atomic_write_prevents_corruption()`
- **Test Fixtures**: `snake_case`
  - Examples: `sample_tasks`, `temp_vault_file`, `mock_console`

### Structure Patterns

**Project Organization:**

```
src/smarttask/
├── __init__.py           # Package initialization
├── cli.py                # Typer CLI entry point (ALL commands here)
├── core/                 # Business logic (NO I/O)
│   ├── __init__.py
│   ├── task.py           # Task model (Pydantic)
│   └── sorting.py        # Anti-gravity algorithm
├── storage/              # Data persistence (ALL I/O)
│   ├── __init__.py
│   └── json_store.py     # JSON file operations
├── ui/                   # User interface (terminal output)
│   ├── __init__.py
│   └── formatter.py      # Rich console formatting
└── exceptions.py         # Custom exception hierarchy
```

**Module Responsibility Rules:**

- **`core/`**: Pure business logic, NO file I/O, NO console output
- **`storage/`**: ALL file operations, database interactions (if added later)
- **`ui/`**: ALL user-facing output, Rich console formatting
- **`cli.py`**: Command definitions ONLY, delegates to core/storage/ui
- **`exceptions.py`**: ALL custom exceptions in one file

**Test Organization:**

```
tests/
├── conftest.py           # Shared pytest fixtures
├── test_task_model.py    # Task Pydantic model tests
├── test_sorting.py       # Anti-gravity algorithm tests
├── test_storage.py       # JSON persistence tests
└── test_cli.py           # End-to-end CLI command tests
```

### Format Patterns

**JSON Structure (tasks.json):**

```json
{
  "tasks": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "description": "Fix critical bug in sorting",
      "priority": "high",
      "completed": false,
      "created_at": "2026-02-14T20:00:00+05:00"
    }
  ],
  "version": "1.0"
}
```

**JSON Field Naming Rules:**

- Field names: `snake_case` (e.g., `created_at`, `task_id`)
- Dates: ISO 8601 format with timezone (e.g., `2026-02-14T20:00:00+05:00`)
- Booleans: `true`/`false` (NOT 1/0 or "true"/"false")
- IDs: UUID4 strings (NOT integers)
- Enums: lowercase strings (e.g., `"high"`, `"medium"`, `"low"`)

**Rich Console Output Patterns:**

```python
from rich.console import Console
console = Console()

# Success messages
console.print("[green]✓[/green] Task added successfully!")

# Error messages
console.print("[red]✗ Error:[/red] Task vault is corrupted")

# Warning messages
console.print("[yellow]⚠ Warning:[/yellow] No tasks found")

# Info messages
console.print("[blue]ℹ[/blue] Loading task vault...")
```

### Process Patterns

**Error Handling Patterns:**

**Exception Hierarchy (MUST follow):**
```python
class TaskVaultError(Exception):
    """Base exception - ALL custom exceptions inherit from this"""
    pass

class CorruptedDataError(TaskVaultError):
    """Raised when JSON file is corrupted"""
    pass

class InvalidTaskError(TaskVaultError):
    """Raised when task validation fails"""
    pass
```

**Error Handling Rules:**

1. **NEVER show stack traces to users** - Use Rich console for user-friendly messages
2. **Always catch specific exceptions** - Don't use bare `except:`
3. **Log errors internally** - Use Python logging for debugging
4. **Provide actionable messages** - Tell users what to do next

**Example:**
```python
try:
    tasks = load_tasks(vault_path)
except FileNotFoundError:
    console.print("[yellow]No vault found. Creating new task vault...[/yellow]")
    tasks = []
except CorruptedDataError:
    console.print("[red]✗ Error:[/red] Task vault is corrupted. Creating backup...")
    backup_and_recreate_vault()
except Exception as e:
    logger.error(f"Unexpected error: {e}")  # Log for debugging
    console.print("[red]✗ Error:[/red] Something went wrong. Please try again.")
```

**Testing Patterns:**

**Test Structure (Arrange-Act-Assert):**
```python
def test_sorting_high_priority_floats_to_top():
    # Arrange
    high_task = Task(id="1", description="High", priority=Priority.HIGH, ...)
    low_task = Task(id="2", description="Low", priority=Priority.LOW, ...)
    tasks = [low_task, high_task]  # Intentionally wrong order
    
    # Act
    sorted_tasks = sort_by_gravity(tasks)
    
    # Assert
    assert sorted_tasks[0].priority == Priority.HIGH
    assert sorted_tasks[1].priority == Priority.LOW
```

**Fixture Pattern:**
```python
@pytest.fixture
def sample_tasks():
    """Provide consistent test data across all tests"""
    return [
        Task(id="1", description="High priority", priority=Priority.HIGH, 
             completed=False, created_at=datetime.now()),
        Task(id="2", description="Medium priority", priority=Priority.MEDIUM, 
             completed=False, created_at=datetime.now()),
        Task(id="3", description="Low priority", priority=Priority.LOW, 
             completed=False, created_at=datetime.now()),
    ]
```

**CLI Command Patterns:**

**Typer Command Structure:**
```python
@app.command()
def add(
    description: str = typer.Argument(..., help="Task description"),
    priority: str = typer.Option("medium", "--priority", "-p", 
                                   help="Priority: high, medium, or low")
):
    """Add a new task to the vault.
    
    Example: smarttask add "Fix bug" --priority high
    """
    # Implementation delegates to core/storage/ui
```

**CLI Pattern Rules:**

1. **All commands have docstrings** - Shown in `--help`
2. **Required parameters use `Argument`** - Positional
3. **Optional parameters use `Option`** - With defaults and flags
4. **Short flags are single letter** - e.g., `-p`, `-a`
5. **Long flags are full words** - e.g., `--priority`, `--all`
6. **Help text is concise** - One sentence per parameter

### Enforcement Guidelines

**All AI Agents MUST:**

1. **Follow PEP 8 naming conventions** - Enforced by Ruff linter
2. **Use type hints on ALL functions** - Enforced by MyPy
3. **Place code in correct modules** - `core/` for logic, `storage/` for I/O, `ui/` for output
4. **Use Rich console for ALL user-facing output** - No plain `print()` statements
5. **Write tests following Arrange-Act-Assert** - Clear test structure
6. **Never show stack traces to users** - User-friendly error messages only
7. **Use Pydantic models for data validation** - Automatic type checking
8. **Implement atomic writes for file operations** - Prevent data corruption

**Pattern Enforcement Tools:**

- **Ruff**: Checks naming, formatting, imports (runs in pre-commit)
- **MyPy**: Enforces type hints (runs in pre-commit)
- **Pytest**: Runs all tests (runs in pre-commit)
- **Pre-commit hooks**: Automatically run all checks before commits

**Pattern Verification:**

```bash
# Run all checks manually
ruff check src/ tests/
mypy src/
pytest tests/ --cov=src --cov-report=term-missing
```

### Pattern Examples

**Good Example - Following All Patterns:**

```python
# src/smarttask/core/sorting.py
from datetime import datetime
from typing import List
from .task import Task, Priority

PRIORITY_WEIGHT = {
    Priority.HIGH: 0.0,
    Priority.MEDIUM: 5.0,
    Priority.LOW: 10.0,
}

def calculate_gravity_score(task: Task) -> float:
    """Calculate anti-gravity score for task sorting.
    
    Lower score = floats higher (zero gravity)
    Higher score = sinks lower (heavy weight)
    """
    base_weight = PRIORITY_WEIGHT[task.priority]
    age_hours = (datetime.now() - task.created_at).total_seconds() / 3600
    time_factor = age_hours * 0.01
    return base_weight + time_factor

def sort_by_gravity(tasks: List[Task]) -> List[Task]:
    """Sort tasks by anti-gravity score."""
    return sorted(tasks, key=calculate_gravity_score)
```

**Anti-Pattern - What to Avoid:**

```python
# ❌ BAD: Wrong naming, no type hints, mixed concerns
def Sort(TaskList):  # PascalCase function, no types
    print("Sorting tasks...")  # Direct print, not Rich console
    for t in TaskList:  # Single letter variable
        with open('tasks.json', 'w') as f:  # I/O in core logic
            pass
    return TaskList
```
