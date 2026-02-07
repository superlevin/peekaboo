# Peekaboo Internationalization (i18n) and Localization (l10n) Strategy

## Overview

This document outlines the internationalization and localization implementation strategy for the Peekaboo project, with a focus on Traditional Chinese support while establishing an extensible framework for future languages.

## Current State

### Existing Architecture
- **No localization infrastructure**: All strings are hardcoded in Swift source code
- **No .strings files**: No language resource files created
- **No .lproj directories**: No localization folder structure
- **Zero NSLocalizedString usage**: No localization APIs used

### String Distribution

| Location | String Types | Estimated Count | Example Files |
|----------|-------------|-----------------|---------------|
| CLI Commands | Help text, flags, descriptions | ~500 | `*+CommanderMetadata.swift` |
| Error Messages | Error descriptions, suggestions | ~200 | `ErrorTypes.swift` |
| Mac App | UI labels, buttons, settings | ~150 | `SettingsWindow.swift` |
| Agent Output | System messages, status | ~100 | `AgentMessages.swift` |
| Documentation | READMEs, docs | ~50 files | `docs/*.md` |

**Total**: ~950+ strings requiring localization

## Recommended Implementation Strategy

### Phase 1: Documentation Localization (Immediate, No Code Changes)

✅ **Completed**:
- [x] Traditional Chinese README (`README.zh-TW.md`)
- [x] Windows platform analysis document (Traditional Chinese)
- [x] Localization strategy document (this document)

🔄 **In Progress**:
- [ ] Translate key documentation:
  - `ARCHITECTURE.md` → `ARCHITECTURE.zh-TW.md`
  - `docs/building.md` → `docs/building.zh-TW.md`
  - `docs/permissions.md` → `docs/permissions.zh-TW.md`
  - Command docs: `docs/commands/*.md` → `docs/commands/*.zh-TW.md`

### Phase 2: Infrastructure Setup (Code Changes Required)

#### 2.1 Create Localization Framework

```
Apps/CLI/Sources/Resources/
├── zh-TW.lproj/
│   ├── Localizable.strings      # General strings
│   ├── Commands.strings         # CLI command strings
│   └── Errors.strings           # Error messages
├── en.lproj/                    # English (base language)
│   ├── Localizable.strings
│   ├── Commands.strings
│   └── Errors.strings
└── ja.lproj/                    # Future: Japanese
    └── ...
```

#### 2.2 Swift Localization API Choice

**Option A: Traditional NSLocalizedString** (Recommended for CLI)
```swift
// Before
let message = "No displays available for capture."

// After
let message = NSLocalizedString(
    "error.no_displays",
    comment: "Error when no displays are available"
)
```

**Option B: Swift 6.2+ String(localized:)** (Recommended for Mac App)
```swift
// Before
Text("Welcome to Peekaboo!")

// After
Text(String(localized: "welcome.title", 
            comment: "Welcome screen title"))
```

**Option C: Structured String Catalogs** (Future, Swift 5.9+)
```swift
// Using .xcstrings file (Xcode 15+)
// Auto-generated type-safe API
Text(Strings.welcome.title)
```

### Phase 3: CLI Localization (Priority)

#### 3.1 Command Descriptions

**Current Structure** (Commander framework):
```swift
extension WindowListCommand: Describable {
    static var commandName: String { "list" }
    static var commandHelp: String { 
        "List all windows across all applications" 
    }
    // ...
}
```

**Localized Implementation**:
```swift
extension WindowListCommand: Describable {
    static var commandName: String { "list" }
    static var commandHelp: String { 
        String(localized: "command.window.list.help",
               defaultValue: "List all windows across all applications",
               comment: "Window list command help text")
    }
    // ...
}
```

#### 3.2 Error Messages

**Current Pattern**:
```swift
enum CaptureError: LocalizedError {
    case noDisplaysAvailable
    
    var errorDescription: String? {
        switch self {
        case .noDisplaysAvailable:
            return "No displays available for capture."
        }
    }
}
```

**Localized Implementation**:
```swift
enum CaptureError: LocalizedError {
    case noDisplaysAvailable
    
    var errorDescription: String? {
        switch self {
        case .noDisplaysAvailable:
            return String(localized: "error.capture.no_displays",
                         defaultValue: "No displays available for capture.",
                         comment: "Error when no displays found")
        }
    }
}
```

### Phase 4: Mac App Localization

#### 4.1 SwiftUI Interface

**Before**:
```swift
struct SettingsWindow: View {
    var body: some View {
        TabView {
            GeneralSettingsView()
                .tabItem {
                    Label("General", systemImage: "gear")
                }
            PermissionsSettingsView()
                .tabItem {
                    Label("Permissions", systemImage: "lock.shield")
                }
        }
    }
}
```

**After**:
```swift
struct SettingsWindow: View {
    var body: some View {
        TabView {
            GeneralSettingsView()
                .tabItem {
                    Label(String(localized: "settings.general"), 
                          systemImage: "gear")
                }
            PermissionsSettingsView()
                .tabItem {
                    Label(String(localized: "settings.permissions"), 
                          systemImage: "lock.shield")
                }
        }
    }
}
```

#### 4.2 XIB/Storyboard (if used)

In Xcode:
1. Select project → Info tab
2. Localizations → Add "Chinese (Traditional)"
3. Enable localization for each .xib/.storyboard
4. Translate generated .strings files

### Phase 5: Tooling and Automation

#### 5.1 String Extraction

**Create Script**: `scripts/extract-strings.sh`
```bash
#!/bin/bash
# Extract all NSLocalizedString calls from Swift files
find Apps/ Core/ -name "*.swift" -exec \
    genstrings -o Apps/CLI/Sources/Resources/en.lproj {} +
```

#### 5.2 Translation Validation

**Create Script**: `scripts/validate-translations.sh`
```bash
#!/bin/bash
# Check for missing translation keys
for lang in zh-TW ja; do
    comm -23 \
        <(grep -o '"[^"]*"' en.lproj/Localizable.strings | sort) \
        <(grep -o '"[^"]*"' ${lang}.lproj/Localizable.strings | sort)
done
```

#### 5.3 Machine Translation Assistance (Draft)

Use AI for initial translation, then human review:
```bash
# Use Claude/GPT to translate .strings files
# Requires human review for technical terms and context
```

## Traditional Chinese Specific Considerations

### Fonts and Rendering
- ✅ macOS has excellent built-in Chinese font support (PingFang TC)
- ✅ SwiftUI/AppKit handle CJK characters automatically
- ⚠️ Ensure text containers have sufficient space (Chinese can be longer)

### Terminology Standardization

| English Term | Traditional Chinese | Notes |
|--------------|-------------------|-------|
| Screen Capture | 螢幕擷取 | Not "截圖" |
| Accessibility | 輔助使用 | macOS official translation |
| Automation | 自動化 | Standard terminology |
| Snapshot | 快照 | Technical term |
| CLI | 命令列介面 or CLI | Keep abbreviation or translate |
| Agent | 代理 | In AI context |
| Window | 視窗 | Traditional spelling (not "窗口") |
| Menu Bar | 選單列 | macOS interface term |
| Dock | Dock | Keep original (macOS-specific) |

### Cultural Adaptation
- **Date format**: ROC year vs. Gregorian (recommend using system settings)
- **Typography**: Support vertical text (not needed for Peekaboo currently)
- **Punctuation**: Use full-width punctuation (，。！？)

## Testing Strategy

### Test Checklist
- [ ] App displays Chinese when system language is Traditional Chinese
- [ ] All CLI command help text properly localized
- [ ] Error messages display in correct language
- [ ] UI elements don't truncate due to text length
- [ ] Mixed language environments correctly fall back to default
- [ ] JSON output remains in English (API compatibility)

### Test Commands
```bash
# Set system language
defaults write NSGlobalDomain AppleLanguages -array "zh-TW"

# Test CLI
peekaboo --help
peekaboo see --help
peekaboo capture invalid-param  # Should show Chinese error

# Restore English
defaults write NSGlobalDomain AppleLanguages -array "en-US"
```

## Maintenance Workflow

### When Adding New Strings
1. Use `String(localized:)` instead of hardcoded strings
2. Provide clear comments explaining context
3. Run `scripts/extract-strings.sh`
4. Update all language .strings files
5. Tag translation needed in PR

### Pre-release Checklist
1. Run `scripts/validate-translations.sh`
2. Manually review new translations
3. Test all supported languages
4. Update CHANGELOG (multilingual)

## Cost Estimation

### Initial Implementation (Traditional Chinese)

| Task | Effort | Priority |
|------|--------|----------|
| Documentation translation | 2-3 weeks | ✅ High |
| Create localization infrastructure | 1 week | High |
| CLI string localization | 2-3 weeks | High |
| Mac app string localization | 1-2 weeks | Medium |
| Testing and QA | 1 week | High |
| **Total** | **7-10 weeks** | - |

### Maintenance (Per Release Cycle)
- New string translation: 2-4 hours
- Testing: 1-2 hours
- **Total**: ~4-6 hours per release

## Recommended Action Plan

### Immediate (This PR)
✅ 1. Add Traditional Chinese README
✅ 2. Create this strategy document
✅ 3. Add Windows platform analysis document

### Short-term (Next 2-4 weeks)
📋 4. Translate core documentation (building, permissions, architecture)
📋 5. Translate CLI command documentation
📋 6. Add language selection links in main README

### Medium-term (1-3 months)
⏳ 7. Create .lproj structure and .strings files
⏳ 8. Refactor CLI error messages to use localization
⏳ 9. Refactor CLI command descriptions to use localization
⏳ 10. Add localization tests

### Long-term (3-6 months)
🔮 11. Full Mac app localization
🔮 12. Add other languages (Japanese, Korean, Simplified Chinese)
🔮 13. Create translation contribution guide
🔮 14. Automate translation workflow

## Alternative Approaches

### Minimal Approach (Documentation Only)
**Pros**:
- Zero code changes
- Quick implementation
- No maintenance burden

**Cons**:
- App itself still in English
- Inconsistent user experience

### Community-Driven Translation
**Tools**:
- Crowdin
- Weblate
- POEditor

**Pros**:
- Community involvement
- Support multiple languages
- Version control integration

**Cons**:
- Setup and maintenance needed
- Quality control challenges

## Conclusion

Traditional Chinese localization of Peekaboo is feasible but requires an organized approach:

1. **Immediate value**: Documentation translation (no code changes)
2. **Short-term investment**: CLI localization (moderate effort)
3. **Long-term goal**: Full app localization (significant investment)

**Recommended starting point**: Complete documentation localization, then evaluate further work based on community feedback and demand.
