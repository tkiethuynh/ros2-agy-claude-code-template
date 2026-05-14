# ROS2 Clean Architecture Environment

## Project Purpose

This project establishes a standardized, robust, and maintainable environment for developing **ROS2 (Robot Operating System 2)** applications. It is designed to strictly adhere to **Clean Architecture** principles, ensuring that business logic is decoupled from the underlying ROS2 framework.

The primary goal is to provide a comprehensive set of rules, templates, and guidelines that enable developers to build scalable robotic software with consistent patterns in both **Python** and **C++**.

## Key Features

### 1. Clean Architecture Implementation

The project enforces a clear separation of concerns:

- **Domain Layer**: Core business logic and entities (Framework-agnostic).
- **Application Layer**: Use cases and application-specific logic.
- **Infrastructure Layer**: ROS2 adapters, hardware interfaces, and repositories.
- **Presentation Layer**: CLI tools, GUIs, and external APIs.

### 2. Bilingual Support (Python & C++)

Recognizing the dual nature of the ROS2 ecosystem, this environment provides equal support for both languages:

- **Standardized Nodes**: Templates for standard Nodes, Lifecycle Nodes, and Managed Nodes.
- **Communication patterns**: Consistent implementation of Publishers, Subscribers, Services, and Actions.
- **Build Systems**: Best practices for `setup.py` (Python) and `CMakeLists.txt` (C++).

### 3. Comprehensive Rule Set

The `.claude/rules` directory contains detailed guidelines for:

- **Architecture**: Defining layer boundaries and dependency rules.
- **Node Development**: Patterns for creating robust and testable nodes.
- **Communication**: Standards for Topic naming, QoS profiles, and custom interfaces.
- **Testing**: Strategies for Unit (GTest/pytest), Integration, and Launch testing.

### 4. Developer Skills & Templates

A library of "Skills" (`.claude/skills`) provides ready-to-use templates and explanations for:

- **Node Creation & Lifecycle Management**
- **Messaging Patterns (Pub/Sub, Services, Actions)**
- **Launch Configuration & Parameters**
- **TF2 Transforms & Diagnostics**
- **Bag Recording & Replay**
- **Nav 2 plugins** — controller / planner / behavior / smoother / costmap-layer / BT-node scaffolding

### 5. Slash Commands

Executable workflow commands under `.claude/commands/`:

| Command                                | Purpose                                       |
| -------------------------------------- | --------------------------------------------- |
| `/build [pkg]`                         | `colcon build` wrapper (symlink, RelWithDebInfo). |
| `/test [pkg] [filter]`                 | `colcon test` + result summary.               |
| `/lint [--all\|files]`                 | `pre-commit` + ament linters.                 |
| `/new-package <name> <py\|cpp>`        | Scaffold a Clean Architecture package.        |
| `/new-node <pkg> <name> <kind> <lang>` | Scaffold a node (standard / lifecycle / component). |
| `/new-launch <pkg> <name>`             | Scaffold a modular launch file.               |
| `/new-nav2-plugin <kind> <Class>`      | Scaffold a Nav 2 plugin.                      |
| `/changelog [base] [pkg]`              | Generate a CHANGELOG.rst block from commits.  |

### 6. Sub-agents

Specialized agents under `.claude/agents/` for use with the `Agent` tool:

- **`ros2-style-reviewer`** — strict PR review against Clean Architecture, lifecycle, QoS, pluginlib, tests, build manifests. Returns a file:line punch list.
- **`clean-arch-architect`** — design advisor: where does this behaviour belong? node vs use case, topic vs service vs action, compose vs split.

## Getting Started

1.  **Skim `.claude/README.md`** for a complete index of what's available.
2.  **Review the Rules**: Check the `.claude/rules/` directory to understand the architectural standards.
3.  **Use the Skills**: Refer to `.claude/skills/` for implementation examples and templates.
4.  **Try the Slash Commands**: `/build`, `/test`, `/new-package my_pkg python`.
5.  **Run Tests**: Use `colcon test` and `pytest` to verify your implementations.

## Adopting the template in a new workspace

```bash
cd ~/your_ws
cp -r /path/to/ros2-claude-code-template/.claude .
cp /path/to/ros2-claude-code-template/.gitignore .gitignore.template  # merge with yours
```

Then open the workspace with Claude Code — it will pick up
`.claude/CLAUDE.md`, the skills, rules, commands, and agents
automatically.

## License

This project is open-source and available under the Apache 2.0 License.
