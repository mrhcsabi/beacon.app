# Beacon (.NET 10)

Windows system tray application built with **.NET 10 + WPF + Blazor Hybrid + Fluent UI + TailwindCSS**.  
The app connects local machine capabilities to **Home Assistant** through MQTT and Home Assistant REST APIs.

## Scope (Current Iteration)

This iteration includes all core functionality **except native AURA hardware backend**.

## Key Features

- **Tray-first desktop app**
  - Runs in notification area
  - Right-click menu: `Settings`, `Exit`
  - Double-click opens/activates Settings
- **Single-instance enforcement**
  - Only one instance runs
  - Secondary launch signals primary instance and exits
- **Settings UI (Blazor Hybrid)**
  - `Connectivity`, `System`, `Entities`, `HA Lighting` tabs
- **MQTT integration**
  - Persistent background connection with reconnect
  - Status visible in tray tooltip and UI
  - MQTT test connection from Settings
- **Home Assistant MQTT Discovery**
  - Button entities: `lock`, `shutdown`, `reset`
  - Sensor entities: `machine_status`, CPU/GPU usage/temp, disk usage
- **System controls**
  - Lock workstation
  - Delayed shutdown/restart
  - Machine lock-state polling
- **Telemetry**
  - Toggleable
  - Configurable interval (`5-300s`)
  - CPU/GPU usage, temperatures, disk usage
- **Home Assistant REST controls**
  - Entity list
  - Ambient preset (`select.ambilight_elobeallitas`)
  - Ambient power (`switch.led`)
  - Climate toggle (`climate.c630e267`)
  - Ambilight controls (`light.ambilight`: state/effect/brightness/color)
- **Persistent INI configuration**
  - `%USERPROFILE%\.desktop-event-handler\config.ini`

---

## Architecture

Solution is split by responsibility:

- `Beacon.App`
  - WPF host
  - Tray integration
  - BlazorWebView shell
  - startup + single-instance orchestration
- `Beacon.UI`
  - Razor components/pages
  - Fluent UI + Tailwind-based UX
- `Beacon.Core`
  - Domain models, contracts, constants, use-case abstractions
- `Beacon.Infrastructure`
  - MQTT, HA REST, Config, Telemetry, System actions, Single-instance transport
- `Beacon.Tests`
  - Unit/integration tests

---

## Tech Stack

- .NET 10
- WPF + Blazor Hybrid (`Microsoft.AspNetCore.Components.WebView.Wpf`)
- Fluent UI Blazor (`Microsoft.FluentUI.AspNetCore.Components`)
- TailwindCSS (CLI build pipeline)
- MQTTnet
- INI parser package
- System.Management
- Tray package (`H.NotifyIcon.Wpf` or equivalent)

---

## Code Quality Rules

The project enforces:

- Clean code principles
- Transparent folder/file naming
- Strict anti-pattern avoidance:
  - No God classes
  - No service locator
  - No static mutable global state
  - No swallowed exceptions
  - No magic strings for critical topic/entity/config names
- `.editorconfig` in repository root
  - Naming/style consistency
  - Nullable enabled
  - Prefer file-scoped namespaces
  - Analyzer + style compliance
  - Warnings-as-errors for main projects (unless explicitly justified)

---

## Prerequisites

- Windows 10/11
- .NET 10 SDK
- Node.js LTS (for Tailwind CLI)
- Home Assistant instance with MQTT integration configured
- Reachable MQTT broker

---

## Configuration

Config file location:

- `%USERPROFILE%\.desktop-event-handler\config.ini`

Main areas:

- MQTT broker/credentials
- Home Assistant host/port/token
- Telemetry enable + interval
- Window/settings state

---

## Home Assistant Entities

After successful connection and discovery publishing, expected entities include:

- `button.<node>_lock`
- `button.<node>_shutdown`
- `button.<node>_reset`
- `sensor.<node>_machine_status`
- `sensor.<node>_cpu_usage`
- `sensor.<node>_gpu_usage`
- `sensor.<node>_cpu_temperature`
- `sensor.<node>_gpu_temperature`
- `sensor.<node>_disk_<drive>_usage`

`<node>` is normalized from machine name.

---

## Build & Run

## 1) Restore + build

```bash
dotnet restore
dotnet build
```

## 2) Run app

```bash
dotnet run --project Beacon.App
```

## 3) Tailwind build (if separate step is used)

```bash
# from UI project folder (example)
npm install
npm run build:css
```

## 4) Tests

```bash
dotnet test
```

Acceptance Checklist

- App starts and remains in tray
- Settings opens from tray and double-click
- Single-instance activation works
- MQTT connect/reconnect status is visible
- MQTT test action works with clear feedback
- Discovery entities appear in Home Assistant
- MQTT commands trigger lock/shutdown/restart
- Telemetry publishes at configured interval
- HA ambient/climate/ambilight controls work
- Config persists across restart
- Codebase passes .editorconfig/analyzer expectations
