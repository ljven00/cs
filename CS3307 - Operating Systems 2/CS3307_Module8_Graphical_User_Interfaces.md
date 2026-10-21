# CS 3307 — Operating Systems 2
## Module 8: Graphical User Interfaces

---

> **Course Progress:** Module 8 of 8 — CS 3307 (OS 2) ✓

---

## 1. GUIs and the OS

Every graphical application — a browser, a text editor, a game — ultimately runs on top of the OS. The GUI is not magic; it is a carefully layered stack that translates user input (mouse clicks, keystrokes, touch) into pixels on a screen, all coordinated by the operating system.

Understanding this stack makes you a better developer — you will know why GUIs behave the way they do, where performance problems come from, and how applications on different platforms achieve different looks and behaviors.

---

## 2. The GUI Stack

A GUI on a modern desktop system involves several distinct layers:

```
+----------------------------+
|    Your Application        |  (browser, text editor, game)
+----------------------------+
|    GUI Toolkit / Framework |  (Qt, GTK, WinForms, Cocoa, SwiftUI)
+----------------------------+
|    Window Manager          |  (manages window positions, decorations)
+----------------------------+
|    Display Server          |  (X11, Wayland, Win32, Quartz)
+----------------------------+
|    Graphics Driver         |  (GPU driver — OpenGL, Vulkan, Metal)
+----------------------------+
|    Hardware (GPU + Display)|
+----------------------------+
```

Each layer has a specific responsibility. An application developer mostly works at the toolkit layer and above — but problems anywhere in the stack can affect the application.

---

## 3. Display Servers

A **display server** is the OS component that manages the screen, input devices, and coordinates which application gets to draw where.

### 3.1 X Window System (X11)

The traditional display server for Unix/Linux, developed at MIT in 1984.

```
Application <---network protocol---> X Server <---> Hardware
```

Key design decisions:
- **Network transparent** — an application can run on a remote machine and display on your local screen (`ssh -X`)
- **Client-server model** — the X server manages the hardware; applications are clients
- The X server handles the display; a separate **window manager** handles window decorations and behavior

```bash
# Run a graphical application from a remote server on your local display
ssh -X user@remote-server
xclock    # clock appears on your local screen, runs remotely
```

### 3.2 Wayland

The modern replacement for X11, designed to fix X11's architectural problems.

| | X11 | Wayland |
|-|-----|---------|
| Age | 1984 | 2008+ |
| Architecture | Client-server, network transparent | Compositor is the server |
| Security | Poor (any app can see other apps' windows) | Strong isolation between apps |
| Performance | Extra copy operations | Direct rendering, lower latency |
| Status | Legacy (but still widely used) | Default on most modern Linux desktops |

In Wayland, the **compositor** does the job of both the display server and the window manager — it is the single authority over the screen.

### 3.3 Windows Display System (Win32 / DWM)

Windows has always had its own integrated display system:

- **Win32 GDI** — the original drawing API (Windows 3.1 onwards)
- **DWM (Desktop Window Manager)** — the compositor introduced in Windows Vista; enables transparency, animations, and hardware acceleration
- **DirectX / Direct2D** — high-performance graphics for games and modern apps

### 3.4 macOS Quartz Compositor

macOS uses the **Quartz Compositor** — fully hardware-accelerated since macOS 10.2. Every window is rendered into its own offscreen buffer and the compositor combines them. This is why macOS window dragging and animations are consistently smooth.

---

## 4. Window Managers and Compositors

A **window manager** controls:
- Window placement and sizing
- Title bars, borders, and decorations
- Focus (which window receives keyboard input)
- Virtual desktops / workspaces

### Types of Window Managers

| Type | Behavior | Example |
|------|----------|---------|
| **Stacking** | Windows overlap like papers on a desk | Openbox, Fluxbox |
| **Tiling** | Windows automatically fill the screen in a grid | i3, Sway, bspwm |
| **Compositing** | Renders each window off-screen and combines them | Mutter (GNOME), KWin (KDE) |

A **compositor** extends the window manager with hardware-accelerated compositing — enabling transparency, shadows, animations, and screen recording.

```
Each window:                      Compositor combines:
+----------+                      +----------------------+
| Window A |  -> offscreen buffer |  [A] overlapping [B] |
+----------+                      |  with shadows/blur   |
| Window B |  -> offscreen buffer +----------------------+
+----------+                      -> sent to display
```

---

## 5. GUI Toolkits

A **GUI toolkit** (also called a widget toolkit or framework) provides the building blocks for applications: buttons, text fields, menus, dialog boxes, and layout managers. It also handles the event loop.

### Major GUI Toolkits

| Toolkit | Language | Platform | Used In |
|---------|----------|----------|---------|
| **Qt** | C++ (Python via PyQt/PySide) | Cross-platform | VLC, KDE, many desktop apps |
| **GTK** | C (Python via PyGObject) | Linux-first, cross-platform | GIMP, GNOME apps |
| **Win32 / MFC** | C / C++ | Windows only | Legacy Windows apps |
| **WinForms / WPF** | C# / .NET | Windows | Enterprise Windows apps |
| **Cocoa / AppKit** | Objective-C / Swift | macOS | Native macOS apps |
| **SwiftUI** | Swift | Apple platforms | Modern iOS/macOS apps |
| **JavaFX / Swing** | Java | Cross-platform | Java desktop apps |
| **Electron** | HTML/CSS/JS | Cross-platform | VS Code, Slack, Discord |
| **Flutter** | Dart | Cross-platform (desktop+mobile) | Modern cross-platform apps |

### Choosing a Toolkit

- **Native look and feel** matters → use the platform's native toolkit (Cocoa on macOS, Win32/WinForms on Windows)
- **Cross-platform** → Qt (best performance), Electron (easiest for web developers), Flutter (modern)
- **Linux desktop** → Qt (KDE) or GTK (GNOME)

---

## 6. The Event Loop

Every GUI application is driven by an **event loop** — the core mechanism that keeps the application alive and responsive.

```
while (application is running):
    event = wait_for_next_event()   // blocks until something happens
    dispatch_event(event)           // route event to correct handler
```

Events include:
- Mouse clicks, moves, scrolls
- Keyboard presses and releases
- Window resize, minimize, close
- Timer expiry
- Network data available
- OS signals

### Event Loop Example (Pseudocode)

```python
app = Application()
window = Window("My App", width=800, height=600)
button = Button("Click me", on_click=handle_click)
window.add(button)

app.run()   # enters the event loop -- does not return until app exits
```

### Why the Event Loop Must Not Be Blocked

If your event handler takes too long (e.g., reading a large file, making a network request), the event loop cannot process new events — the UI **freezes**.

```
BAD:  on_click -> download_file()  # takes 5 seconds -- UI hangs!

GOOD: on_click -> start_download_in_background_thread()
                  update_ui_when_done_via_event()
```

Every GUI framework has a mechanism for running work off the main thread and posting results back to the event loop safely.

---

## 7. Rendering and Drawing

### 7.1 Immediate vs. Retained Mode

| Mode | How it works | Example |
|------|-------------|---------|
| **Retained mode** | Build a tree of objects (widgets); framework decides when to redraw | Qt, GTK, HTML DOM |
| **Immediate mode** | Redraw everything every frame based on current state | Dear ImGui, game UIs |

Most desktop toolkits use retained mode. Games and real-time tools often use immediate mode.

### 7.2 The Widget Tree

In retained mode, an application's UI is represented as a tree of widgets:

```
Window
├── MenuBar
│   ├── FileMenu
│   └── EditMenu
├── Toolbar
│   ├── Button (New)
│   └── Button (Save)
└── CentralWidget
    ├── Sidebar
    │   └── FileTree
    └── Editor
        └── TextArea
```

When a widget changes (e.g., text is typed), it marks itself **dirty** — the framework redraws only the dirty region, not the whole screen. This is called **dirty rectangle optimization**.

### 7.3 GPU Acceleration

Modern toolkits use the GPU for rendering via:

- **OpenGL** — cross-platform 3D/2D graphics API
- **Vulkan** — lower-level, higher-performance successor to OpenGL
- **Metal** — Apple's GPU API (macOS/iOS)
- **Direct2D / Direct3D** — Windows GPU API

Even simple text and button rendering benefits from GPU acceleration — the CPU is freed for application logic.

---

## 8. Input Handling

User input travels through several layers before reaching your application:

```
Physical device (keyboard/mouse)
        |
Device driver (kernel space)
        |
Input event system (e.g., Linux evdev, Windows WM_INPUT)
        |
Display server (X11/Wayland/Win32)
        |
Window manager (routes to focused window)
        |
GUI toolkit (dispatches to widget under cursor / in focus)
        |
Application event handler
```

### Keyboard Input

- The OS maps physical key scancodes to **keysyms** (logical key names)
- Modifier keys (Shift, Ctrl, Alt) are tracked separately
- Input method editors (IME) handle complex scripts (Chinese, Japanese, Arabic)

### Mouse / Touch Input

- Mouse events include position, button state, scroll delta
- Touch events add multi-touch tracking (finger IDs, pressure)
- **HiDPI / Retina** displays have a scale factor (e.g., 2x) — a 200x200 logical window might be 400x400 physical pixels

---

## 9. GUI Application Architecture

### Model-View-Controller (MVC)

The most common pattern for structuring GUI applications:

```
+----------+       updates       +----------+
|  Model   | <────────────────── | Controller|
| (data,   |                     | (handles  |
|  logic)  | ───────────────────►| user input|
+----------+    notifies         +----------+
     |                                |
     | notifies                       |
     ▼                                |
+----------+                          |
|   View   | <────────────────────────+
| (widgets,|     updates UI
| display) |
+----------+
```

- **Model** — application data and business logic (no UI code)
- **View** — widgets that display the model's data
- **Controller** — handles user input, updates the model, tells the view to refresh

Keeping these separated makes the code testable — you can test the model without a display.

### Signal-Slot Pattern (Qt)

Qt uses a **signal-slot** mechanism to connect UI events to handlers without tight coupling:

```cpp
// Connect button's clicked signal to a slot (handler function)
connect(button, &QPushButton::clicked, this, &MyWindow::onButtonClicked);

// The slot
void MyWindow::onButtonClicked() {
    label->setText("Button was clicked!");
}
```

Signals and slots can be connected across threads — Qt handles the thread safety automatically.

---

## 10. A Simple GUI Application — Python + Tkinter

Tkinter is Python's built-in GUI toolkit — available on every Python installation.

```python
import tkinter as tk
from tkinter import ttk

class TemperatureConverter(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Temperature Converter")
        self.geometry("300x150")
        self.resizable(False, False)

        # Input
        ttk.Label(self, text="Celsius:").grid(row=0, column=0, padx=10, pady=10)
        self.celsius_var = tk.StringVar()
        ttk.Entry(self, textvariable=self.celsius_var, width=10).grid(row=0, column=1)

        # Button
        ttk.Button(self, text="Convert", command=self.convert).grid(
            row=1, column=0, columnspan=2, pady=5)

        # Result
        self.result_var = tk.StringVar(value="Fahrenheit: --")
        ttk.Label(self, textvariable=self.result_var).grid(
            row=2, column=0, columnspan=2)

    def convert(self):
        try:
            c = float(self.celsius_var.get())
            f = c * 9/5 + 32
            self.result_var.set(f"Fahrenheit: {f:.1f}")
        except ValueError:
            self.result_var.set("Please enter a number")

if __name__ == "__main__":
    app = TemperatureConverter()
    app.mainloop()   # enters the event loop
```

This demonstrates: a widget tree (window → labels, entry, button), an event handler (`convert`), and `mainloop()` — the event loop.

---

## 11. GUI and the OS — How They Connect

The GUI toolkit communicates with the OS through several channels:

### File Dialogs

When you click "Open File", the toolkit calls an OS-provided dialog:
- Windows: `GetOpenFileName()` / `IFileOpenDialog`
- macOS: `NSOpenPanel`
- Linux: `GtkFileChooserDialog` or a portal to the desktop's file picker

### Clipboard

The clipboard is an OS service — `Ctrl+C` stores data in a shared OS buffer that any application can read.

### Drag and Drop

Drag-and-drop is coordinated by the display server — it tracks which window the cursor is over and passes data between applications.

### System Tray / Notifications

Background apps communicate status through OS notification APIs:
- Windows: `Shell_NotifyIcon`, Win32 toast notifications
- macOS: `NSUserNotificationCenter` / `UNUserNotificationCenter`
- Linux: `libnotify` / D-Bus notification service

### Accessibility

The OS provides an **accessibility API** that allows screen readers and assistive tools to inspect the widget tree:
- Windows: UI Automation (UIA)
- macOS: NSAccessibility
- Linux: AT-SPI (Assistive Technology Service Provider Interface)

---

## 12. Real-World Example: What Happens When You Click a Button

You click a "Save" button in a text editor:

1. **Hardware** — mouse generates an electrical signal
2. **Kernel / driver** — USB/HID driver converts it to an input event
3. **Display server** — X11 or Wayland receives the event, determines which window/widget is under the cursor
4. **Window manager** — confirms the window is in focus and passes the event
5. **GUI toolkit** — maps the screen coordinates to the button widget, fires the `clicked` signal
6. **Application event handler** — your `on_save_clicked()` function runs
7. **OS file API** — calls `open()`, `write()`, `fsync()`, `close()` (Module 4)
8. **File system** — inode updated, data blocks written (Module 3)
9. **UI update** — title bar updated ("document.txt" instead of "document.txt*"), status bar shows "Saved"

The entire path from muscle movement to data on disk, coordinated across hardware, kernel, display server, toolkit, and application.

---

## 13. CS 3307 Full Course Review

| Module | Topic | Core Idea |
|--------|-------|-----------|
| 1 | Modern Operating Systems | Design goals, kernel architectures, major OSes compared |
| 2 | Parallel Computing & Concurrency | Amdahl's Law, parallel models, advanced synchronization, lock-free |
| 3 | Memory and File Structure | TLB, multi-level page tables, inodes, journaling, CoW |
| 4 | File Handling | File descriptors, system calls, locking, atomic save pattern |
| 5 | File Processing | Text vs binary, streaming large files, formats, error handling |
| 6 | Command Shells and Control | Signals, traps, pipes, robust scripting patterns |
| 7 | Operating System Utilities | grep/sed/awk, system monitoring, process and network tools |
| 8 | Graphical User Interfaces | Display servers, window managers, toolkits, event loops |

---

## Module 8 Quiz

---

**Question 1**
What is the difference between X11 and Wayland? What was the main motivation for replacing X11?

> **Answer:** X11 is the traditional Unix/Linux display server developed in 1984. It uses a client-server model over a network protocol, making it network-transparent. However, X11 has poor security (any application can see and inject input into other applications' windows), extra rendering overhead (data is copied multiple times), and accumulated decades of legacy complexity. Wayland was designed to replace it by unifying the display server and compositor into one component, giving each application strong isolation, reducing latency through direct rendering, and providing a cleaner modern architecture. Most major Linux desktops (GNOME, KDE) now default to Wayland.

---

**Question 2**
What is an event loop and why must long-running operations never run directly inside an event handler?

> **Answer:** An event loop is the central mechanism of every GUI application — it continuously waits for events (mouse clicks, key presses, timer ticks), then dispatches each event to the appropriate handler. It must run continuously and return quickly from every handler to remain responsive. If a long-running operation (file download, database query, heavy computation) runs directly in an event handler, the event loop is blocked — no other events can be processed, the UI freezes, and the OS may mark the application as "not responding." Long operations must be offloaded to a background thread, with results posted back to the event loop when complete.

---

**Question 3**
Describe the MVC (Model-View-Controller) pattern and explain why separating the Model from the View is beneficial.

> **Answer:** MVC divides an application into three components: the Model holds the data and business logic (no UI code); the View displays the data using widgets; the Controller handles user input, updates the Model, and tells the View to refresh. Separating the Model from the View is beneficial because: (1) the Model can be tested without a display — unit tests run fast and reliably; (2) the same Model can have multiple Views (e.g., a list view and a chart view of the same data); (3) changes to the UI don't affect the business logic and vice versa; (4) the code is easier to understand and maintain because each piece has one clear responsibility.

---

**Question 4**
A user reports that a desktop application freezes for several seconds when they click "Export to PDF". What is the likely cause and how should it be fixed?

> **Answer:** The likely cause is that the PDF export operation (file I/O, rendering, compression) is running directly on the **main thread** inside the button's event handler, blocking the event loop. The fix is to move the export work to a **background thread**: when the button is clicked, start the export in a worker thread, show a progress indicator (spinner or progress bar), and when the thread finishes, post a completion event back to the main thread to update the UI (hide the spinner, show "Export complete"). Most GUI frameworks provide mechanisms for this — Qt's `QThread`, Python's `threading` + tkinter's `after()`, or async patterns in modern frameworks.

---

**Question 5**
Trace the complete path of what happens at the OS and GUI level when you press Ctrl+S in a text editor to save a file. Name at least five distinct layers or components involved.

> **Answer:** The complete path involves:
>
> 1. **Keyboard hardware** — generates a USB HID scan code for the 'S' key with Ctrl modifier
> 2. **Kernel input driver** — converts the scan code to a Linux `evdev` input event
> 3. **Display server (X11/Wayland)** — receives the key event and routes it to the focused window
> 4. **GUI toolkit** — recognizes the Ctrl+S key combination and fires the associated action/signal
> 5. **Application event handler** — the `on_save()` function is called
> 6. **OS file API** — the application calls `open()`, `write()`, `fsync()`, `close()` (or the atomic temp-file-then-rename pattern)
> 7. **VFS (Virtual File System)** — the kernel's file system abstraction layer routes the call to the correct file system driver
> 8. **File system driver (ext4/NTFS)** — updates the inode (timestamps, size), writes data blocks, commits to the journal
> 9. **Block device driver** — issues write commands to the SSD or HDD
> 10. **UI update** — the application posts an event back to remove the unsaved indicator from the title bar

---

## Congratulations — CS 3307 Complete!

You have completed the full Operating Systems 2 curriculum:

```
✓ Module 1 — Modern Operating Systems
✓ Module 2 — Parallel Computing and Concurrency
✓ Module 3 — Memory and File Structure
✓ Module 4 — File Handling
✓ Module 5 — File Processing
✓ Module 6 — Command Shells and Control
✓ Module 7 — Operating System Utilities
✓ Module 8 — Graphical User Interfaces
```

---

## Full Two-Course Summary

### CS 2301 — Operating Systems 1

| Module | Topic |
|--------|-------|
| 1 | Introduction to Operating Systems |
| 2 | Process Management & CPU Scheduling |
| 3 | Concurrency |
| 4 | Memory Management |
| 5 | Storage Management and Shell Scripting |
| 6 | Input / Output Management |
| 7 | Security and Protection |
| 8 | Advanced Topics and Case Studies |

### CS 3307 — Operating Systems 2

| Module | Topic |
|--------|-------|
| 1 | Modern Operating Systems |
| 2 | Parallel Computing and Concurrency |
| 3 | Memory and File Structure |
| 4 | File Handling |
| 5 | File Processing |
| 6 | Command Shells and Control |
| 7 | Operating System Utilities |
| 8 | Graphical User Interfaces |

You now have a complete foundation in operating systems — from how the kernel manages hardware, to how GUIs sit on top of it all.

---

*CS 3307 Operating Systems 2 — Module 8 of 8 — Course Complete*
