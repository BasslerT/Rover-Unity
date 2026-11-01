# Contributing Guide

How to make a change to this repository that meets our Definition of Done.

## Development Setup

Follow the [README](README.md). Use Unity 2022.3.32f1. Never commit secrets or keys.

## Local Checks

Run all tests in Unity's Test Runner (**Window > General > Test Runner**). A linter and automatic CI tests are not planned yet.

## Branches and Commits

- Default branch is `main`. Don't commit to it directly.
- Branch names: `feature/short-description` (example: `feature/telemetry-panel`)
- Commits: short and start with a verb (example: `Add telemetry panel`)

## Pull Requests and Review

Describe what changed and why. Any teammate can review. Needs 1 approval, reviewed within 2 days, then squash and merge.

## Definition of Done

- Tests pass in the Test Runner
- 1 approval (branch protection on `main`)
- Docs updated if needed (checked in review)

## Known Pitfalls

These come from the Rover-Unity code we build on (see `docs/audit.md`).

- **Setup docs say Humble.** Rover-Unity's READMEs were written for ROS 2 Humble on Ubuntu 22.04. The rover now runs Jazzy, so follow Jazzy steps and fix the docs as you go.
- **Python requirements are out of date.** `ros2_ws/requirements.txt` pins Humble-era versions that may not install on Ubuntu 24.04.
- **Don't hardcode paths.** Some files point at `/home/nolan/...`. Use relative paths instead.
- **The bridge is local only.** It listens on `127.0.0.1`, so Unity and the bridge must run on the same machine.
- **Don't delete `Packages/packages-lock.json`.** It keeps everyone on the same package versions.
- **Cloning is slow.** The project is about 490 MB.
- **Check where your pull request is going.** This repo is a fork, so GitHub often sets the target to the club's `OSURoboticsClub/Rover-Unity`. Before clicking Create, make sure "base repository" says `BasslerT/Rover-Unity`.
- **Getting the club's updates.** Our fork doesn't update on its own. Use the **Sync fork** button on the repo page to pull in the club's changes.

## Where Other Work Lives

Tasks go in GitHub Issues. Design decisions go in `docs/rfcs/`.

## Getting Help

Ask in the team Discord. Expect a reply within a day.
