# Project Sicario

## Project Overview
Project Sicario (or Sicario Patcher) is an experimental tool for building and merging datatable mods for the game Project Wingman. It allows users to combine multiple data table edits into a single mod, enhancing compatibility. Under the hood, it utilizes HexPatch for applying byte-level edits to game files (`uasset` files).

Key features include:
- Templating for patches using the Liquid language (via the Fluid engine) to create dynamic replacements.
- Auto-length patching specifically for `uasset` files.
- Parameter support for user input in templates.
- A web-based Blazor User Interface (SicarioPatch.App).
- An optional CLI tool/merger (SicarioPatch.Loader) for packing the final mod.

**Core Technologies:**
- Language: C#
- Framework: .NET 8.0 (Migrated from .NET 6.0)
- Web UI: Blazor (ASP.NET Core)
- Build System: Cake (`build.cake`) and standard `dotnet` CLI.

## Architecture
The project is a .NET solution (`src/ProjectWingmanPatcher.sln`) containing several components:
- `SicarioPatch.App`: The Blazor-based web application providing the primary UI.
- `SicarioPatch.Loader`: A command-line tool or merger for processing and packing mods.
- `SicarioPatch.Core` & `SicarioPatch.Engine`: Core logic for parsing patches, loading options, and the HexPatch-based engine logic.
- `SicarioPatch.Components`: Shared Razor components for the Blazor UI.
- `SicarioPatch.Integration`: Logic for finding local game installations and unpacking/repacking archives.
- `SicarioPatch.Templating`: Integration with the Fluid templating engine.
- `UnPak.Core`: (Local source fork) Logic for parsing and unpacking `.pak` archives.

### Project Wingman 2.1.1A / Unreal Engine 4.27 Compatibility
The project has been updated to fully support UE 4.27 (Pak Version 11), resolving major compatibility issues with the latest game updates:
- **Pak Version 11 Format**: `UnPak.Core` correctly reads the UE 4.26+ `PathHashIndex` and `FullDirectoryIndex` compressed tree structures to parse file offsets.
- **Base Game Chunks**: The game now uses chunked `.pak` files (`pakchunk0-WindowsNoEditor.pak` instead of the old monolithic `ProjectWingman-WindowsNoEditor.pak`). `BuildCommand` and `GameArchiveFileService` have fallback logic to correctly identify `pakchunk0` to extract base game `uexp/uasset` files before patching.
- **Standard Mods vs. Datatable Mods**: The patcher is strictly a metadata-driven datatable merger. It identifies Sicario instructions via `_meta/sicario/*.json` or `.dtp` extensions. It ignores standard cooked asset replacement mods (like full `uasset` replacers without `.dtm` metadata) to prevent conflicts, though it does run a `SkinSlotLoader` to dynamically generate patches from custom skin `.uasset` paths.

## Building and Running

The project relies on a Cake build script (`build.cake`) for orchestration and the standard `dotnet` CLI for compilation.

### Prerequisites
- .NET 8.0 SDK

### Commands
- **Restore Dependencies:**
  ```powershell
  dotnet restore src/ProjectWingmanPatcher.sln
  ```
- **Build Solution:**
  ```powershell
  dotnet build src/ProjectWingmanPatcher.sln -c Release
  ```
- **Publish Standalone CLI:**
  ```powershell
  dotnet publish src/SicarioPatch.Loader/SicarioPatch.Loader.csproj -c Release
  ```
  *(Produces a self-contained executable in `src/SicarioPatch.Loader/bin/Release/net8.0/win-x64/publish/ProjectSicario.exe`)*
- **Run Tests:**
  ```powershell
  dotnet test src/ProjectWingmanPatcher.sln -c Release
  ```
- **Run the Blazor App:**
  ```powershell
  dotnet run --project src/SicarioPatch.App/SicarioPatch.App.csproj
  ```

Alternatively, you can use the Cake script to run the build pipeline (requires Cake tool installed):
```powershell
dotnet tool restore
dotnet cake build.cake --target=Default
```

## Development Conventions
- The codebase follows standard C# and .NET conventions.
- Patch definitions are authored in JSON format (`.dtm` files) and deserialized into the `WingmanMod` type.
- The project includes integration for deploying the merger CLI to Nexus Mods (using `unex`).
- For UI development, Blazor components are split across `SicarioPatch.App` and the shared `SicarioPatch.Components` library.
