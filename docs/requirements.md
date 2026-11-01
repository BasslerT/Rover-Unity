# Requirements

## Constraints

These come from building on the club's Rover-Unity project and the rover itself (see `docs/audit.md`).

- **ROS 2 Jazzy.** The rover runs Jazzy, set by the rover team. Jazzy targets Ubuntu 24.04.
- **Unity 2022.3.32f1.** We use the same version as Rover-Unity. Changing it would be its own project.
- **Bridge runs on the same machine as Unity.** The club's UDP bridge only accepts connections from `127.0.0.1`.
- **Rover message definitions come from the club's rover repo.** The bridge needs the club's `Rover_2026_2027` repo cloned next to this one.
- **Keep `Packages/packages-lock.json`.** It pins the GitHub-sourced Unity packages (UniTask, NuGetForUnity, Newtonsoft JSON) to exact commits.
- **Runs on team members' personal laptops.** The club has no budget for a dedicated machine (see `docs/design.md`).
- **Reuse is by permission, not license.** Rover-Unity has no license. Jared Northrop and Nolan Kessler gave us permission to fork it on November 1, 2026.
