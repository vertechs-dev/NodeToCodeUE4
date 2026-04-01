# NodeToCode UE4.27 Port

UE4.27.2 port of the NodeToCode plugin (originally UE5.3+). Translates Blueprint graphs to code via LLM providers.

## Project Context

- **Engine:** Unreal Engine 4.27.2 (installed at `F:\UE4\UE_4.27`)
- **Test project:** `F:\UE4\Projects\Wiz98\Wiz98.uproject`
- **Toolchain:** Visual Studio 2019 Community
- **Platform:** Windows 10

## Key Paths

| Path | Purpose |
|------|---------|
| `C:\NodeToCodeUE4` | Git repo root |
| `.claude/worktrees/<worktree>/` | Active development worktree |
| `F:\UE4\Projects\Wiz98\Plugins\NodeToCode` | Junction into worktree |

## Build Commands

Always close the UE4 editor before building. Live Coding blocks CLI builds.

```bash
# Build
"F:/UE4/UE_4.27/Engine/Build/BatchFiles/Build.bat" Wiz98Editor Win64 Development -Project="F:/UE4/Projects/Wiz98/Wiz98.uproject" -WaitMutex -FromMsBuild

# Generate project files (after adding/removing source files or changing Build.cs)
"F:/UE4/UE_4.27/Engine/Binaries/DotNET/UnrealBuildTool.exe" -projectfiles -project="F:/UE4/Projects/Wiz98/Wiz98.uproject" -game -engine -progress
```

## Junction Setup

The plugin source is linked into the test project via a Windows junction. Check it before building:

```bash
# Verify
ls "F:/UE4/Projects/Wiz98/Plugins/NodeToCode/NodeToCode.uplugin"

# Recreate if broken (replace <worktree> with actual worktree directory name)
cmd //c "mklink /J F:\UE4\Projects\Wiz98\Plugins\NodeToCode C:\NodeToCodeUE4\.claude\worktrees\<worktree>"
```

Junctions break if the target directory is deleted (e.g., worktree pruning). Always commit before ending a session.

## Architecture

Pipeline: **Collect -> Translate -> Serialize -> LLM -> Parse -> Display**

- `FN2CNodeCollector` - Extracts UK2Nodes from Blueprint graphs
- `FN2CNodeTranslator` - Converts to FN2CBlueprint via 12 specialized processors
- `FN2CSerializer` - JSON serialization (compact, token-efficient)
- `UN2CLLMModule` - Orchestrates LLM request/response (6 providers)
- `SN2CEditorWindow` - Results display (full Slate UI with code editors, loading states, graph switching)

See `docs/PROJECT_OVERVIEW.md` for full architecture details.

## UE4.27 Port Rules

When writing new code or modifying existing code, follow these UE4.27 constraints:

- Use `FEditorStyle` not `FAppStyle`
- Use `FSyntaxTokenizer` not `ISyntaxTokenizer`
- Use `ESPMode::ThreadSafe` explicitly on `TSharedRef<IHttpRequest>`
- Use `GetPathName()` not `GetStructPathName()` on UScriptStruct
- Use `operator->` on TScriptInterface, not `.GetInterface()->`
- `DECLARE_DYNAMIC_MULTICAST_DELEGATE` requires UObject+UFUNCTION for binding; use parallel native `DECLARE_MULTICAST_DELEGATE` for Slate widget binding via `AddRaw()`
- `TryGetNumberField` expects `double&` not `float&`
- No `PC_Double` or `PC_Real` pin types (only `PC_Float`)
- No `UK2Node_ExternalGraphInterface` or `UK2Node_PromotableOperator`
- `SMultiLineEditableText` lacks: `DeleteSelectedText`, `GetCursorLocation`, `SelectText`, `GetSelection`
- Include explicitly: `Misc/FileHelper.h`, `Internationalization/Regex.h`, `Policies/CondensedJsonPrintPolicy.h`, `Widgets/Layout/SGridPanel.h`
- Asset editor events: use `UAssetEditorSubsystem` via `GEditor->GetEditorSubsystem<>()`, not `FAssetEditorManager`

See `docs/UE4_PORT_GUIDE.md` for the complete API change reference.

## Remaining Work

- **Toolbar icon**: Registered but may not render on all configurations. Investigate if not visible.
- **Code editor stubs**: `SN2CCodeEditor` has stubbed cursor/selection methods for 4.27 compatibility.
- **Settings location**: Plugin settings are in **Project Settings > Plugins > Node to Code** (not Editor Preferences), because `UDeveloperSettings` with `Config=NodeToCode` defaults container to "Project".

## Completed Work

- **Slate UI rebuild**: `SN2CEditorWindow` fully rebuilt with SWidgetSwitcher (welcome/loading/results/error panels), dual SN2CCodeEditor widgets, SComboBox graph selector, copy/open-folder buttons, and native delegate subscriptions.
- **Native delegates**: Added `FOnTranslationRequestSentNative` and `FOnTranslationResponseReceivedNative` alongside existing dynamic delegates in `UN2CLLMModule` for Slate widget binding.
- **Anthropic model updates**: Updated to Claude Opus 4.6, Sonnet 4.6, Haiku 4.5 with correct API model IDs (no date suffix for 4.6 models). Updated pricing for Opus 4.6 ($5/$25 per MTok).
- **Max tokens**: Increased Anthropic max_tokens from 8192 to 16384 to prevent truncated responses on larger Blueprints.

## Documentation

- `docs/PROJECT_OVERVIEW.md` - Architecture, source layout, LLM providers, data flow
- `docs/UE4_PORT_GUIDE.md` - Every UE5->UE4.27 API change with tables
- `docs/DEVELOPMENT_SETUP.md` - Junction setup, build commands, agent instructions
