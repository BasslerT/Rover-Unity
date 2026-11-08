# GSCAP: Mars Rover Ground Station Control and Planning

A Unity ground station for the DAM Robotics Club Mars rover. Operators use it to view telemetry, send commands, and plan missions. This repository is our team's fork of the club's [Rover-Unity](https://github.com/OSURoboticsClub/Rover-Unity), made with permission from Jared Northrop and Nolan Kessler. The project is being developed as an Oregon State University senior capstone.

> **Status:** Early development. The front end can be opened and viewed; the Unity-to-ROS 2 bridges and physical Rover communication have not been verified as a complete system.

## Quick start: see the Unity front end

The quickest way to view the ground station is to clone the Unity project, open it in Unity Editor **2022.3.32f1**, open the `Application` scene, and press **Play**. You do not need ROS 2 or Rover hardware just to see the interface.

The walkthrough below was verified on **native Ubuntu 24.04.5 LTS** with ROS 2 Jazzy installed. The Windows 11 + WSL2 instructions are a planning reference and have not been verified end to end.

### What you need to view the interface

| Component | Required to view the UI? | Purpose |
| --- | --- | --- |
| Git | Yes | Clone the repositories |
| Unity Hub | Yes | Install and manage Unity Editor |
| Unity Editor **2022.3.32f1** | Yes | Open and run the ground station |
| Ubuntu 24.04 | Yes for the verified route | Development environment used for verification |
| ROS 2 Jazzy | No | Needed for backend and bridge development |
| `colcon`, `rosdep` | No | Needed to build ROS 2 packages |
| VS Code | No | Optional code editor |
| GStreamer, Gazebo, MoveIt 2, Nav2, RealSense | No | Optional Rover subsystems; not needed for a UI preview |

### Native Ubuntu 24.04 setup (verified UI path)

#### 1. Install Git and basic tools

Open Terminal (`Ctrl+Alt+T`):

```bash
sudo apt update
sudo apt install -y git curl gpg
```

#### 2. Clone the repositories

The Unity ground station and the Rover ROS 2 software live in separate repositories. For team development, clone this team's fork of Rover-Unity. If you are following the verified club-source setup, use the club repository URL shown below instead.

```bash
cd ~
# Team fork:
git clone https://github.com/BasslerT/Rover-Unity.git
# Rover ROS 2 packages and interfaces:
git clone https://github.com/OSURoboticsClub/Rover_2026_2027.git
```

If you are reproducing the original UI verification against the club's Unity source, use this instead of the team-fork clone:

```bash
git clone https://github.com/OSURoboticsClub/Rover-Unity.git
```

If you already cloned the repositories, do not clone them again. Check which folders exist:

```bash
ls -d ~/Rover-Unity ~/Rover_2026_2027
```

- `~/Rover-Unity` contains the Unity ground station and Unity-related bridge code.
- `~/Rover_2026_2027/software` contains the current Rover ROS 2 packages, interfaces, and hardware-side code.

> **Verification note:** The interface walkthrough was tested with the club's `Rover-Unity` checkout. The team fork is based on that project, but any fork-specific changes should be checked against the same steps.

#### 3. Install Unity Hub

Follow the [official Unity Hub Linux installation guide](https://docs.unity3d.com/hub/manual/InstallHub.html). On Ubuntu, the Unity package repository can be configured with:

```bash
curl -fsSL https://hub.unity3d.com/linux/keys/public \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/unityhub.gpg > /dev/null

echo 'deb [signed-by=/usr/share/keyrings/unityhub.gpg] https://hub.unity3d.com/linux/repos/deb stable main' \
  | sudo tee /etc/apt/sources.list.d/unityhub.list

sudo apt update
sudo apt install unityhub
```

Launch Unity Hub from the applications menu or run:

```bash
unityhub
```

Sign in with a Unity account. A browser may open to finish authentication.

#### 4. Install the required Unity Editor version

In Unity Hub:

1. Select **Installs** > **Install Editor**.
2. Find **2022.3.32f1**. If it is not listed, use the [Unity 2022.3.32 release page](https://unity.com/releases/editor/whats-new/2022.3.32) or Unity archive.
3. Install the base Editor. Optional Android, iOS, Mac, and Linux IL2CPP build-support modules were not needed to preview the UI on the verified Ubuntu machine.
4. Wait for the installation to finish.

> **Security note:** Unity Hub may show a security alert for this older Editor. It is the version this project currently uses. Treat any Editor upgrade as a separate change and test the project before adopting it.

#### 5. Open the Unity project

1. In Unity Hub, open **Projects**.
2. Choose **Add project from disk**.
3. Select `~/Rover-Unity` (the repository root containing `Assets`, `Packages`, and `ProjectSettings`).
4. Open it with Unity **2022.3.32f1**.
5. Wait for the first asset import and C# compilation to finish.

Do not select `Rover_2026_2027` as the Unity project. It is the separate ROS 2 repository.

#### 6. Open and view the ground station

In the Unity Editor:

1. Find the **Project** panel, usually at the bottom.
2. Open `Assets/Control Interface App/Application.unity` by double-clicking **Application**.
3. Click the **Game** tab beside **Scene**.
4. Click the **Play** triangle at the top of the Editor.
5. Check the six interface tabs: **Driver**, **Arm Operator**, **Task Tools**, **Science**, **Autonomy**, and **System Health**.

**Expected result:** The ground station interface appears. Camera panes may show `NO SIGNAL`; telemetry or system status may show `NOT CONNECTED`. That is expected for a UI-only preview without bridges or hardware.

To enlarge the Game view, right-click the **Game** tab and choose **Maximize** (or use the panel menu). Toggle the same option to restore the layout. Click **Play** again to stop before editing the scene.

## Optional: ROS 2 Jazzy backend workspace

> This section is not required to view the Unity interface. The workspace build was verified on native Ubuntu, but that does not mean all bridges or Rover integrations are working.

### Install ROS 2 Jazzy

Follow the [official ROS 2 Jazzy Ubuntu installation instructions](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html) for Ubuntu 24.04. Install the desktop distribution and build tools, then verify the environment in a terminal:

```bash
source /opt/ros/jazzy/setup.bash
echo "$ROS_DISTRO"
# Expected: jazzy
```

Install build helpers if needed:

```bash
sudo apt install -y python3-colcon-common-extensions python3-rosdep
```

If `rosdep` has not been initialized on this machine, follow its initialization instructions, then run:

```bash
rosdep update
```

### Install dependencies and build the Rover workspace

```bash
source /opt/ros/jazzy/setup.bash
cd ~/Rover_2026_2027/software

# Rover packages are in ros_packages, not src.
rosdep install --from-paths ros_packages --ignore-src -r -y --rosdistro jazzy

colcon build --symlink-install
source install/setup.bash

ros2 pkg list | grep -E 'rover|comms_control'
```

**Observed on the verified machine:** The main workspace built 30 packages, with some warnings. `rosdep` reported unresolved keys including `odrive`, `pcl`, `warehouse_ros_mongo`, `harvest_interfaces`, `matplotlib`, and `eigen3`. Investigate those by package; do not assume every warning is fatal to the whole build.

GStreamer, MoveIt 2, Nav2, Gazebo, Intel RealSense, and Zenoh are used by different Rover subsystems. Install them as needed for the subsystem you are working on; they are not prerequisites for viewing Unity.

## Windows 11 + WSL2 setup (not yet verified)

> This is a proposed teammate setup path, not an end-to-end verified route. Install Unity on Windows and use Ubuntu 24.04 inside WSL2 for ROS 2 development. The Unity UI can be viewed without starting ROS 2.

### 1. Install WSL2 and Ubuntu 24.04

Open **PowerShell as Administrator**:

```powershell
wsl --install -d Ubuntu-24.04
```

Restart Windows if prompted. Open Ubuntu from the Start menu and create a Linux username and password. Verify from PowerShell:

```powershell
wsl --list --verbose
```

`Ubuntu-24.04` should appear with **VERSION 2**. If WSL2 is already installed, do not overwrite an existing distribution.

### 2. Clone Rover-Unity on Windows

Install [Git for Windows](https://git-scm.com/downloads/win) and [Unity Hub](https://unity.com/download). In PowerShell:

```powershell
cd $HOME
git clone https://github.com/BasslerT/Rover-Unity.git
```

To reproduce the club-source verification instead, clone `https://github.com/OSURoboticsClub/Rover-Unity.git`. The Unity project will be in a Windows path such as `C:\Users\<WindowsUser>\Rover-Unity`.

### 3. Install Unity Editor and view the front end

1. Sign in to Unity Hub on **Windows**.
2. Go to **Installs → Install Editor** and choose **2022.3.32f1**. Use the [Unity archive](https://unity.com/releases/editor/archive) if needed.
3. The base Editor is enough for an initial UI preview.
4. Under **Projects → Add project from disk**, choose the Windows `Rover-Unity` folder containing `Assets`, `Packages`, and `ProjectSettings`.
5. Open the project and wait for asset import.
6. In Unity's **Project** panel, double-click `Assets/Control Interface App/Application.unity`.
7. Open the **Game** tab and click **Play**.
8. Look for the six pages: **Driver, Arm Operator, Task Tools, Science, Autonomy, System Health**.

Stop here if you only need to preview the Unity interface. Camera feeds may show `NO SIGNAL` and status panels may show `NOT CONNECTED` because bridges and hardware have not been started.

### 4. Optional: set up ROS 2 Jazzy inside WSL2

Open the **Ubuntu 24.04 terminal**, not PowerShell:

```bash
sudo apt update
sudo apt install -y git curl gpg python3-colcon-common-extensions python3-rosdep
```

Install ROS 2 Jazzy for Ubuntu 24.04 using the [official Debian-package instructions](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html). Then verify:

```bash
source /opt/ros/jazzy/setup.bash
echo "$ROS_DISTRO"
# Expected: jazzy
```

### 5. Clone and build the Rover ROS 2 workspace inside WSL2

```bash
cd ~
git clone https://github.com/OSURoboticsClub/Rover_2026_2027.git
source /opt/ros/jazzy/setup.bash
cd ~/Rover_2026_2027/software

# If rosdep is not initialized, follow its initialization prompt first.
rosdep update
rosdep install --from-paths ros_packages --ignore-src -r -y --rosdistro jazzy
colcon build --symlink-install
source install/setup.bash
```

The native Ubuntu workspace build completed 30 packages, with several unresolved `rosdep` keys. The WSL2 build itself has not been verified.

For future bridge work, keep a separate Linux checkout of Rover-Unity inside WSL2:

```bash
cd ~
git clone https://github.com/BasslerT/Rover-Unity.git
```

The Windows checkout is for the Unity Editor and the WSL2 checkout is for Linux ROS 2 bridge builds. The `ros2_ws`, `unity_udp_ros_bridge`, and `start_bridges.sh` components have not yet been successfully built and launched together in this setup.

**Networking caution:** Unity on Windows and ROS 2 inside WSL2 are in different environments. UDP/TCP ports, bind addresses, Windows Firewall, and WSL networking mode may need configuration. Do not assume the existing bridges will communicate across Windows and WSL2 without testing.

## Troubleshooting

### Unity Hub stays on “Signing in…”

On the verified Ubuntu setup, Unity Hub initially stayed on the sign-in spinner. Restarting Unity Hub and retrying browser authentication eventually opened the dashboard, though the exact action that fixed it was not isolated. If the browser asks to open Unity Hub, allow it.

### Unity opens an empty `Untitled` scene

The correct project may be open, but the ground station scene is not. Open `Assets/Control Interface App/Application.unity` from the Project panel.

### Game view is small or the keyboard shortcut does not work

Right-click the **Game** tab and choose **Maximize**. `Shift+Space` did not work reliably in the tested desktop session.

### Camera panes show `NO SIGNAL` or telemetry shows `NOT CONNECTED`

Those messages were observed during the UI-only preview. The bridges, video streams, and Rover hardware were not connected. These messages alone do not mean the interface failed to load.

### Unity Console errors

During UI exploration, `NullReferenceException` and `ArgumentException: method return type is incompatible` errors appeared. These remain unresolved and may affect individual features. Capture the full Console stack trace before changing scripts.

### ROS 2 package name contains `ros--...`

`$ROS_DISTRO` may be empty. In the same terminal, run:

```bash
source /opt/ros/jazzy/setup.bash
echo "$ROS_DISTRO"
```

For example, the explicit Gazebo package name is `ros-jazzy-ros-gz`.

### `rosdep` says `src` does not exist

The current Rover repository stores packages under `software/ros_packages`. Run `rosdep` with `--from-paths ros_packages` from `~/Rover_2026_2027/software`.

## Project notes

### Ground station architecture

The club's original setup describes three main pieces:

- Unity controls and readouts.
- A connection interface between Unity and ROS 2.
- Data handling for structured CSV data sent over UDP ports in a ROS-like encode/decode format.

The Unity interface has pages for **Cameras**, **Telemetry**, and **Autonomy** in the original notes. The current UI walkthrough exposes six tabs listed above. The original design was intended for at least two laptops for ease of use, though one laptop could run the components.

### Repository and version notes

The club's older README was written for Ubuntu 22.04, ROS 2 Humble, and the `Rover_2023_2024` repository. The current Rover team uses Ubuntu 24.04 and ROS 2 Jazzy with `Rover_2026_2027`. Compatibility between the older Unity bridge and Jazzy is not yet verified.

The older guide also contains these ROS 2 workspace steps:

```bash
# From the Rover-Unity checkout
cd Rover-Unity/ros2_ws
colcon build --packages-select py_pubsub
. install/setup.bash

# The older club guide used Rover_2023_2024 for message definitions
cd Rover_2023_2024/software
colcon build --packages-select rover2_control_interface rover2_status_interface
. install/setup.bash
```

These commands are preserved as historical reference only. They are not the verified Jazzy setup path above, and the bridge's compatibility with the current Rover has not been established. The old guide also mentioned running `ros2 launch py_pubsub local_net_launch.py`; that launch path has not been verified with the current setup.

### Decisions

Decided so far:

- The Rover team uses **ROS 2 Jazzy**.
- The Unity ground station uses Unity **2022.3.32f1**.
- The team is developing from a fork of Rover-Unity.

Still open:

- How Unity will connect to ROS 2: keep the club's UDP bridge, adapt it, or use another approach.

Major decisions are documented as RFCs in [`docs/rfcs/`](docs/rfcs/).

### Tests and CI

There are no tests or CI yet. Rover-Unity has no tests, and running Unity in GitHub Actions requires a Unity license that the team has not set up yet.

The tentative first test and CI run target is **November 14, 2026**, subject to change. The plan is to:

1. Create a free Unity account for the team and activate a Personal license in Unity Hub. This creates a `Unity_lic.ulf` license file.
2. Add GitHub repository secrets under **Settings > Secrets and variables > Actions**: `UNITY_LICENSE` (license file contents), `UNITY_EMAIL`, and `UNITY_PASSWORD`.
3. Add an Edit Mode test that checks the project loads key assets, such as the `Application` scene and `UdpController` script.
4. Add a GitHub Actions workflow using [GameCI](https://game.ci) to run tests on every pull request.
5. Open a pull request and confirm the run passes.

### Contributing

See [`CONTRIBUTING.md`](CONTRIBUTING.md). Team discussion happens on Discord.

### License

Rover-Unity has no license. We use it with permission from the DAM Robotics Club.

## Setup verification checklist

- [ ] `Rover-Unity` repository cloned (team fork or club source identified)
- [ ] Unity Hub opens and signs in
- [ ] Unity Editor **2022.3.32f1** installed
- [ ] `Rover-Unity` project opens in Unity Editor
- [ ] `Application.unity` scene loads
- [ ] Game view shows the six ground station tabs in Play mode
- [ ] (Optional) ROS 2 Jazzy is sourced and `ROS_DISTRO=jazzy`
- [ ] (Optional) Main Rover ROS 2 workspace builds
- [ ] (Future validation) Unity and ROS 2 exchange actual messages

## Out of scope / future documentation

- Build and launch the Unity UDP and other ROS 2 bridges together
- Verify actual command and telemetry flow
- Repair missing or broken Unity controls
- Set up simulated ROS 2 responses or a Gazebo Rover model
- Connect to physical Rover hardware and Zenoh networking
- Validate Windows/WSL2 Unity-to-ROS 2 connectivity

Update this README when a teammate verifies a step on another machine. Keep untested instructions clearly marked.
