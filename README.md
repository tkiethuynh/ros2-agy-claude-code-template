# ROS 2 + Gazebo Clean Architecture Environment

## Project Purpose

This project establishes a standardized, robust, and maintainable environment for developing **two adjacent worlds** that robotics teams cross between every day:

1. **ROS 2 (Robot Operating System 2)** applications — designed to strictly adhere to **Clean Architecture** principles so business logic is decoupled from `rclpy` / `rclcpp`.
2. **Gazebo Sim (`gz-sim`)** plugins and ECS systems — designed around the canonical **Entity-Component-System** model used by Gazebo.

Both sides ship side by side in the configuration trees (`.claude/` and `.agents/`), prefix-disambiguated so they never collide:

| Concern | ROS 2 side | gz-sim side |
|---------|------------|-------------|
| Build / Test / Lint | `/build` `/test` `/lint` (colcon, ament) | `/gz-build` `/gz-test` `/gz-lint` (cmake, ninja, bazel) |
| Scaffold | `/new-package` `/new-node` `/new-nav2-plugin` | `/gz-new-component` `/gz-new-system` |
| Review agent | `ros2-style-reviewer` | `gz-style-reviewer` |
| Architecture agent | `clean-arch-architect` | `ecs-architect` |

The primary goal is to provide a comprehensive set of rules, templates, and guidelines that enable developers to build scalable robotic software with consistent patterns in both **Python** and **C++** — across the ROS 2 application layer **and** the Gazebo simulation layer.

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

The `rules/` directory (under `.claude/` or `.agents/`) contains detailed guidelines for:

- **Architecture**: Defining layer boundaries and dependency rules.
- **Node Development**: Patterns for creating robust and testable nodes.
- **Communication**: Standards for Topic naming, QoS profiles, and custom interfaces.
- **Testing**: Strategies for Unit (GTest/pytest), Integration, and Launch testing.

### 4. Developer Skills & Templates

A library of "Skills" (`skills/` under `.claude/` or `.agents/`) provides ready-to-use templates and explanations for:

- **Node Creation & Lifecycle Management**
- **Messaging Patterns (Pub/Sub, Services, Actions)**
- **Launch Configuration & Parameters**
- **TF2 Transforms & Diagnostics**
- **Bag Recording & Replay**
- **Nav 2 plugins** — controller / planner / behavior / smoother / costmap-layer / BT-node scaffolding

### 5. Slash Commands / Workflows

Executable workflow commands / custom slash commands (under `.claude/commands/` or `.agents/workflows/`):

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
| `/gz-build [extra cmake args]`         | cmake + ninja gz-sim build.                   |
| `/gz-test [ctest regex]`               | ctest filtered run for gz-sim.                |
| `/gz-lint [--all\|files]`              | pre-commit over changed-vs-main (gz-sim).     |
| `/gz-new-component <Name>`             | Scaffold an ECS component header.             |
| `/gz-new-system <name> [Class]`        | Scaffold a full gz-sim system plugin.         |
| `/gz-changelog [base]`                 | Generate a `Changelog.md` block (gz-sim style). |

### 6. Sub-agents

Specialized sub-agents (under `.claude/agents/` or `.agents/agents/`):

ROS 2 / Nav 2:
- **`ros2-style-reviewer`** — strict PR review against Clean Architecture, lifecycle, QoS, pluginlib, tests, build manifests. Returns a file:line punch list.
- **`clean-arch-architect`** — design advisor: where does this behaviour belong? node vs use case, topic vs service vs action, compose vs split.

gz-sim / Gazebo:
- **`gz-style-reviewer`** — strict PR review for gz-sim: ECS conventions, `GZ_ADD_PLUGIN`, CMake/Bazel parity, Migration.md / Changelog.md drift.
- **`ecs-architect`** — design advisor: which Component holds this state, which `PreUpdate / Update / PostUpdate` phase, how to avoid singleton coupling.

## Getting Started

1.  **Skim the configuration README** (`.claude/README.md` or `.agents/README.md`) for a complete index of what's available.
2.  **Review the Rules**: Check the `rules/` directory to understand the architectural standards.
3.  **Use the Skills**: Refer to `skills/` for implementation examples and templates.
4.  **Try the Slash Commands / Workflows**: `/build`, `/test`, `/new-package my_pkg python`.
5.  **Run Tests**: Use `colcon test` and `pytest` to verify your implementations.

## Adopting the template in a new workspace

To support both **Antigravity (AGY)** and **Claude Code** in your workspace:

```bash
cd ~/your_ws
# Copy the configuration directories
cp -r /path/to/ros2-agy-claude-code-template/.claude .
cp -r /path/to/ros2-agy-claude-code-template/.agents .

# Copy the root orientation files
cp /path/to/ros2-agy-claude-code-template/CLAUDE.md .
cp /path/to/ros2-agy-claude-code-template/AGENTS.md .

# Merge the gitignore settings
cp /path/to/ros2-agy-claude-code-template/.gitignore .gitignore.template  # merge with yours
```

Then open the workspace with your preferred agent tool (**agy** or **claude**) — it will pick up the rules, skills, subagents, and commands/workflows automatically.

## License

This project is open-source and available under the Apache 2.0 License.
