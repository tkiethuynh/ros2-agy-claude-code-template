# Agent Configuration — ROS 2 / Nav 2 + Gazebo Sim Clean Architecture Template

This directory establishes the workspace configurations, rules, skills, sub-agents, and commands/workflows for both **Antigravity (AGY)** and **Claude Code** agents. 

By separating instructions and tooling from your core application logic, this template ensures consistent code quality, strict adherence to architectural boundaries, and reproducible development workflows.

> [!NOTE]
> Two parallel, prefix-disambiguated configurations operate side-by-side:
> * **ROS 2 / Nav 2 side** → `colcon`, `ament`, `pluginlib`, Clean Architecture layers, Python and C++.
> * **gz-sim / Gazebo side** → `cmake`, `ninja`, `bazel`, Entity-Component-System (ECS) pattern, plugin scaffolding.
>
> Command and workflow names are prefix-disambiguated: `/build` (colcon) ⟷ `/gz-build` (cmake); `/test` ⟷ `/gz-test`; `ros2-style-reviewer` ⟷ `gz-style-reviewer`; `clean-arch-architect` ⟷ `ecs-architect`.

---

## Directory Structure

```text
.claude/ (and mirrored via .agents/)
├── README.md            # This file (Unified overview)
├── CLAUDE.md            # Orientation context (referenced by agents)
├── settings.json        # Execution permissions, workspace hooks, and environments
├── commands/            # Custom commands / workflows (under .agents/workflows/)
│   ├── build.md             — /build           colcon build wrapper
│   ├── test.md              — /test            colcon test & summary generator
│   ├── lint.md              — /lint            ament linter & pre-commit hooks
│   ├── new-package.md       — /new-package     Scaffold a Clean Architecture ROS 2 package
│   ├── new-node.md          — /new-node        Scaffold standard/lifecycle/composable nodes
│   ├── new-launch.md        — /new-launch      Scaffold modular launch configurations
│   ├── new-nav2-plugin.md   — /new-nav2-plugin Scaffold Nav 2 plugin integrations
│   ├── changelog.md         — /changelog       Generate CHANGELOG.rst block from git history
│   ├── ros2.md              —                  ROS 2 cheat sheet & quick reference
│   ├── nav2.md              —                  Nav 2 cheat sheet & quick reference
│   ├── gz-build.md          — /gz-build        cmake & ninja Gazebo build wrapper
│   ├── gz-test.md           — /gz-test         ctest Gazebo integration test runner
│   ├── gz-lint.md           — /gz-lint         pre-commit hooks for Gazebo modules
│   ├── gz-new-component.md  — /gz-new-component Scaffold ECS component headers
│   ├── gz-new-system.md     — /gz-new-system   Scaffold ECS system plugin structures
│   └── gz-changelog.md      — /gz-changelog    Generate Gazebo Changelog.md block
├── skills/              # Reusable agent recipe manuals
│   ├── ros2_node_creation/        — Guidelines for Clean-arch compliant nodes
│   ├── ros2_lifecycle/            — Managed lifecycle nodes template
│   ├── ros2_messaging/            — Robust publisher & subscriber patterns
│   ├── ros2_service_action/       — Service and Action server wrappers
│   ├── ros2_launch_config/        — Modular launch recipes
│   ├── ros2_transforms/           — TF2 management avoiding leaky domain dependencies
│   ├── ros2_diagnostics/          — Health monitoring & diagnostic updaters
│   ├── ros2_bag/                  — Rosbag2 record and playback configurations
│   ├── ros2_testing/              — GTest, pytest, and launch_testing integration
│   ├── new_ros2_package/          — Base template for package creation
│   ├── new_nav2_plugin/           — Base template for Nav 2 plugins
│   ├── nav2_core_interfaces/      — Plugin base class definitions
│   ├── nav2_controllers/          — DWB, MPPI, RPP controller guides
│   ├── nav2_planners/             — NavFn, SMAC, Theta* planners
│   ├── nav2_behavior_tree/        — Nav 2 BT node implementations
│   ├── nav2_costmap/              — Costmap layer filters and guides
│   ├── nav2_localization/         — AMCL and Map Server setups
│   ├── nav2_servers/              — Lifecycle, Velocity, Collision monitor servers
│   ├── gz-build/                  — cmake, ninja, bazel simulation build guides
│   ├── gz-test/                   — Headless ctest, gtest, xvfb automation
│   ├── gz-ecs-overview/           — Entity-Component-System simulation loops
│   ├── new-component/             — Custom component header templates
│   └── new-system/                — Complete Gazebo system plugin layouts
├── rules/               # Architectural & developmental constraints
│   ├── clean_architecture.md      — Boundaries between domain, application, infra, and presentation
│   ├── ros2_general.md            — Workspace-wide ROS 2 style & project setup
│   ├── ros2_nodes.md              — Execution models, callbacks, parameters
│   ├── ros2_communication.md      — Topics, QoS profiles, custom messages
│   ├── testing.md                 — Coverage and integration test plans
│   ├── nav2_architecture.md       — Navigation 2 pipeline diagrams
│   ├── nav2_parameters.md         — Nav 2 tuning parameter configurations
│   ├── nav2_msgs_reference.md     — Nav 2 standard message schemas
│   └── robot_specific.md          — Project-specific overrides (override as needed)
└── agents/              # Sub-agent personas
    ├── ros2-style-reviewer.md     — Automated PR review bot for ROS 2 standards
    ├── clean-arch-architect.md    — Design consultant for layers and node structures
    ├── gz-style-reviewer.md       — Automated PR reviewer for Gazebo Sim conventions
    └── ecs-architect.md           — Design consultant for Gazebo Sim ECS data flows
```

---

## Interacting with the Configuration

### Custom Commands & Workflows
Execute directly from your agent's interactive terminal session:
```bash
# ROS 2 / Nav 2
/build                    # Build the entire workspace
/build my_pkg             # Build a specific package only
/test                     # Run colcon test and print a clean summary
/lint --all               # Run linters over the entire repository
/new-package my_pkg python # Scaffold a python package
/new-node my_pkg my_node lifecycle cpp # Scaffold a C++ lifecycle node
/new-nav2-plugin controller MyController # Scaffold a controller plugin
/changelog                # Generate CHANGELOG.rst block

# gz-sim / Gazebo
/gz-build                 # Build the Gazebo sim module using cmake & ninja
/gz-test                  # Run Gazebo simulation ctests
/gz-lint                  # Run Gazebo pre-commit linters
/gz-new-component Foo     # Scaffold an ECS Component
/gz-new-system bar BarSys # Scaffold an ECS System plugin
/gz-changelog             # Generate Gazebo Changelog.md block
```

### Agent Skills
Agents automatically search `.agents/skills/` or `.claude/skills/` when executing tasks. You can instruct the agent to use a specific manual:
> "Implement a lifecycle node following the guidelines in skills/ros2_lifecycle/SKILL.md."

### Specialized Sub-agents
Dispatch sub-agents to parallelize tasks using the agent execution model:
*   `clean-arch-architect`: Consults on where logic belongs, layer boundaries, and ROS dependencies.
*   `ros2-style-reviewer`: Performs rigorous review on your branch diffs prior to opening a PR.
*   `ecs-architect`: Consults on component data layouts, system update phases, and Gazebo thread safety.
*   `gz-style-reviewer`: Validates system plugin registration and Bazel/CMake consistency for Gazebo code.

---

## Core Development Philosophy

1.  **Strict Dependency Inversion**: The `domain/` layer is pure business logic and must never import `rclpy`, `rclcpp`, or any ROS/Gazebo message definitions.
2.  **Recipes Over Ad-hoc Coding**: Consult `skills/` and `rules/` before writing code to maintain unified codebase architecture.
3.  **Explicit Tooling**: Prefer explicit command wrappers over hidden automation hooks for transparent builds, tests, and linting.
4.  **Single Source of Truth**: Link related files rather than duplicating content across manuals.
5.  **User-specific Overrides**: Create `.claude/settings.local.json` or `.agents/settings.local.json` for personal environment keys and preferences (pre-configured in `.gitignore` to prevent committing secrets).
