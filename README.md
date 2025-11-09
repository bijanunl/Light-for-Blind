# Light for Blind
**Track:** Banana (Wildcard/Creative)  
**Status:** Prototype (WinForms, .NET Framework 3.5)  
**Short pitch:** A small Windows app that **speaks what has focus** and **echoes keystrokes** so blind/low‑vision users can navigate any PC quickly. Built in C# with offline TTS.  


## What this prototype does
- **Focus announcements:** When the foreground window or UI focus changes, it speaks the **accessible name** of the focused element.  
  - Implemented via Win32 `SetWinEventHook` for `EVENT_SYSTEM_FOREGROUND` and `EVENT_OBJECT_FOCUS`, then MSAA (`oleacc.dll`) to get `IAccessible` and read `accName`.  
- **Keyboard echo:** Announces the **keys you press** (e.g., letters, numbers, function keys).  
- **Caps Lock state:** Speaks “caps lock turns on/off” when toggled.  
- **Offline TTS:** Uses `System.Speech.Synthesis.SpeechSynthesizer` for local speech (no internet).

> This repo contains two projects in a single solution. **BlindLight** is the main app; **SpeechBuilder** provides a tiny speech wrapper (`SpeechControl`).

---

## Project structure
```
Source Code/
├─ BlindLight/                # Main WinForms app (.NET Framework 3.5, WinExe)
│  ├─ Form1.cs                # Core logic: hooks, focus handler, key echo
│  ├─ Program.cs              # App entry; passes SpeechControl into Form1
│  ├─ Properties/             # Assembly info, resources, settings
│  ├─ bin/Debug/              # Built artifacts (includes Gma.UserActivityMonitor.dll)
│  └─ BlindLight.csproj
├─ SpeechBuilder/             # Speech wrapper (also WinForms project, but used as lib)
│  ├─ SpeechControl.cs        # `speak(string)` and `stop()` using SpeechSynthesizer
│  ├─ Properties/
│  └─ SpeechBuilder.csproj
└─ BlindLight.sln             # Solution with both projects
```

**Key files (confirmed from source):**
- `BlindLight/Form1.cs` → Hooks system events + MSAA to announce `accName` and echo keys.  
- `BlindLight/Program.cs` → Creates `SpeechControl` and launches `Form1`.  
- `SpeechBuilder/SpeechControl.cs` → Wraps `SpeechSynthesizer` with `speak()` and `stop()`.

---

## How it works (under the hood)

1) **Listen for focus changes**  
   - P/Invoke: `SetWinEventHook` subscribes to `EVENT_OBJECT_FOCUS` and `EVENT_SYSTEM_FOREGROUND`.  
2) **Fetch accessible object**  
   - P/Invoke: `AccessibleObjectFromEvent` → `IAccessible` + child id; then read `iAccessible.get_accName(child)`.  
3) **Speak it**  
   - `SpeechBuilder.SpeechControl.speak(string)` → `SpeechSynthesizer.SpeakAsync(...)`.  
4) **Keyboard echo**  
   - Global key hooks via **Gma.UserActivityMonitor** (`HookManager.KeyDown/KeyUp`) announce pressed keys; special handling for Caps Lock.

> Note: This prototype uses **MSAA (oleacc / IAccessible)** rather than the newer UI Automation API. It’s simple and works on many apps, but some modern controls may expose richer info only via UIA.

---

## System requirements

- Windows 10/11 (x64)
- **.NET Framework 3.5** (Developer Pack recommended for building)
- Visual Studio 2019/2022 with **.NET desktop development** workload
- Runtime: no internet required (speech is local)

---

## Build & Run (from source)

1. Open `Source Code/BlindLight.sln` in Visual Studio.  
2. Ensure **.NET Framework 3.5** targeting pack is installed (VS Installer ▸ Individual components).  
3. Build the solution (Debug or Release).  
4. Run the **BlindLight** project.

### Dependencies
- **Gma.UserActivityMonitor** (global keyboard hook).  
  - The DLL exists under `BlindLight/bin/Debug/Gma.UserActivityMonitor.dll`.  
  - Optionally, add via NuGet: `Gma.System.MouseKeyHook` (modern package) and update code namespaces if you migrate.

---

## 📦 Install from packaged EXE/MSI

The `Software EXE` archive includes a **Visual Studio Deployment Project** with MSI:

- `Software EXE/Debug/setup.exe` → launches the installer.  
- `Software EXE/Debug/Light For Blind.msi` → direct MSI install.

> On first run, allow the app through any AV prompts (it uses global keyboard hooks).

---

## Behaviors & controls

- **Automatic focus readout:** Move focus with `Tab`, click controls, or Alt‑Tab between windows → it announces the control’s name.  
- **Type & hear:** Keystrokes are spoken as you type.  
- **Caps Lock feedback:** Toggles are announced (“caps lock turns on/off”).  
- **Stop speech:** Starting a new announcement cancels the previous (`SpeakAsyncCancelAll`).

> This prototype **does not implement OCR** or browse-mode yet. It focuses on reliable focus announcements + key echo as a foundation.

---

## Known limitations

- **MSAA-only:** Uses `IAccessible.accName`; some apps expose richer info via **UI Automation** (not yet implemented).  
- **Element value/text:** Prototype reads names; not all values or states are announced.  
- **No OCR fallback:** Image-only dialogs/PDFs are not read yet.  
- **No user-configurable hotkeys:** Key echo is global; there is no hotkey layer for commands in this version.  
- **Two WinExe projects:** `SpeechBuilder` is referenced as a project; it also has its own Form and EXE but is used here for its speech class.


---

## License

- MIT License and Copyright (c) by Bijan Paul, 2025
- This project — **Light for Blind** — was initiated and developed in part during the University of Nebraska–Lincoln event **CornHacks 2025**. I gratefully acknowledge the CornHacks organizers, mentors, and peers whose feedback informed the early prototypes and direction of this work.



## 📝 Acknowledgements

- **Gma.UserActivityMonitor** for simple global key hooks.  
- Windows **MSAA (oleacc / IAccessible)** and **System.Speech.Synthesis** for the lightweight baseline.  
- The blind and low‑vision community who inspired the focus‑first design.

