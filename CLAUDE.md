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
| `.claude/worktrees/ue4-port/` | Active development worktree |
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

# Recreate if broken
cmd //c "mklink /J F:\UE4\Projects\Wiz98\Plugins\NodeToCode C:\NodeToCodeUE4\.claude\worktrees\ue4-port"
```

Junctions break if the target directory is deleted (e.g., worktree pruning). Always commit before ending a session.

## Architecture

Pipeline: **Collect -> Translate -> Serialize -> LLM -> Parse -> Display**

- `FN2CNodeCollector` - Extracts UK2Nodes from Blueprint graphs
- `FN2CNodeTranslator` - Converts to FN2CBlueprint via 12 specialized processors
- `FN2CSerializer` - JSON serialization (compact, token-efficient)
- `UN2CLLMModule` - Orchestrates LLM request/response (6 providers)
- `SN2CEditorWindow` - Results display (currently stub UI)

See `docs/PROJECT_OVERVIEW.md` for full architecture details.

## UE4.27 Port Rules

When writing new code or modifying existing code, follow these UE4.27 constraints:

- Use `FEditorStyle` not `FAppStyle`
- Use `FSyntaxTokenizer` not `ISyntaxTokenizer`
- Use `ESPMode::ThreadSafe` explicitly on `TSharedRef<IHttpRequest>`
- Use `GetPathName()` not `GetStructPathName()` on UScriptStruct
- Use `operator->` on TScriptInterface, not `.GetInterface()->`
- `TryGetNumberField` expects `double&` not `float&`
- No `PC_Double` or `PC_Real` pin types (only `PC_Float`)
- No `UK2Node_ExternalGraphInterface` or `UK2Node_PromotableOperator`
- `SMultiLineEditableText` lacks: `DeleteSelectedText`, `GetCursorLocation`, `SelectText`, `GetSelection`
- Include explicitly: `Misc/FileHelper.h`, `Internationalization/Regex.h`, `Policies/CondensedJsonPrintPolicy.h`, `Widgets/Layout/SGridPanel.h`
- Asset editor events: use `UAssetEditorSubsystem` via `GEditor->GetEditorSubsystem<>()`, not `FAssetEditorManager`

See `docs/UE4_PORT_GUIDE.md` for the complete API change reference.

## Remaining Work

- **Slate UI rebuild**: `SN2CEditorWindow` is a placeholder. Needs full translation display UI in pure Slate.
- **Toolbar icon**: Registered but may not render on all configurations. Investigate if not visible.
- **Code editor stubs**: `SN2CCodeEditor` has stubbed cursor/selection methods for 4.27 compatibility.

## Documentation

- `docs/PROJECT_OVERVIEW.md` - Architecture, source layout, LLM providers, data flow
- `docs/UE4_PORT_GUIDE.md` - Every UE5->UE4.27 API change with tables
- `docs/DEVELOPMENT_SETUP.md` - Junction setup, build commands, agent instructions
