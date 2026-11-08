# RFC: How Unity Connects to ROS 2

| Field | Value |
| --- | --- |
| Author | Tetsuya Bassler |
| Date | 2026-11-08 |
| Status | Proposed |
| Decision Date | (fill when decided) |
| Deciders | (who agreed to the outcome) |
| Superseded By | |

## Problem and Context

Our ground station is a Unity app, and the rover runs ROS 2 Jazzy. Unity can't talk to ROS 2 on its own, so something has to translate between them. Right now that's the club's UDP bridge (`unity_udp_ros_bridge/`). Unity sends JSON over UDP to ports on the same machine, and the bridge turns it into ROS 2 topics and service calls, then sends replies back the same way. Every drive command and every piece of telemetry goes through it. In our fork, 13 Unity scripts use it through `UdpController.cs`, including drive, arm control, cameras, and science.

The bridge is new. The club moved Rover-Unity off Ros2ForUnity in October 2026: scripts were converted on October 17, and one commit that day says the author couldn't run a full test yet. Ros2ForUnity was removed on October 24, and the change was merged on October 30 (pull request #31). The commits don't say why the club switched, so this RFC doesn't assume a reason. Reading the code also turns up a few limits. The bridge only accepts connections from `127.0.0.1`, so it has to run on the same laptop as Unity. Its README says Copilot wrote it and it may contain errors. And the setup docs still describe ROS 2 Humble.

We need to decide now because our walking skeleton runs straight through this connection: one drive command from the app reaches the rover or the Gazebo simulator, and one piece of telemetry comes back (see `docs/design.md`). Every UI feature we add after that also calls the bridge, and every one of those scripts builds its messages in the bridge's JSON format. If we do nothing, we keep building on a bridge nobody has fully tested, and we find out what's wrong when Luis first runs it or at a club test.

## Options

Two other Unity and ROS options exist but aren't serious candidates here, so they're noted and set aside:

- **Go back to Ros2ForUnity.** The club just removed it. Its README lists support for ROS 2 Galactic and Humble, not Jazzy, and says the maintainers can't support the general community.
- **Unity's ROS-TCP-Connector.** It works, but its listed ROS 2 support is Foxy and Galactic, and its ROS-side package was last pushed about 10 months ago. We'd be trading one bridge with unclear Jazzy support for another.

### Option A: Keep the club's UDP bridge, and prove it works

Use the bridge as it is, run a short communication test (see the Verification Plan), and fix what the test finds in small pull requests.

- **Benefits:** No migration. We stay on the same code the club uses, so our fixes help them and theirs help us. The Unity side is already written for it.
- **Costs:** We inherit a bridge with no tests and docs that may be wrong. We'd own debugging it, at least until the club picks our fixes up. It stays local-only unless someone changes the code.
- **Right choice when:** the club plans to stay on the bridge, and it passes the baseline test, or fails it in ways that are small to fix.

### Option B: Switch to rosbridge_suite

rosbridge_suite is the standard ROS package for this job. It's released for Jazzy (version 2.7.1) and talks JSON over WebSockets. We'd run its server in place of the UDP bridge, and change `UdpController.cs` to talk to it.

- **Benefits:** A widely used, maintained package with a released Jazzy version, so it isn't on us to maintain. It isn't limited to the same machine.
- **Costs:** Unity would need a WebSocket client, either a library like ROS# or our own code, and we haven't checked which works with Unity 2022.3. The message format is different, so `UdpController.cs` would have to translate, or all 13 scripts would change. And we'd split from the club's code, which they just finished moving to the UDP bridge.
- **Right choice when:** the UDP bridge fails the baseline test badly, the club wants to move to something standard, or the ground station has to run on a different machine than the bridge.

### Option C: Do nothing

Keep using the UDP bridge without testing it on purpose, and deal with problems as features hit them.

- **Costs:** It costs nothing today. The cost shows up at Luis's first run or at a club test, when a broken connection is mixed in with every other new thing and is hard to trace. Each feature we add on an untested bridge is one more thing that might need redoing.
- **Right choice when:** never for long. It only makes sense if the club is about to replace the bridge themselves and any work on it would be wasted.

## Analysis

**Both bridges work the same way.** The club's bridge looks up message types by name (`get_message` in `UDPBridgePub.py` and `UDPBridgeSub.py`), so it handles any message type, including the club's custom ones in `rover2_control_interface`. rosbridge works the same way. Switching wouldn't add a capability we're missing today. What it would add is a maintainer other than us.

**Switching costs less than it first looks, but it isn't free.** Unity scripts call the bridge only through `UdpController.inst`, using a few methods (`PublishMessage`, `ConfigureSubscription`, `GetLatestMessage`, `PublishClientReq`). That means a switch could mostly stay inside `UdpController.cs`. But the scripts build their own JSON in the bridge's format (`topic`, `msgType`, `data`), so `UdpController.cs` would have to convert that to rosbridge's format on the way out and back on the way in. That's doable, but it's new code we'd have to test, on a team where no one has the app running yet.

**There's at least one bug already, found by reading.** `arucoMarkerID.cs` subscribes to `/infared/aruco_id` (and two other topics) with the type `"int32"`. The bridge passes that straight to `get_message`, which expects a full name like `std_msgs/msg/Int32`. As far as we can tell, those subscriptions fail, and the bridge logs an error without stopping. We haven't confirmed this by running it. It's a small fix, but it shows the bridge's users haven't been fully tested together, which matches the October 17 commit.

**The local-only limit doesn't hurt us yet.** Our deployment plan already runs Unity and the bridge on the same laptop (`docs/design.md`), and the laptop reaches the rover through ROS 2 over Zenoh, not through the bridge. So that limit only matters if the plan changes.

**What we're unsure about:**

- Why the club switched away from Ros2ForUnity. If it was for a reason that also applies to rosbridge, Option B is weaker.
- Whether the bridge's 30 Hz per-topic limit on outgoing messages is fast enough for drive and arm control.
- Which Unity WebSocket client would work with Unity 2022.3, if we went with Option B.
- How well the bridge holds up under real use. No one on our team has run it.

## Recommendation and Verification Plan

**We recommend Option A:** keep the club's UDP bridge and prove it works with a baseline test. It's what the club runs, the Unity side is already built for it, and a switch would add new, untested code before we have anything running. If the test turns up problems too big to fix in a few small pull requests, we write a new RFC for Option B.

**Baseline test.** Each step adds one thing to the last, so a failure points at one piece. It runs on one laptop with ROS 2 Jazzy, the rover repo's message packages built, and the bridge launched (`ros2 launch unity_udp_ros_bridge udp_bridge_launch.py`).

| Step | What we do | Passes when |
| --- | --- | --- |
| 1. ROS 2 alone | `ros2 topic pub /chatter std_msgs/msg/String "{data: hello}"` in one terminal, `ros2 topic echo /chatter` in another | The echo shows `hello` |
| 2. Bridge, Unity to ROS | A small Python script sends `{"topic": "/chatter", "msgType": "std_msgs/msg/String", "data": {"data": "hello"}}` to `127.0.0.1:65434` | `ros2 topic echo /chatter` shows `hello` |
| 3. Bridge, ROS to Unity | The script sends `/chatter;std_msgs/msg/String` to port 65436 and listens on 65435, then we publish with `ros2 topic pub` | The script receives JSON with `"data": "hello"` |
| 4. Bridge, service call | Run the bridge's `setbool_server.py`, send a request to port 65437, listen on 65438 | The script receives the service's reply |
| 5. Unity drive command | Run the Unity app and move the drive control | `ros2 topic echo /cmd_vel` shows a `Twist` that changes with the stick |
| 6. Full loop in the simulator | Launch `rover_gazebo gazebo_unity.launch.py` and drive from Unity | The simulated rover moves, and one telemetry value (such as `joint_states`) shows up in the app |

Steps 2 to 4 don't need Unity, so they can be checked before the app runs. Steps 5 and 6 match the walking skeleton in `docs/design.md`.

**Who and when.** Tetsuya Bassler writes the test script for steps 2 to 4 and any fixes the test turns up, and pushes them to a branch. Luis Pringle runs them on his laptop, since it's the first one with the software installed, and reports what passed or failed. Steps 1 to 4 happen at Luis's first run. Steps 5 and 6 happen with the staging work in `docs/risks.md` (December 5, 2026, tentative).

**What would make us revisit this:**

- Any step fails, and the fix needs more than a small change to the bridge or `UdpController.cs`.
- Drive or arm commands feel laggy at the bridge's 30 Hz limit.
- The club decides to move off the bridge.
- Nolan's answer about why they left Ros2ForUnity changes the picture.

## Constraints

- **Licensing.** Rover-Unity, including the bridge, has no license. We use it because Jared Northrop and Nolan Kessler gave us permission on November 1, 2026 (`docs/risks.md`). Option B would add rosbridge_suite, which is BSD-licensed, so it's fine to use. If we go with Option A, any fixes to the bridge stay in our fork unless the club accepts them.
- **Data use.** None. The bridge carries rover commands and telemetry, not personal data.
- **Partner confidentiality.** None applies. Both the club's repos are public, and this RFC only cites what's in them.

## Delegation and Validation

- **What was handed to AI tools:** Claude (Anthropic) read both repos and drafted most of this RFC: the code findings, the option research, and the writing. It also checked rosbridge_suite, Ros2ForUnity, and ROS-TCP-Connector status from their public pages.
- **What checks made that reasonable:** Every claim cites a file, commit, or public page a reviewer can check. This is a document, not code, so nothing ships from it directly. It goes through our normal pull request review (1 approval, per `CONTRIBUTING.md`). There are no tests or CI in the repo yet.
- **Where validation happens:** The author hasn't separately re-checked the AI's findings. Instead, every finding is cited so a reviewer can check it in a minute or two, and the ones this recommendation leans on most are these:
  - The 13 scripts that use `UdpController.cs` (search the repo for `UdpController`)
  - The `"int32"` subscription in `arucoMarkerID.cs`, and that `get_message` rejects it
  - The commit dates and pull request #31 in the club's history
  - The rosbridge Jazzy release, Ros2ForUnity's supported distros, and its maintenance note

  The real check on the recommendation is the baseline test. If the bridge doesn't work the way this RFC expects, the test will show it before we build on it.

This decision is easy to reverse. Option A changes nothing yet, and moving to Option B later would mostly be contained in `UdpController.cs`.

## Decision

Proposed. Not yet decided.

## Sources

- Club rover repo: [OSURoboticsClub/Rover_2026_2027](https://github.com/OSURoboticsClub/Rover_2026_2027) (`joy_to_drive.py`, `gazebo_unity.launch.py`, `zenoh_config/`)
- Club ground station: [OSURoboticsClub/Rover-Unity](https://github.com/OSURoboticsClub/Rover-Unity) (commits `7278b5f`, `65d1f6d`, `3990287`; `unity_udp_ros_bridge/`; `UdpController.cs`; `arucoMarkerID.cs`)
- [rosbridge_suite on ROS Index](https://index.ros.org/r/rosbridge_suite/)
- [Ros2ForUnity README](https://github.com/RobotecAI/ros2-for-unity)
- [ROS-TCP-Connector](https://github.com/Unity-Technologies/ROS-TCP-Connector) and [ROS-TCP-Endpoint activity](https://repos.ecosyste.ms/hosts/GitHub/repositories/Unity-Technologies%2FROS-TCP-Endpoint)
