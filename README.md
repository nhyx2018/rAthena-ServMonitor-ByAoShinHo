# 🎛️ RO Serv Monitor
> A high-performance, non-blocking process orchestrator and log virtualization platform engineered in C# (WPF) to monitor, build, and scale rAthena and Hercules private server infrastructures.

![Platform](https://img.shields.io/badge/Platform-Windows_Desktop-blue?style=for-the-badge&logo=windows)
![Framework](https://img.shields.io/badge/.NET_Framework-4.8.1-blue?style=for-the-badge&logo=dotnet)
![UI Architecture](https://img.shields.io/badge/UI_Subsystem-WPF_//_Win32_Interop-green?style=for-the-badge)
[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/AoShinRO/rAthena-ServMonitor-ByAoShinHo)

<img width="1608" height="790" alt="RO Serv Monitor Interface" src="https://github.com/user-attachments/assets/c2001d04-772f-4aa4-8495-2267424351f0" />

**RO Serv Monitor** is a professional desktop management dashboard built specifically to abstract the complexities of running multi-tier MMORPG server architectures. By combining asynchronous Windows process piping with granular string streaming evaluation, the monitor offers centralized runtime controls, hot-swappable environment configurations, local node automation (ROBrowser compilation/serving), and automated cross-compiling diagnostics.

---

## 🏗️ Technical Architecture & Internal Modules

To bypass standard WPF layout-coupling limitations and prevent UI thread starvation (freezing), the codebase is organized into a decoupled helper pipeline layer interacting with asynchronous UI update loops:


```
              ┌────────────────────────────────────────┐
              │          WPF Window Subsystem          │
              │   (MainWindow / OptionsWnd / Logs)     │
              └───────────────────┬────────────────────┘
                                  │ (Data Binding & Actions)
                                  ▼
┌───────────────────┬───────────────────┬───────────────────┐
│                   │                   │                   │
▼                   ▼                   ▼                   ▼

[ IProcess ]        [ IText ]          [ ILogging ]       [ IAnimation ]
──► Process IO      ──► ANSI Stripping  ──► Log Buffering  ──► Buttons Animation
──► Parallel Kills  ──► Regex Parsing   ──► Tray Context   ──► Thickness

```

### ⚙️ Under-The-Hood Implementations

* **Asynchronous Process Interception:** Uses the Task-Based Asynchronous Pattern (`TAP`) combined with custom stream redirection to capture child process standard outputs (`stdout`/`stderr`) dynamically without dropping frames or blocking the core STA thread.
* **Orphaned Subprocess Garbage Collection:** Features a defensive lifecycle routine (`KillOrphanProcesses`) executing via `Parallel.ForEach` over active process pools, enforcing atomic cleanup across rogue backend instances (`taskkill /F /T`) to prevent memory/port leaks.
* **ANSI Stream Ingestion & Parsing:** Incorporates a fast compiled Regex pattern wrapper (`IText.RemoveAnsi`) running at the ingestion stage to wipe byte formatting escape sequences ($0x1B[...m$), making console feedback readable before passing it to the UI text buffers.
* **Low-Overhead Log Virtualization:** Maps intercepted telemetry streams directly into memory models (`std::unordered_map` layout concepts modeled in C# dictionaries) to update diagnostic badge counts (`CounterError`, `CounterSql`) on the system tray context menus in real time.

---

## 🛠️ Deep Dive Core Subsystems

### 1. Unified Management Profile Matrix
The engine handles isolated configuration scopes for asymmetric backend binaries seamlessly through unified configurations:
* **Core Emulators:** Direct runtime attachment to `login-server.exe`, `char-server.exe`, `map-server.exe`, and `web-server.exe`.
* **Node Infrastructure Support:** Spawns and supervises localized instance sockets for `wsproxy` gateways and fully compiled `npm` web applications.

### 2. Multi-Target Compiler Automation
Detects project configurations on initialization by scanning the workspace footprint (`rAthena.sln`, `brHades.sln`, `Hercules.sln`), automatically injecting preprocessor directives:
* **Pre-Renewal Flags:** Injects the global `#define PRERE` constant dynamically through automated standard build calls (`MSBuild` / `CMake`).

---

## ⚙️ Configuration Macro Mapping (UI & Runtime)

The orchestration rules can be calibrated inside the `OptionsWnd` boundary layer:

| Component Feature | Internal Execution Logic | UI Component Vector |
| :--- | :--- | :--- |
| **White Mode Toggle** | Switches system coloring dictionaries to low-contrast templates | `WhiteMode_Checked` |
| **DevMode Automation** | Exposes auxiliary standard compilation windows for environment pipelines | `DevMode_Checked` |
| **CMake Native Pipe** | Forces generation via Unix Makefiles instead of classic solution files | `CmakeMode_Checked` |
| **Numeric Restrictions** | Rejects non-digit char arrays at input boundary for port binding loops | `WSproxy_PreviewTextInput` |
| **Font Rendering Cache** | Ingests System Fonts (`Fonts.SystemFontFamilies`) on background thread loops | `FontSelector_Loaded` |

---

## 🚀 Environment Requirements & Production Build

### Minimum OS Compatibility
* Windows 7 / 8 / 10 / 11 ($x86$/$x64$)

### Target Environment Dependencies
* **Development Target:** .NET Framework 4.8.1 (WPF Subsystem)
* **Build Targets (Path Visible):** MSBuild Engine / CMake Compiler Core
* **Web Serving (Optional Stack):** Node.js LTS Engine + Global npm packages

## 💾 Download & Quick Start

### 🚀 Pre-Compiled Release
If you don't want to compile the source code manually, you can download the ready-to-use executable directly from the **[GitHub Releases](https://github.com/AoShinRO/ROServMonitor/releases)** section.

### Local Compilation Steps
1. Open the primary project file `AoShinhoServ_Monitor.sln` inside Visual Studio 2022.
2. Alter the configuration profile target to **Release | AnyCPU** or **Release | x86**.
3. Rebuild the solution. The pipeline generates an isolated deployment artifact under the target output root directory:

```
\bin\Release\AoShinhoServ_Monitor.exe
```

---

## ⚠️ Research & Diagnostics Notice

This software is an open-source automation tool provided to the server research community under the **Unlicense** terms. It is engineered strictly to handle system logs virtualization, build scripting execution, and server health tracking inside independent development environments. Always confirm that local executables adhere to secure connection privileges before binding to live server interfaces.
