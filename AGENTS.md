# AGENTS.md

Instructions for AI coding tools working in this repository.

## Project

A Unity ground station for controlling and planning missions for the DAM Robotics Club Mars rover. Written in C# using Unity 2022.3.32f1.

## Rules

- Follow [CONTRIBUTING.md](CONTRIBUTING.md) for branches, commits, and pull requests.
- Never commit to `main` directly.
- Never commit secrets, keys, or license files.
- Don't edit anything in `Library/`, `Temp/`, or `Logs/`. Unity generates these.
- When adding, moving, or renaming a file in `Assets/`, keep its `.meta` file with it.
- Keep changes small and focused on one thing.

## Testing

Run tests in Unity's Test Runner (**Window > General > Test Runner**) before opening a pull request.

## Design Decisions

Larger changes need an RFC in `docs/rfcs/` before work starts.
