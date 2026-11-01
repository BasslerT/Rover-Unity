# Inherited-Codebase Audit: Rover-Unity

We're building on the DAM Robotics Club's existing ground station, [Rover-Unity](https://github.com/OSURoboticsClub/Rover-Unity), by forking it. This repo, [BasslerT/Rover-Unity](https://github.com/BasslerT/Rover-Unity), is that fork. This audit is based on the `main` branch as of commit `3990287` (October 30, 2026).

## What runs and how, and what doesn't

Rover-Unity has three parts:

- **A Unity app** (Unity 2022.3.32f1). The main scene is `Assets/Control Interface App/Application.unity`, with pages for cameras, telemetry, autonomy, science, and task tools. There are also two test scenes for a PID controller and a pure pursuit path follower.
- **A UDP bridge** (`unity_udp_ros_bridge/`), a ROS 2 Python package. Unity sends and receives JSON over UDP, and the bridge turns that into ROS topics and services. It only listens on `127.0.0.1`, so it must run on the same machine as Unity.
- **Extra ROS 2 packages** in `ros2_ws/src/` (`py_pubsub`, `AutonControlNode`, `auton_control_interfaces`), plus a separate Python science UI.

`start_bridges.sh` launches both ROS sides, and expects the club's `Rover_2026_2027` repo to be cloned next to Rover-Unity for message definitions.

What we know works: the club used it at the CIRC competition in August 2026, with drive controls, camera feeds, GPS on the map, and the science page.

What's uncertain:

- On October 24 the club removed Ros2ForUnity and switched everything to the new UDP bridge (merged October 30). A commit from October 17 says the author couldn't run a full test of the switch yet.
- The setup instructions were written for ROS 2 Humble on Ubuntu 22.04, but the rover now runs Jazzy.
- Our team hasn't run it yet. `TODO: after Luis's first run, note what worked and what didn't.`

## Tests and CI

- No tests. Unity's test framework (1.1.33) is installed, but there are no test files in the project's own code. The only tests are inside third-party libraries.
- No CI. The `.github/` folder only has a bug report template.

## Dependencies and their age

- **Unity 2022.3.32f1**, an older long-term-support version. Moving to a newer Unity version is possible later but would be its own project.
- **Unity packages from GitHub** (UniTask, NuGetForUnity, Newtonsoft JSON) are pinned to exact commits in `Packages/packages-lock.json`, so installs are repeatable as long as that file is kept.
- **Bundled libraries** copied into `Assets/`: Minis 1.3.1 and RtMidi 2.1.3 (MIDI controller input) and LibVLCSharp 3.9.5 (video).
- **Python requirements** (`ros2_ws/requirements.txt`) pin versions that match Ubuntu 22.04 and ROS 2 Humble, such as `rclpy==3.3.16` and `numpy==1.21.5`. These will likely need updating for Jazzy on Ubuntu 24.04, since that ships a newer Python and Jazzy has its own `rclpy`. The file also notes it hasn't been cleaned up and may list packages that aren't needed.

## Open risks and known debt

- **No license.** The repo is public, but that doesn't give us permission to reuse it. We need the club's permission on record.
- **Docs are behind the code.** READMEs still say Humble, and the bridge README says it was written by Copilot and may contain errors.
- **Hardcoded paths.** The bridge README and `.vscode/settings.json` reference `/home/nolan/...`.
- **Fresh, partly tested bridge migration** (see above).
- **Large repo.** About 490 MB with no Git LFS, so cloning is slow.
- **Many branches.** Twelve branches, including `gazebo_sim`, which has the club's simulator setup we plan to use for staging.
- `TODO: open issues and pull requests on Rover-Unity (we couldn't check these, review them on GitHub)`

## Handoff document and retrospective

There isn't one. Rover-Unity isn't a previous capstone team's project; it's the club's live, actively maintained tool. Knowledge lives with the club: Nolan Kessler maintains the code, and Jared Northrop, our project partner, meets with our team weekly.

## Upstream contribution rules

None written down. There's no CONTRIBUTING file, only a bug report issue template. From the history, changes go through branches and pull requests that Nolan merges.

## What we'll change first

1. **Done:** Jared Northrop and Nolan Kessler gave us permission to fork Rover-Unity on November 1, 2026.
2. **Add a test and CI**, following the plan in `README.md`.
3. **Update setup docs for Jazzy** and get it running on Luis's laptop, recording what breaks.
4. **Update the Python requirements** for Jazzy and Ubuntu 24.04.
