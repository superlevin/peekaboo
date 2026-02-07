# Windows Platform Compatibility Analysis

## Executive Summary

**Peekaboo cannot currently run on Windows** due to its deep dependency on macOS-specific system APIs and frameworks. This document analyzes the technical challenges of porting to Windows and provides possible alternative approaches.

## Key macOS Dependencies

### 1. Screen Capture Technologies

| macOS API | Purpose | Windows Equivalent |
|-----------|---------|-------------------|
| `ScreenCaptureKit` | High-performance screen recording and window capture | Windows Graphics Capture API (Windows 10 1803+) |
| `CGWindowListCreateImage` | Window and screen screenshots | BitBlt/PrintWindow (Win32) or Graphics Capture |
| `CGDisplayCreateImage` | Display capture | Desktop Duplication API |

**Impact**: Complete rewrite of screen capture layer required
- Files: `Core/PeekabooCore/Sources/PeekabooCapture/*.swift`
- Lines of code: ~2000

### 2. Accessibility and Automation

| macOS API | Purpose | Windows Equivalent |
|-----------|---------|-------------------|
| `AXUIElement` | UI element inspection and manipulation | UI Automation (UIA) |
| `CGEvent` | Mouse and keyboard event synthesis | SendInput/mouse_event/keybd_event |
| `AXObserver` | Listen to accessibility notifications | UI Automation event handlers |
| `NSAccessibility` | Accessibility protocols | IAccessible/IUIAutomation |

**Impact**: Complete rewrite of automation engine required
- Files: `AXorcist/*.swift`, `Core/PeekabooAutomation/*.swift`
- Lines of code: ~5000+
- Complexity: Very High (different API design philosophy)

### 3. Application and Window Management

| macOS API | Purpose | Windows Equivalent |
|-----------|---------|-------------------|
| `NSRunningApplication` | Application lifecycle management | Process API + WMI |
| `CGWindowListCopyWindowInfo` | Enumerate windows | EnumWindows/FindWindow |
| `Spaces API (private)` | macOS workspace management | Virtual Desktops (Windows 10+) |
| `NSWorkspace` | Application launching and management | ShellExecute/CreateProcess |

**Impact**: Medium rewrite
- Files: `Core/PeekabooCore/Sources/PeekabooAppManagement/*.swift`
- Lines of code: ~1500

### 4. Operating System Integration

| Feature | macOS Implementation | Windows Challenge |
|---------|---------------------|-------------------|
| Menu bar capture | Direct menu bar access | No equivalent concept (Taskbar ≠ Menu Bar) |
| Dock interaction | Dock API | Taskbar (different API) |
| System dialogs | NSOpenPanel/NSSavePanel | Common Dialogs (different architecture) |
| Permission system | TCC (Transparency, Consent & Control) | UAC + Privacy Settings (different model) |

## Technical Barriers

### 1. Programming Language and Runtime

- **Current**: 100% Swift with Swift Package Manager
- **Windows Challenge**:
  - Swift on Windows is still evolving (limited official support)
  - Many macOS frameworks don't exist on Windows
  - Would require port to C#/.NET or C++ with Win32 APIs

### 2. Dependent macOS Frameworks (167 references)

```swift
import AppKit          // UI and application framework
import Cocoa           // macOS UI framework
import ScreenCaptureKit // Screen capture
import ApplicationServices // Accessibility and events
import CoreGraphics    // Graphics and window APIs
import IOKit           // Hardware interaction
```

These **do not exist** on Windows.

### 3. Architecture Differences

| Aspect | macOS | Windows |
|--------|-------|---------|
| UI Framework | AppKit/SwiftUI | WPF/WinUI/Win32 |
| Accessibility | AX API (tree-oriented) | UIA (pattern-oriented) |
| Permissions | Fine-grained app permissions | UAC + App capabilities |
| Packaging | .app bundles | .exe + DLLs |
| Distribution | Homebrew, .dmg, App Store | MSI, MSIX, Store |

## Porting Effort Estimation

### Full Port (Native Windows App)

| Component | Effort | Complexity |
|-----------|--------|------------|
| Screen capture | 4-6 weeks | High |
| Automation engine | 8-12 weeks | Very High |
| App management | 3-4 weeks | Medium |
| CLI interface | 2-3 weeks | Medium |
| UI (if porting Mac app) | 6-8 weeks | High |
| Testing & debugging | 4-6 weeks | High |
| **Total** | **27-39 weeks** (~6-9 months) | - |

### Partial Port (CLI only, basic features)

- Screen capture + click/type: ~8-12 weeks
- Omitting: menu bar, Dock, Spaces, advanced window management

## Alternative Approaches

### Option 1: Cross-Platform Abstraction Layer (Recommended for POC)

```
┌─────────────────────────────────────┐
│     Peekaboo Core Logic             │
│     (Protocol-based)                │
├─────────────────────────────────────┤
│  Platform Abstraction Layer         │
├───────────────┬─────────────────────┤
│  macOS Impl   │   Windows Impl      │
│  (current)    │   (new)             │
└───────────────┴─────────────────────┘
```

**Pros**:
- Share core logic
- Can develop incrementally

**Cons**:
- Significant interface design work
- Feature parity challenges (macOS has features Windows doesn't)

### Option 2: Remote Protocol (Cross-platform Compatible)

Peekaboo as macOS server, Windows clients connect remotely:

```
┌──────────────┐      Network     ┌──────────────┐
│   Windows    │ ◄─────────────► │   macOS      │
│   Client     │   (HTTP/WS)     │   Peekaboo   │
│   (control)  │                 │   Server     │
└──────────────┘                 └──────────────┘
```

**Pros**:
- No porting of core features needed
- Windows users can remotely control Macs
- Good for cloud deployments

**Cons**:
- Requires macOS machine
- Network latency
- Security considerations

### Option 3: Dual Development (Separate Projects)

Create independent Windows version:
- Different codebase
- Different tech stack (C#/.NET)
- Similar user interface and features

**Pros**:
- Optimize for platform-specific features
- No cross-platform compromises

**Cons**:
- Double development and maintenance cost
- Feature disparity risk

### Option 4: Focus on MCP Protocol (Recommended Short-term)

Keep Peekaboo macOS-only, but:
- Enhance MCP server functionality
- Windows users connect via MCP clients
- Support remote Mac automation

**Pros**:
- Minimal additional work
- Leverage existing MCP ecosystem
- Cross-platform interface already exists via Claude Desktop/Cursor, etc.

**Cons**:
- Windows users still need access to macOS machine

## Windows-Specific Tools (Alternatives)

If Windows automation is needed, consider these mature tools:

| Tool | Description | License |
|------|-------------|---------|
| **Playwright** | Browser automation (cross-platform) | Apache 2.0 |
| **PyAutoGUI** | Python GUI automation library | BSD |
| **AutoHotkey** | Windows scripting and automation | GPL |
| **Sikuli** | Visual automation (screen images) | MIT |
| **UI Automation** | Microsoft official API (.NET) | Free |
| **WinAppDriver** | Windows app UI testing | MIT |

## Recommendations

### Short-term (0-6 months)
1. **Remain macOS-only** - Focus on core features and stability
2. **Enhance MCP support** - Allow Windows users to use remotely
3. **Document remote setup** - Provide Windows → Mac setup guides
4. **Clarify in README** - Avoid expectation mismatches

### Medium-term (6-12 months)
1. **Assess market demand** - How many Windows users?
2. **Explore partnerships** - Any Windows automation experts interested?
3. **Design abstraction layer** - If porting, design interfaces first

### Long-term (12+ months)
1. **Decision point**: Full port vs. hybrid approach vs. remain macOS-only
2. If porting, consider:
   - Seek sponsorship or funding
   - Build Windows-specialized team
   - Use phased approach (CLI first, then GUI)

## Conclusion

**Direct answer**: Peekaboo **cannot currently** run on Windows, and porting would require significant engineering investment (6-9 months full-time development).

**Recommended path**:
1. Stay macOS-only, enhance MCP server
2. Provide remote connection option for Windows users
3. Evaluate future porting work based on demand

**Key considerations**:
- macOS and Windows automation APIs are fundamentally different
- Full feature parity is extremely challenging
- Maintaining two platforms requires ongoing resource commitment
