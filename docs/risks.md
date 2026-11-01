# Risks

## Approvals

### Provided laptop instead of personal laptops

- **What:** A laptop provided for the project, with the ground station software preinstalled, so team members don't have to install ROS 2, Unity, and the UDP bridge on their personal devices. The club can't supply one because of budget, so we asked the course.
- **Asked by:** Tetsuya Bassler
- **Asked:** October 26, 2026, to our TA, who passed it to the instructor
- **Status:** Waiting on a response
- **If denied:** Luis and any other team members who need it install the software on their personal laptops.

### Equivalent deployment milestone

- **What:** That the instruction team accept Luis installing the release on his own laptop as our deployment milestone, since no device outside the team is available (see `docs/design.md`).
- **Asked by:** Tetsuya Bassler
- **Asked:** November 1, 2026, by email to our TA
- **Status:** Waiting on a response

### Permission to fork Rover-Unity

- **What:** Permission to fork and build on the club's Rover-Unity project, which has no license.
- **Granted by:** Jared Northrop (project partner) and Nolan Kessler (Rover-Unity maintainer)
- **Granted:** November 1, 2026
- **Status:** Approved

## Staging and rollback

**Staging.** Changes are exercised against the club's Gazebo simulator before they're used with the real rover. A change is tested in the simulator on Luis's laptop before a release goes out. Lands: December 5, 2026 (tentative, subject to change). Depends on getting the club's Gazebo simulator running on Luis's laptop.

**Rollback.** Every release stays on the GitHub Releases page, so if a new build breaks, the operator reinstalls the previous release. The bad change is then reverted on `main` with a pull request. Lands: November 21, 2026 (tentative, subject to change), with our first GitHub release. Depends on the scaffold and CI being in place.
