# 🏁 Kinetiq — Rider Telemetry System

Kinetiq is a high-performance, premium-designed rider telemetry application built with Flutter. It tracks, logs, analyzes, and visualizes ride data in real-time by fusing GPS data with raw device sensors (accelerometer and magnetometer). The application uses digital signal processing and kinematics modeling to calculate vehicle-aligned metrics, classify riding modes dynamically, and export print-ready analytical reports.

---

## ⚡ Core Features

- **📍 High-Precision GPS Tracking**: Real-time recording of GPS coordinates (latitude, longitude), speed (km/h), bearing (heading in degrees), and calculated accumulative distance.
- **🌀 Multi-Sensor Fusion & World-Alignment**: 
  - Fuses raw Accelerometer and Magnetometer data to build a dynamic device-to-world rotation matrix (East-North-Up coordinate system).
  - Isolates gravity via a Low-Pass Filter (LPF) and projects linear acceleration vectors onto the vehicle's body axes (Longitudinal, Lateral, and Vertical).
- **🧠 Intelligent Riding Mode Classification**:
  - Automatically identifies user locomotion type: **Walking**, **Scooter**, or **Motorbike**.
  - Calculates device tilt angle relative to the gravity vector.
  - Applies a rolling 5-second time window average and implements hysteresis logic to filter noise and prevent state flickering.
- **📊 Real-Time & Historic Visualizations**:
  - Real-time ride recorder showing longitudinal/lateral/vertical G-forces, speed, and heading.
  - Interactive charts utilizing `fl_chart` for speed profile, acceleration vectors, and deceleration spikes.
  - Map overlays with custom polyline routes and interactive playback slider on `flutter_map` (OpenStreetMap).
- **📁 Advanced Reporting & Exports**:
  - Export complete granular high-Hz session records to structured **CSV** format.
  - Generate premium, print-ready **PDF Reports** summarizing average acceleration/deceleration, metrics, and detailed data tables using `pdf`.

---

## 🏗️ Architecture & Codebase Structure

The codebase is organized following clean architecture patterns:

```
lib/
├── main.dart             # Application entry point, service initialization, and MaterialApp theme configuration
├── models/
│   ├── ride.dart         # Represents a complete recorded ride session
│   ├── sample.dart       # Individual sensor & GPS data point including raw and derived values
│   └── telemetry_model.dart  # Legacy telemetry data structure mapping
├── services/
│   ├── db_service.dart   # SQLite database implementation, table creation, indexes, schema upgrades
│   └── sensor_service.dart # Real-time sensor reading, low-pass filter gravity estimation, coordinate transformations
├── screens/
│   ├── home_screen.dart          # Launch screen with a custom red-glow glassmorphism design
│   ├── rides_list_screen.dart    # History list of recorded rides with active state tracking
│   ├── ride_record_screen.dart   # Session recorder with real-time HUD and controls
│   ├── ride_detail_screen.dart   # Dashboard summary, timestamp searching, and sensor data points listing
│   ├── overview_screen.dart      # Interactive route replay map and mode classification HUD
│   └── advanced_data_screen.dart # Interactive graphical charts for speed, deceleration, and 3D acceleration vectors
├── theme/
│   └── app_styles.dart   # Centralized theme tokens, premium dark mode palette, custom buttons
├── utils/
│   └── pdf_generator.dart # PDF builder layout, formatting tables, margins, and custom style generator
└── assets/
    └── fonts/            # Premium sci-fi & sports fonts (Orbitron, Exo 2)
```

---

## 🗄️ Database Schema

Kinetiq uses a local SQLite database (`sqflite`) containing two primary relational tables:

### 1. `rides` Table
Tracks summarized metadata for each recorded ride session.
- `id` (INTEGER, Primary Key Autoincrement)
- `start_time` (TEXT, ISO-8601 representation)
- `end_time` (TEXT, ISO-8601 representation, nullable)
- `distance_m` (REAL, default `0`)
- `avg_speed_kmh` (REAL, default `0`)
- `sample_count` (INTEGER, default `0`)

### 2. `samples` Table
Logs high-frequency, synchronized sensor telemetry points linked to a parent ride.
- `id` (INTEGER, Primary Key Autoincrement)
- `ride_id` (INTEGER, Foreign Key referencing `rides(id)` ON DELETE CASCADE)
- `timestamp` (TEXT, ISO-8601 representation)
- `ax_long` (REAL) - Vehicle longitudinal acceleration (forward/backward G-forces)
- `ay_lat` (REAL) - Vehicle lateral acceleration (cornering/yaw G-forces)
- `az_up` (REAL) - Vehicle vertical acceleration (bump/suspension G-forces)
- `ax`, `ay`, `az` (REAL) - Raw device-aligned linear accelerations
- `speed_kmh` (REAL) - GPS derived speed
- `lat`, `lon` (REAL) - GPS coordinates
- `bearing_deg` (REAL) - Vehicle heading direction
- `decel` (REAL) - Calculated deceleration magnitude (braking force)
- `tilt_deg` (REAL) - Calculated tilt angle relative to earth's gravitational vertical

---

## ⚙️ Mathematical & DSP Implementations

### Gravity Estimation via LPF
To calculate true linear acceleration, gravity must be separated from raw accelerometer readings. A low-pass filter (LPF) estimates gravity:

$$\vec{g}_{smoothed} = \alpha \cdot \vec{g}_{prev} + (1 - \alpha) \cdot \vec{a}_{raw}$$

*(Where $\alpha = 0.8$ dynamically balances sensor latency and gravity stabilization).*

### East-North-Up Rotation
Device coordinates are translated to world-aligned coordinates using a rotation matrix constructed from the normalized gravity vector $\vec{g}_{norm}$ and magnetometer vector $\vec{m}_{norm}$:

$$\vec{east} = \vec{m}_{norm} \times \vec{g}_{norm}$$
$$\vec{north} = \vec{g}_{norm} \times \vec{east}$$
$$R = \begin{bmatrix} \vec{east} & \vec{north} & -\vec{g}_{norm} \end{bmatrix}$$

This matrix projects the device-frame acceleration vector into the world-frame. By combining this with the GPS bearing (heading $\theta$), we calculate vehicle-centric **longitudinal** (forward/backward) and **lateral** (left/right leaning/sliding) acceleration vectors.

---

## 🚀 Getting Started

### Prerequisites

- Flutter SDK: `^3.9.2` or later
- Dart SDK: Compatible with Flutter SDK
- Android SDK / iOS Xcode for compiling to mobile devices

### Installation

1. **Clone the Repository**:
   ```bash
   git clone git@github.com:lakshya-noir/Ignition-Rider-Telemetry-App.git
   cd Ignition-Rider-Telemetry-App/rider_telemetry_app
   ```

2. **Install Dependencies**:
   ```bash
   flutter pub get
   ```

3. **Configure Permissions**:
   The app requires location and sensor access. The following permissions must be enabled:
   - **Android**: Location (Fine & Coarse), Sensors.
   - **iOS**: CoreMotion (Accelerometer/Gyroscope) and CoreLocation (`NSLocationWhenInUseUsageDescription`).

4. **Run the Application**:
   ```bash
   flutter run
   ```

---

