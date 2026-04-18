# Usage Flows

This document summarizes the user-facing usage flows currently implemented in Spec Kit, based on the repository's CLI, templates, reference docs, and built-in workflows.

## Flow 1: Install and verify the CLI

**Purpose:** get `specify` available locally.

**Typical entry points:**

- Persistent install with `uv tool install`
- One-shot execution with `uvx`
- Air-gapped installation from built wheels

**Key outputs:**

- `specify` executable in PATH, or an ephemeral `uvx` execution context
- Ability to run `specify version` and `specify check`

```mermaid
flowchart TD
    A[Choose install mode] --> B{Persistent, one-shot,<br/>or air-gapped?}
    B -->|Persistent| C[uv tool install specify-cli]
    B -->|One-shot| D[uvx --from ... specify ...]
    B -->|Air-gapped| E[Build/download wheels and install offline]
    C --> F[Run specify version]
    D --> F
    E --> F
    F --> G[Run specify check]
```

## Flow 2: Initialize a project with `specify init`

**Purpose:** bootstrap a repository for Spec-Driven Development.

**Variants supported by the CLI:**

- Create a new directory or initialize the current one with `--here` / `.`
- Choose one integration with `--integration`
- Choose script flavor with `--script sh|ps`
- Skip Git with `--no-git`
- Install an initial preset with `--preset`
- Use sequential or timestamp branch numbering

**Key outputs:**

- `.specify/` workspace
- templates, scripts, memory files, and optional Git setup
- command files for the selected agent integration

```mermaid
flowchart TD
    A[specify init] --> B{New directory or current directory?}
    B -->|New| C[Create project folder]
    B -->|Current| D[Initialize in place]
    C --> E[Install shared Spec Kit assets]
    D --> E
    E --> F[Install selected integration]
    F --> G{Preset requested?}
    G -->|Yes| H[Install preset]
    G -->|No| I[Skip preset]
    H --> J{Git enabled?}
    I --> J
    J -->|Yes| K[Initialize or reuse Git repo]
    J -->|No| L[Skip Git initialization]
    K --> M[Project ready]
    L --> M
```

## Flow 3: Establish the project constitution

**Purpose:** define the non-negotiable principles that govern later phases.

**Primary command:** `/speckit.constitution`

**Key output:** `.specify/memory/constitution.md`

```mermaid
flowchart TD
    A[Open project in the AI agent] --> B[/speckit.constitution/]
    B --> C[Capture principles and governance]
    C --> D[Write or update constitution.md]
    D --> E[Use constitution as input for later commands]
```

## Flow 4: Create and refine the feature specification

**Purpose:** transform a feature idea into a structured `spec.md`.

**Primary commands:**

- `/speckit.specify`
- `/speckit.clarify` for iterative ambiguity reduction
- `/speckit.checklist` for additional requirement-quality checks

**Key outputs:**

- feature branch and `specs/<feature>/spec.md`
- review checklists under `specs/<feature>/checklists/`

```mermaid
flowchart TD
    A[Feature idea] --> B[/speckit.specify/]
    B --> C[Create feature branch and spec.md]
    C --> D{Spec still ambiguous?}
    D -->|Yes| E[/speckit.clarify/]
    E --> F[Update clarifications in spec]
    F --> D
    D -->|No| G{Need a custom validation lens?}
    G -->|Yes| H[/speckit.checklist/]
    G -->|No| I[Spec ready for planning]
    H --> I
```

## Flow 5: Generate the implementation plan

**Purpose:** translate approved requirements into technical design artifacts.

**Primary command:** `/speckit.plan`

**Key outputs:**

- `plan.md`
- `research.md`
- `data-model.md`
- `contracts/`
- `quickstart.md`

```mermaid
flowchart TD
    A[Approved spec] --> B[/speckit.plan/]
    B --> C[Read constitution and spec]
    C --> D[Choose architecture and stack]
    D --> E[Generate plan.md]
    E --> F[Generate research, models, contracts, quickstart]
    F --> G{Plan acceptable?}
    G -->|No| H[Refine spec or plan and rerun]
    H --> B
    G -->|Yes| I[Plan ready for task generation]
```

## Flow 6: Generate tasks and validate artifact coverage

**Purpose:** derive an executable work backlog from the plan and validate cross-artifact consistency.

**Primary commands:**

- `/speckit.tasks`
- `/speckit.analyze` (recommended after tasks)

**Key outputs:**

- `tasks.md`
- coverage and consistency feedback before implementation

```mermaid
flowchart TD
    A[Plan artifacts ready] --> B[/speckit.tasks/]
    B --> C[Generate dependency-ordered tasks.md]
    C --> D{Run quality analysis?}
    D -->|Yes| E[/speckit.analyze/]
    E --> F{Critical issues found?}
    F -->|Yes| G[Revise spec, plan, or tasks]
    G --> B
    F -->|No| H[Tasks approved]
    D -->|No| H
```

## Flow 7: Convert tasks into GitHub issues

**Purpose:** hand off a finished task plan into repository issue tracking.

**Primary command:** `/speckit.taskstoissues`

**Preconditions:**

- `tasks.md` exists
- repository remote is GitHub

**Key outputs:** one GitHub issue per actionable task

```mermaid
flowchart TD
    A[tasks.md exists] --> B[/speckit.taskstoissues/]
    B --> C[Validate prerequisites and Git remote]
    C --> D{Remote is GitHub?}
    D -->|No| E[Abort safely]
    D -->|Yes| F[Create issues from tasks]
    F --> G[Track execution in GitHub]
```

## Flow 8: Implement the feature

**Purpose:** execute the generated task plan.

**Primary command:** `/speckit.implement`

**What the flow does:**

- validates required artifacts
- checks checklist completeness
- follows task ordering and `[P]` parallel markers
- executes work against the files listed in `tasks.md`

```mermaid
flowchart TD
    A[spec.md + plan.md + tasks.md ready] --> B[/speckit.implement/]
    B --> C[Validate prerequisites and checklists]
    C --> D{Prerequisites satisfied?}
    D -->|No| E[Stop and request fixes]
    D -->|Yes| F[Parse tasks.md]
    F --> G[Execute tasks in order]
    G --> H{Implementation complete?}
    H -->|No| I[Continue remaining tasks]
    I --> G
    H -->|Yes| J[Feature implemented]
```

## Flow 9: Manage AI agent integrations

**Purpose:** install, switch, remove, or upgrade the active agent integration.

**CLI surface:**

- `specify integration list`
- `specify integration install`
- `specify integration uninstall`
- `specify integration switch`
- `specify integration upgrade`

**Important rule:** only one integration is active per project at a time.

```mermaid
flowchart TD
    A[Need an agent integration] --> B[specify integration list]
    B --> C{Already installed?}
    C -->|No| D[specify integration install <key>]
    C -->|Yes, same agent| E[specify integration upgrade]
    C -->|Yes, different agent| F[specify integration switch <key>]
    D --> G[Commands and context files installed]
    E --> G
    F --> G
    G --> H{Remove integration?}
    H -->|Yes| I[specify integration uninstall]
    H -->|No| J[Continue using project]
```

## Flow 10: Customize the workflow with presets

**Purpose:** change how Spec Kit resolves templates, commands, and scripts without adding new capabilities.

**CLI surface:**

- `specify preset search`
- `specify preset add`
- `specify preset list`
- `specify preset info`
- `specify preset resolve`
- `specify preset enable|disable`
- `specify preset set-priority`
- `specify preset remove`

**Key behavior:** multiple presets can coexist and are resolved by priority.

```mermaid
flowchart TD
    A[Need to customize default behavior] --> B[specify preset search]
    B --> C[specify preset add <id>]
    C --> D[Preset registered in project]
    D --> E{Need to inspect or tune precedence?}
    E -->|Inspect| F[specify preset resolve / info / list]
    E -->|Change order| G[specify preset set-priority]
    E -->|Temporarily disable| H[specify preset disable]
    E -->|Remove| I[specify preset remove]
    F --> J[Runtime resolution uses winning file]
    G --> J
    H --> J
    I --> J
```

## Flow 11: Extend Spec Kit with extensions

**Purpose:** add new commands, hooks, or external integrations beyond the built-in SDD flow.

**CLI surface:**

- `specify extension search`
- `specify extension add`
- `specify extension list`
- `specify extension info`
- `specify extension update`
- `specify extension enable|disable`
- `specify extension set-priority`
- `specify extension remove`

**Key behavior:** multiple extensions can coexist and can attach hooks to core commands.

```mermaid
flowchart TD
    A[Need new capability] --> B[specify extension search]
    B --> C[specify extension add <name>]
    C --> D[Commands and config installed]
    D --> E{Operate extension?}
    E -->|Inspect| F[specify extension list / info]
    E -->|Update| G[specify extension update]
    E -->|Temporarily disable| H[specify extension disable]
    E -->|Change precedence| I[specify extension set-priority]
    E -->|Remove| J[specify extension remove]
    F --> K[Project continues with selected extension state]
    G --> K
    H --> K
    I --> K
    J --> K
```

## Flow 12: Automate the lifecycle with workflows

**Purpose:** run a resumable multi-step automation pipeline instead of manually invoking each command.

**CLI surface:**

- `specify workflow search`
- `specify workflow add`
- `specify workflow run`
- `specify workflow status`
- `specify workflow resume`
- `specify workflow info`
- `specify workflow remove`

**Supported sources:** installed workflow ID, HTTPS URL, or local YAML file.

```mermaid
flowchart TD
    A[Need end-to-end automation] --> B{Installed workflow or local YAML?}
    B -->|Installed| C[specify workflow add <id>]
    B -->|Local| D[Use local workflow file]
    C --> E[specify workflow run <source>]
    D --> E
    E --> F[Engine executes steps and persists state]
    F --> G{Gate or failure encountered?}
    G -->|Gate pause| H[specify workflow status]
    H --> I[Human review]
    I --> J[specify workflow resume <run_id>]
    J --> F
    G -->|No| K[Workflow completed]
```

## Flow 13: Operate and maintain the installation

**Purpose:** keep the local setup healthy as the project evolves.

**Common commands:**

- `specify check`
- `specify version`
- `specify integration upgrade`
- `specify extension update`
- `specify preset resolve`
- `specify workflow status`

```mermaid
flowchart TD
    A[Ongoing project maintenance] --> B[specify check]
    B --> C[specify version]
    C --> D{Need refresh or diagnosis?}
    D -->|Integration drift| E[specify integration upgrade]
    D -->|Extension drift| F[specify extension update]
    D -->|Template precedence doubt| G[specify preset resolve]
    D -->|Workflow run follow-up| H[specify workflow status]
    E --> I[Project stays aligned]
    F --> I
    G --> I
    H --> I
```

## Recommended end-to-end path

For most teams, the default path through the repository is:

1. Install the CLI
2. Run `specify init`
3. Use `/speckit.constitution`
4. Use `/speckit.specify`
5. Iterate with `/speckit.clarify` and `/speckit.checklist` as needed
6. Use `/speckit.plan`
7. Use `/speckit.tasks`
8. Run `/speckit.analyze`
9. Optionally run `/speckit.taskstoissues`
10. Use `/speckit.implement`
11. Layer presets, extensions, or workflows when the team needs more automation or customization
