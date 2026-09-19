# ci_workflow

Shared, reusable GitHub Actions CI workflows for firmware and ROS 2 projects.
Consumer repositories call these workflows instead of duplicating CI logic.

Repository: `git@github.com:fofolevrai/ci_workflow.git`

## Available reusable workflows

| Workflow file | Purpose | Key inputs |
| --- | --- | --- |
| [.github/workflows/reusable-firmware.yml](.github/workflows/reusable-firmware.yml) | Builds firmware (Debug/Release) with CMake + arm-none-eabi toolchain and uploads artifacts | `project-path` (default `color_sensing_poc`) |
| [.github/workflows/reusable-ros.yml](.github/workflows/reusable-ros.yml) | Builds a ROS 2 Docker image and runs `colcon build`/`colcon test` in a container | `project-path` (default `ros_bridge`), `ros-distro` (default `lyrical`), `require-docker-rebuild` (default `false`) |

## How to call this CI from another repository

1. In the consumer repository, add a workflow file, e.g. `.github/workflows/ci.yml`.
2. Reference the reusable workflows from this repo with `uses: fofolevrai/ci_workflow/.github/workflows/<file>@<ref>`,
   where `<ref>` is a branch, tag, or commit SHA of `ci_workflow` (e.g. `main`).
3. Pass any inputs needed via `with:`.

Example `ci.yml` to add in the consumer repo:

```yaml
name: ci

on:
  push:
    branches:
      - "**"
  pull_request:
    branches:
      - master

permissions:
  contents: read

jobs:
  firmware-build:
    name: firmware-build
    uses: fofolevrai/ci_workflow/.github/workflows/reusable-firmware.yml@master
    with:
      project-path: color_sensing_poc

  ros-colcon-test:
    name: ros-colcon-test
    uses: fofolevrai/ci_workflow/.github/workflows/reusable-ros.yml@master
    with:
      project-path: ros_bridge
      ros-distro: lyrical
      require-docker-rebuild: ${{ github.event_name == 'pull_request' && github.base_ref == 'master' }}
```

Notes:
- Pin `@master` to a tagged release (e.g. `@v1.0.0`) or commit SHA for reproducible CI once this repo has releases.
- The consumer repository must have the checked-out paths (`project-path`) matching its own folder layout.
- `require-docker-rebuild: true` forces a fresh Docker image build as a required gate (typically for PRs into the default branch).
