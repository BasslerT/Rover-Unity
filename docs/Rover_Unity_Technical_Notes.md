# Technical Notes: Unity Ground Station and ROS 2

**Project:** Mars Rover Ground Station Controls and Planning (Project 33)  
**Environment:** Ubuntu 24.04, ROS 2 Jazzy, Unity / Rover-Unity  
**Scope:** Sprint 2 investigation, troubleshooting, and next checks

> **Evidence status:** These notes distinguish observed results, local troubleshooting, and **suggested but uncommitted code changes**. They are not a claim that the Unity-to-ROS-to-rover communication path has been tested end to end.

## 1. Unity issues investigated

### UI slider / missing-reference errors

- Investigated errors related to `Assets/Control Interface App/arucoMarkerID.cs` and missing UI references (including `slider_t_1`).
- Identified **Slider T1** and **Slider T2** as relevant UI controls and discussed assigning their references through the Unity Inspector.
- During local troubleshooting, **unchecking the Sprite Transparency setting** eliminated the red Unity errors, leaving warnings. This was an observed workaround, **not proof that all missing slider references were permanently repaired or saved to version control**.
- Follow-up: verify the actual Inspector field assignments, save the affected Unity scene/prefab if appropriate, and reopen the project to confirm the error remains resolved.

### Other warnings / earlier errors

- Earlier Unity logs included Ros2ForUnity compatibility complaints about ROS 2 Jazzy, a `DisplayManager` index error, and an `ObjectTransparency` null reference. Some warnings (for example, compiler warnings) were not necessarily blockers.
- Do **not** infer that every subsystem works simply because Unity now runs with warnings only.

## 2. Specific code fixes discussed (proposed, not confirmed as applied)

These are suggestions from troubleshooting conversations, **not changes claimed as committed**.

### A. Guard against null incoming messages in `arucoMarkerID.cs`

An earlier error involved accessing a null `message` object, around the logic that reads ArUco marker data. Before reading a message's contents, check whether it exists:

```csharp
if (message != null && message.HasValues)
{
    // Existing logic that reads this message goes here.
}
```

Apply the guard at the relevant message-handling sites (previously discussed near lines 45, 49, and 53; line numbers can change). Also check `arucoText`, `towerText`, and `gripperText` for null before updating them where applicable. **Do not paste this blindly**: confirm the actual type of `message` and its API (`HasValues`) in the current code.

### B. Use a fully qualified ROS 2 message type for ArUco subscriptions

The RFC identified three ArUco subscriptions (`/infared/aruco_id`, `/tower/aruco_id`, `/gripper/aruco_id`) that appear to pass a short type name such as `"int32"`. The bridge's ROS 2 message lookup expects a fully qualified name such as:

```text
std_msgs/msg/Int32
```

Proposed fix: **where the subscriptions specify `"int32"`, use `"std_msgs/msg/Int32"` if that is indeed the topic's published type**. Verify first with:

```bash
ros2 topic info /infared/aruco_id
ros2 topic info /tower/aruco_id
ros2 topic info /gripper/aruco_id
```

Also verify whether `/infared/aruco_id` is intentional; an earlier comment used `/infrared/aruco_id`. This is a **possible typo**, not a confirmed correction.

### C. Jazzy compatibility in older Ros2ForUnity code

An earlier `ROS2ForUnity.cs` support check listed older distributions and did not include `jazzy`, causing a distro-support error. Adding `"jazzy"` to a hard-coded allowlist was a **discussed compatibility workaround**, not a validation that the underlying plugin actually supports Jazzy. The club's newer UDP bridge is the current integration path, so **do not change this old plugin solely to suppress its compatibility check** without confirming which integration the branch uses.

## 3. ROS 2 and bridge setup verified locally

- ROS 2 Jazzy was installed and sourced on Ubuntu 24.04.
- Built the `unity_udp_ros_bridge` package with `colcon build --symlink-install`.
- `ros2 node list` showed:

```text
/tcp_relay
/udp_bridge_pub
/udp_bridge_srv
/udp_bridge_sub
```

- `ros2 topic list` showed:

```text
/autonomous/auton_control_response
/autonomous/simple_position
/gps/fix
/imu/heading
/joint_states
/nodetopiclisten
/odrive_telem
/parameter_events
/rosout
```

These checks establish that the nodes and topics were visible in that environment. **They do not establish that Unity messages reached the rover or that telemetry appeared in the Unity UI.**

### Reproducible commands

```bash
source /opt/ros/jazzy/setup.bash
cd ~/Rover-Unity/unity_udp_ros_bridge
colcon build --symlink-install
source install/setup.bash
ros2 node list
ros2 topic list
```

Run the appropriate launch/bridge processes before listing their nodes. Do not expect node names to appear from a build alone.

## 4. Communication architecture findings

- The Unity front end communicates through a local UDP bridge, which converts Unity-side JSON messages to ROS 2 topics or service calls, and routes replies back.
- Bridge nodes observed include UDP publisher, subscriber, and service nodes, plus `/tcp_relay`.
- The bridge is configured for loopback (`127.0.0.1`) communication between Unity and the locally running bridge processes. This is distinct from ROS 2 communication onward to simulation or rover hardware.
- Port details from the team's communication RFC: Unity-to-ROS publishing uses port **65434**; receiving telemetry uses ports **65436** (subscription request) and **65435** (reply); service requests/replies use **65437**/**65438**.
- The number of nodes does **not** by itself mean there are three independent bridges that should be merged. Confirm their roles before considering consolidation.

## 5. Still unverified / next steps

1. Reopen the saved Unity scene and confirm UI errors remain resolved; identify whether any `.unity` or `.prefab` files changed.
2. Check the ArUco topic types and whether the null-message guards are necessary before changing code.
3. Run a minimal Unity-to-ROS publication test and a ROS-to-Unity telemetry test, recording pass/fail output.
4. Test a drive command and telemetry feedback with Gazebo before claiming the full loop works.
5. Save screenshots or command output for the team's Sprint 2 evidence and link any accepted GitHub PR that contains this documentation or actual fixes.

## 6. Sprint 2 summary

Investigated Unity UI errors and reduced red errors to warnings during local troubleshooting. Successfully built the Unity UDP bridge and confirmed ROS 2 nodes and topics were visible. Examined the existing communication architecture and documented potential fixes and remaining verification work. **Code fixes listed above are proposed unless separately confirmed in a commit or PR.**
