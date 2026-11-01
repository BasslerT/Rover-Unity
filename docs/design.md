# Design

## Decisions

**Keep Rover-Unity.** We build on a fork of the club's Rover-Unity project (this repo) instead of starting a new one. It's the club's working, competition-tested ground station, and starting over would mean a second ground station competing with theirs. Jared Northrop and Nolan Kessler gave us permission to fork it on November 1, 2026.

**Keep the club's UDP bridge for now.** The club just finished moving Rover-Unity onto it in October 2026, so we use it as is. Whether to replace it with a standard option like rosbridge is still open and would need an RFC in `docs/rfcs/`.

## Deployment

**Production target.** The ground station runs on team members' personal laptops, with the required software (the Unity build, ROS 2, and the club's UDP bridge) installed on each. Luis Pringle's laptop is the first one it will run on. Jared Northrop, our project partner from the DAM Robotics Club, set this target because the club doesn't have budget for a dedicated laptop; we've worked with him in weekly Saturday meetings (plus occasional Tuesdays) through October 2026. Each team member owns and maintains their own laptop. Because we'd rather not install this software on personal devices, we've separately asked our TA, who passed the request to the instructor, about an alternative such as a provided laptop (see `docs/risks.md`). Our walking skeleton is the thinnest slice that works end to end: a Unity build of the ground station, downloaded from this repo's GitHub Releases page and installed on the laptop, with the club's UDP bridge running on the same machine (the bridge only accepts connections from the same machine). Pressing a drive control in the app sends one command that reaches the rover, or the club's Gazebo simulator when the rover isn't available, and one piece of rover telemetry shows up in the app. "Deployed" means Luis installs the release on his laptop by following only `README.md` and sees this round trip work. Since the course defines deployment as installing on a device outside the team, and the club has no device to offer, we have asked the instruction team to accept this as an equivalent milestone (see `docs/risks.md`).
