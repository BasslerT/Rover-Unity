## Rover Frontend Setup Guide

For detailed instructions on installing Unity and
launching the Rover frontend, see:

[Frontend Installation Guide](README_And_ProjectScope_Info.md)

# GSCAP: Mars Rover Ground Station Control and Planning

A Unity ground station for the DAM Robotics Club Mars rover. Operators use it to view telemetry, send commands, and plan missions. Built as an Oregon State University senior capstone.

This repo is our team's fork of the club's [Rover-Unity](https://github.com/OSURoboticsClub/Rover-Unity), made with permission from Jared Northrop and Nolan Kessler. The club's original setup notes are kept below in [Rover-Unity setup notes](#rover-unity-setup-notes).

> **Status:** Early development. We haven't changed the club's code yet.

## Getting started

You'll need Git, Unity Hub, and Unity 2022.3.32f1.

1. Download the project by running this in a terminal (the link is under the green **Code** button on the GitHub repo page):

   ```bash
   git clone https://github.com/BasslerT/Rover-Unity.git
   ```

   The project is about 490 MB, so this takes a while.

2. Open **Unity Hub**, click **Add**, and select the `Rover-Unity` folder.
3. Open the project (the first compile takes a while), then open the main scene: `Assets/Control Interface App/Application.unity`.
4. Press **Play** at the top of the Unity editor to run it.

This starts the Unity app only. Connecting it to the rover also needs ROS 2 and the club's bridge; see [Rover-Unity setup notes](#rover-unity-setup-notes). Those notes were written for ROS 2 Humble, and the rover now runs Jazzy.

## Decisions

Decided so far:

- The rover runs **ROS 2 Jazzy** (set by the rover team).
- We build on a fork of Rover-Unity, using its Unity version, 2022.3.32f1.

Still open:

- How Unity connects to ROS 2 (keep the club's UDP bridge or switch to something standard)

Major decisions are written up as RFCs in [`docs/rfcs/`](docs/rfcs/).

## Tests and CI

There are no tests or CI yet. Rover-Unity has no tests, and running Unity in GitHub Actions needs a Unity license that no one on the team has set up yet.

The first test and CI run happen once these steps are done (owner: Luis Pringle, tentative; target: November 14, 2026, tentative, subject to change):

1. Create a free Unity account for the team and activate a Personal license in Unity Hub. This creates a license file, `Unity_lic.ulf`.
2. Add three repository secrets in GitHub (Settings > Secrets and variables > Actions): `UNITY_LICENSE` (contents of the license file), `UNITY_EMAIL`, and `UNITY_PASSWORD`.
3. Add one Edit Mode test that fails if the project breaks, such as checking that the `Application` scene and the `UdpController` script load.
4. Add a GitHub Actions workflow using [GameCI](https://game.ci) that runs the tests on every pull request.
5. Open a pull request and confirm the run passes.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). Team discussion happens on Discord.

## License

Rover-Unity has no license. We use it with permission from the DAM Robotics Club.

## Rover-Unity setup notes

The rest of this file is the club's original Rover-Unity README, unchanged except that its headings are one level smaller.

## Rover-Unity

### Groundstation Setup

#### Getting Started with Groundstation Code

The groundstation code is divided into three main parts:

- **Unity controls and readouts**
- **Connection interface between Unity and ROS2**
- **Data handling** — structured CSV data sent over UDP ports that follows a ROS-like encode/decode format.

---
All setup has been verified working on a fresh installation of Ubuntu 22.04
#### Required Software

- **Unity (2022.3.32f1)**  
  [https://docs.unity3d.com/hub/manual/InstallHub.html](https://docs.unity3d.com/hub/manual/InstallHub.html)

- **ROS2 Humble**  
  [https://docs.ros.org/en/humble/index.html](https://docs.ros.org/en/humble/index.html)

- **GStreamer**  
  [https://gstreamer.freedesktop.org/documentation/installing/index.html?gi-language=c](https://gstreamer.freedesktop.org/documentation/installing/index.html?gi-language=c)

- **Colcon**  
  [https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html)

---

#### Unity

Fairly standard Unity setup.  
The main difference is the tie-in with ROS, which uses structured CSV data sent over UDP ports that emulate ROS message structures.

There are multiple Unity pages, each representing a different data stream:

- **Cameras**
- **Telemetry**
- **Autonomy**

> **Note:** This system is intended to be used by at least **two laptops** (for ease of use), though one can handle everything if needed.

---

### Install Guide

> **Note:**  
> There is both a **Unity** and **ROS2** aspect to this project.  
> Once ROS2 and Unity are installed, you must clone **this repository** and the **rover_2023_2024** repository to get the bridge between them working.

#### Shell Sourcing Reminder

All ROS2 sourcing must be done in the **same shell (terminal)** session.  
You’ll source ROS2 and run `setup.sh` scripts — make sure these steps happen in the same terminal.
Building only has to occur if you ever change any code,.

To make the process easier, consider adding the relevant sourcing commands to your `~/.bashrc`.

---

### Unity ROS2 Repository Setup
#### Clone this repository
git clone https://github.com/OSURoboticsClub/Rover-Unity.git

#### Navigate to the workspace
cd Rover-Unity/ros2_ws

#### Build the py_pubsub package
colcon build --packages-select py_pubsub

#### Source the local setup
. install/setup.bash

#### Install pip requirements (if necessary)
pip install -r requirements.txt (Not pruned yet, could hold eroneous python packages)


### Rover ROS2 Repository Setup (required for message definitions)

### Clone the rover repository
git clone https://github.com/OSURoboticsClub/Rover_2023_2024.git

### Navigate to the workspace
cd Rover_2023_2024/software

### Build the py_pubsub package
colcon build --packages-select rover2_control_interface rover2_status_interface

### Source the local setup
. install/setup.bash


### How to run
#### Unity
In Unity hub, add the whole Rover-Unity directory as a new project from your disk. Compilation takes a while.

Once completed, and the editor is open, double click the "Application" file in your bottom file explorer.

#### ROS Bridge
cd Rover-Unity/ros2_ws

ros2 launch py_pubsub local_net_launch.py
