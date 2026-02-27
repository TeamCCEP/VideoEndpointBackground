# Cisco RoomOS Idle Video Background

This repository hosts a lightweight HTML solution designed to display a full-screen, looping video background on Cisco Webex Devices (RoomOS) when they are in an idle state.

## Overview

This solution utilizes the **Digital Signage** feature within Cisco RoomOS. When the device is not in a call and detects no active user interaction for a set period, it loads the hosted HTML file.

The HTML file contains an HTML5 video player configured to:
*   **Autoplay** immediately upon loading.
*   **Loop** continuously.
*   **Mute** audio (required for autoplay compliance on modern web engines).
*   Scale to **Full Screen** regardless of the display aspect ratio.

## Understanding RoomOS Power States

To successfully deploy video backgrounds, it is critical to understand the difference between the two "sleep" modes in RoomOS:

### 1. Halfwake (The Target State)
*   **Status:** The screen remains **ON**.
*   **Behavior:** The Digital Signage (this web page) is displayed.
*   **Trigger:** Occurs after the *Standby Delay* timer expires.

### 2. Standby (Deep Sleep)
*   **Status:** The screen turns **OFF** (black) to save power.
*   **Behavior:** The device kills the web engine and signage to enter deep sleep.
*   **Configuration:** For continuous video playback, **Standby Mode must be disabled** so the device remains in Halfwake indefinitely.

### Timings and Sensors
By default, the system waits **10 minutes** of inactivity before entering Halfwake.
*   **Note:** Cisco devices utilize ultrasound and camera-based motion detection. If a person is detected in the room, the timer resets, and the device will not enter Halfwake. The signage will typically only activate when the room is physically empty or the occupants are very still.  See the troubleshooting section for further info on this.

---

## Configuration Guide

You can configure these settings on a single device via the device's web interface, or via Cisco Control Hub.  Take a copy of the idle.htm file, and host it on a web server alongside your video.  Change the reference to the video file in idle.htm to point to your own video file link, then adjust configurations on control hub on the relevant endpoints.

### Required Configurations
To enable the video background, [apply the following configurations:](https://help.webex.com/en-us/article/n5pqqcm/Device-configurations-for-Board,-Desk,-and-Room-Series-devices)

| Configuration | Value | Description |
| :--- | :--- | :--- |
| `WebEngine Mode` | **On** | Enables the browser engine. |
| `Standby Signage Mode` | **On** | Tells the device to load a URL instead of the default black screen. |
| `Standby Signage Url` | *URL Where the website is hosted* | The direct link to the `idle.htm` file in this repo. |
| `Standby Control` | **Off** | **Crucial:** Prevents the device from entering deep sleep (black screen). |

### Optional Configurations
| Configuration | Value | Description |
| :--- | :--- | :--- |
| `Standby Delay` | **1 - 480** | Minutes of inactivity before the video starts (Default: 10). |
| `Video Output Connector [n] CEC Mode` | **Off** | Set this if your TV physically turns off despite the settings above. |

---

## Deployment via Control Hub (Templates)

For deploying this configuration to multiple devices (e.g., an entire office or campus), it is recommended to use **Configuration Templates** in Cisco Control Hub.

1.  Log in to **Control Hub**.
2.  Navigate to **Devices** > **Configurations**.
3.  Select **Templates** and create a new template.
4.  Search for the configurations listed in the table above and set the values.
5.  Apply the template to your specific devices or device groups.

**Documentation:**
For detailed steps on managing templates, please refer to the official Cisco documentation:
[Manage Device Configurations in Control Hub](https://help.webex.com/en-us/article/n5pqqcm/Device-configurations-for-Board,-Desk,-and-Room-Series-devices#Cisco_Reference.dita_32fda655-3543-41c1-a1c1-b9df163c7cf4)

---

## Troubleshooting

## Wake-up Behaviors & Interaction

By default, Cisco devices are designed to wake up (exit Halfwake/Signage mode) as soon as they detect motion or people in the room. You can customize this behavior based on the device's location (e.g., a meeting room vs. a hallway display).  Either adjust the parameters from the device configurations within control hub, or adjust them directly via SSH or the devices local controls.

### 1. The "Screensaver" Experience (Default)
*   **Behavior:** The video plays when the room is empty. As soon as someone walks in, the device wakes up and shows the standard Cisco interface, ready for a call.
*   **Best For:** Meeting rooms and conference spaces.
*   **Configuration:**
    ```
    xConfiguration Standby WakeupOnMotionDetection: On
    ```

### 2. The "Billboard" Experience (Persistent Video)
*   **Behavior:** The video continues playing even if people are walking around or sitting in the room. The device only wakes up if someone physically touches the Navigator panel or an incoming call occurs.
*   **Best For:** Lobbies, hallways, and digital signage displays.
*   **Configuration:**
    ```
    xConfiguration Standby WakeupOnMotionDetection: Off
    ```

### Advanced Sensor Control
The `WakeupOnMotionDetection` command relies on two underlying sensor technologies to detect people, and wake the endpoint up (Cancelling the video playback).

You can fine-tune these individually if necessary:

*   **Ultrasound (Proximity):** Detects presence via audio waves and pairs with Webex Apps.
    *   Command: `xConfiguration Proximity Mode: [On/Off]`
*   **People Presence (Camera):** Uses the camera to detect faces.
    *   Command: `xConfiguration RoomAnalytics PeoplePresenceDetector: [On/Off]`