## Project Overview

**RO Serv Monitor** is a Windows desktop application built with **WPF (.NET Framework 4.8.1)** in **C#**. Its purpose is to monitor, manage, and interact with game servers based on rAthena, Hercules, and brHades (Ragnarok Online emulators). The application provides real-time process control, categorized log viewing, server statistics, and developer tools such as build automation and ROBrowser integration.

- **Root namespace:** `AoShinhoServ_Monitor`
- **Platform:** Windows-only (WPF)
- **Target Framework:** .NET Framework 4.8.1
- **Solution file:** `AoShinhoServ-Monitor.sln`
- **Project file:** `AoShinhoServ-Monitor.csproj`

---

## Directory Structure

```
.
├── .github/workflows/       # CI/CD (dotnet-desktop.yml, codeql.yml)
├── Forms/                   # WPF sub-windows (Options, Logs)
│   ├── Logs.xaml / Logs.xaml.cs
│   └── OptionsWnd.xaml / OptionsWnd.xaml.cs
├── Properties/              # AssemblyInfo, Resources, Settings
│   ├── Settings.settings    # Persisted user settings
│   ├── Settings.Designer.cs
│   ├── Resources.resx
│   └── AssemblyInfo.cs
├── Images/                  # Icon and background assets
├── App.xaml / App.xaml.cs   # WPF application entry point
├── MainWindow.xaml          # Main layout (XAML)
├── MainWindow.xaml.cs       # Code-behind: process control, log pipeline, UI orchestration
├── Configuration.cs         # Persistence of server executable paths
├── Consts.cs                # Data types: rAthena.Type, rAthena.Error, rAthena.Data
├── ErrorHandler.cs          # Centralized error display via MessageBox
├── IAnimation.cs            # Button hover/press animations (ThicknessAnimation)
├── ILogging.cs              # Global state: counters, tray icon, process/error lists
├── IProcess.cs              # Process lifecycle control (kill, type detection, path validation)
├── IText.cs                 # Log parsing: ANSI removal, colorization, WPF formatting
└── README.md
```

---

## Architecture and Key Components

### UI Layer (WPF)

| File | Responsibility |
|---|---|
| `MainWindow.xaml` / `MainWindow.xaml.cs` | Main dashboard. Contains RichTextBox panels for each server (Login, Char, Map, Web, Dev, ROBrowser, wsProxy), Start/Stop/Restart/Compile buttons, and all orchestration logic. |
| `Forms/OptionsWnd.xaml.cs` | Settings window: executable paths, modes (Dev, White, ROBrowser, CMake, PreRenewal), font family and size. |
| `Forms/Logs.xaml.cs` | Log viewer and exporter for error/warning/SQL/debug entries. |

### Logic and State Layer

| File | Responsibility |
|---|---|
| `IProcess.cs` | Static class for process management: `KillAll()` kills processes by PID, `Do_Kill_All()` iterates all tracked processes, `CheckServerPath()` validates executable existence, `GetProcessType()` identifies server type by process name. |
| `IText.cs` | Static class for text parsing: `RemoveAnsi()` strips ANSI escape codes, `RunColoredText()` creates colored WPF `Run`, `AppendColoredText()` builds `Paragraph` with colored header, `GetMessageTypeColor()` maps headers like `[Error]`, `[SQL]`, `[Warning]` to colors. |
| `ILogging.cs` | Global application state: counters (`CounterError`, `CounterSql`, `CounterWarning`, `CounterDebug`, `CounterOnline`), system tray icon (`NotifyIcon`), tracked process list (`processesInfos`), error log list (`errorLogs`), and references to sub-windows (`OptWin`, `LogWin`). |
| `Consts.cs` | Core data types under the `rAthena` class: `Type` enum (Map, Login, Char, Web, DevConsole, WSproxy, ROBrowser), `Error` class, `ProcessesInfo` class, `Data` struct (Header, Body, Paint). |
| `Configuration.cs` | Static class holding executable paths (`LoginPath`, `CharPath`, `MapPath`, `WebPath`, `RobPath`) with a `Save()` method that persists to `Properties.Settings.Default`. |
| `ErrorHandler.cs` | Static class with a single `ShowError()` method that displays errors via `MessageBox`. |
| `IAnimation.cs` | Static class for button animations: `F_Thickness()` creates `ThicknessAnimation`, `F_Grid()` applies hover/press margin animations to Grid elements. |

---

## Data Flow

```
Server Process (stdout/stderr)
        │
        ▼
  Proc_DataReceived()          ← DataReceivedEventHandler in MainWindow.xaml.cs
        │
        ▼
  ParseServerData()            ← Strips ANSI, extracts [Header] and Body, assigns color
        │
        ▼
  Proc_Data2Box()              ← Dispatches to UI thread, updates counters, appends to RichTextBox
        │
        ├──► Error/Warning/SQL/Debug counters (ILogging)
        ├──► Error log list (ILogging.errorLogs)
        └──► RichTextBox display (IText.AppendColoredText)
```

---

## User Settings (Properties/Settings.settings)

All settings are user-scoped and persisted automatically:

| Setting | Type | Default | Description |
|---|---|---|---|
| `LoginPath` | string | (empty) | Path to `login-server.exe` |
| `CharPath` | string | (empty) | Path to `char-server.exe` |
| `MapPath` | string | (empty) | Path to `map-server.exe` |
| `WebPath` | string | (empty) | Path to `web-server.exe` |
| `ROBPath` | string | (empty) | Path to ROBrowser project folder |
| `WhiteMode` | bool | false | Light/dark mode toggle |
| `DebugMode` | bool | false | Show/hide loading messages in logs |
| `DevMode` | bool | false | Enable developer mode (compile panel) |
| `UseCMake` | bool | false | Use CMake instead of MSBuild |
| `PreRenewal` | bool | false | Build with PRERE define |
| `ROBMode` | bool | false | Enable ROBrowser integration panel |
| `ROBH` | bool | false | ROBrowser `-H` flag toggle |
| `wsport` | int | 5999 | WebSocket proxy port |
| `FontFamily` | string | Calibri | UI font family |
| `FontSize` | int | 12 | UI font size |

---

## Build and CI

- **Local build:** Open `AoShinhoServ-Monitor.sln` in Visual Studio or run `msbuild` from the command line. Configurations: `Debug`, `Release`.
- **CI pipeline:** `.github/workflows/dotnet-desktop.yml` runs on `windows-latest`, builds both Debug and Release configurations using MSBuild, and runs `dotnet test`.
- **Code analysis:** `.github/workflows/codeql.yml` provides CodeQL scanning.

---

## Key Patterns and Conventions

1. **Static utility classes prefixed with `I`:** `IProcess`, `IText`, `ILogging`, `IAnimation` — these are NOT interfaces despite the `I` prefix. They are static classes holding global state or utility methods.
2. **Process tracking:** Every spawned process is tracked via `ILogging.processesInfos` (a `List<rAthena.ProcessesInfo>`) using PID and type. This list is used for kill-all operations and process type resolution.
3. **UI thread marshalling:** All UI updates from background process output go through `Application.Current.Dispatcher.BeginInvoke()` with `DispatcherPriority.Background`.
4. **Log parsing pipeline:** Raw stdout/stderr → `RemoveAnsi()` → extract `[Header]` → `GetMessageTypeColor()` → `AppendColoredText()` → RichTextBox.
5. **Multi-emulator support:** The compile feature auto-detects the emulator by scanning for `rAthena.sln`, `brHades.sln`, or `Hercules.sln` in the server directory. brHades gets C++20 flags automatically.
6. **Window management:** Sub-windows (`OptionsWnd`, `Logs`) are created once as static instances in `ILogging` and shown/hidden rather than created/destroyed.
7. **No MVVM:** The project uses WPF code-behind pattern directly, not MVVM. All logic lives in `.xaml.cs` files and static utility classes.

---

## Common Tasks

### Adding a new server type
1. Add a new value to `rAthena.Type` enum in `Consts.cs`.
2. Add a new `ProcessesInfo` tracking case in `IProcess.GetProcessType()`.
3. Add a new RichTextBox in `MainWindow.xaml` and wire it in `Proc_DataReceived()`.
4. Add kill handling in `IProcess.KillAll()` and `Do_Kill_All()`.

### Adding a new log category
1. Add color mapping in `IText.GetMessageTypeColor()`.
2. Add counter and UI update in `Proc_Data2Box()` in `MainWindow.xaml.cs`.
3. Add counter property in `ILogging.cs`.

### Adding a new user setting
1. Add the setting in `Properties/Settings.settings`.
2. Reference it via `Properties.Settings.Default.<Name>`.
3. If it needs to be exposed in the options window, add a control in `Forms/OptionsWnd.xaml` and wire it in `OptionsWnd.xaml.cs`.

---

## Testing

There are currently no unit tests in the project. The CI workflow runs `dotnet test` but there is no test project configured.

---

## Important Notes

- This is a **Windows-only** application (WPF dependency).
- The `I`-prefixed classes (`IProcess`, `IText`, `ILogging`, `IAnimation`) are **static classes, not interfaces**.
- Process killing for cmd-spawned processes (DevConsole, ROBrowser, WSproxy) uses `taskkill /F /T` to kill the entire process tree, while direct server processes use `Process.Kill()`.
- The application uses `System.Windows.Forms.NotifyIcon` for system tray integration alongside WPF.
