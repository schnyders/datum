# Prompt History

## 2026-02-22 — Initial Memory Bank Creation
**Prompt**: "Please study the project and create a memory bank to track purpose and progress in the project."
**Actions Taken**:
- Read README.md, CHANGELOG.md, module manifest, module root, build.yaml
- Read all class files (DatumProvider, FileProvider, Node)
- Read key public functions (Resolve-Datum, Merge-Datum, New-DatumStructure, Get-DatumRsop)
- Read Resolve-NodeProperty scripts-to-process
- Read RequiredModules.psd1 for dependency list
- Reviewed git log (last 15 commits) and git status
- Created memory-bank/ directory with 6 core files:
  - projectbrief.md — Project scope and goals
  - productContext.md — Why it exists and user workflows
  - systemPatterns.md — Architecture and design patterns
  - techContext.md — Technology stack and development setup
  - activeContext.md — Current work focus and recent changes
  - progress.md — Feature status and version history
  - promptHistory.md — This file

## 2026-02-22 — Test Analysis & Build Run
**Prompt**: "Analyze the tests folder, run the build, analyze output, update memory bank."
**Actions Taken**:
- Read all 9 integration test files: Merge, Override, Rsop, RsopProtectedDatum, RsopWithInvokCommandHandler, Demo3, Copy-Object, Expand-RsopHashtable, Get-RsopValueString
- Identified 7 test data hierarchies in tests/Integration/assets/
- Documented test patterns: Pester 5 syntax, data-driven, InModuleScope for private functions, real hierarchies (not mocks)
- Ran build in separate process (`Start-Process pwsh`) to avoid VS Code hanging (previous attempts caused VS Code to become unresponsive)
- **Build result**: Succeeded — 16 tasks, 0 errors, 0 warnings, 33 seconds
- **Test result**: 158 passed, 0 failed, 3 skipped
- **Skipped tests**: 3 tests in RsopWithInvokCommandHandler.tests.ps1 due to known merge logic bug (Ethernet 3 Gateway, DnsServer, Interface Count for DSCFile01)
- **Expected warnings**: ProtectedData handler errors (encrypted with different key = expected behavior, tested explicitly)
- Updated memory bank: systemPatterns.md (full testing architecture section), progress.md (build results, known bug), activeContext.md (next steps, important patterns)

## 2026-02-23 — Docs Code Sample Audit & Fix
**Prompt**: "Go through the docs and verify if the provided code samples really work. The AllNodes iteration pattern from docs failed against DscWorkshopConfigData."
**Root Cause Found**:
- AllNodes iteration pattern `$Datum.AllNodes.psobject.Properties | ForEach-Object { (@{} + $Datum.AllNodes.($_.Name)) }` assumes **flat** AllNodes directory (node files directly under AllNodes/)
- DscWorkshop reference implementation uses **nested** AllNodes: AllNodes/Dev/DSCFile01.yml, AllNodes/Prod/..., AllNodes/Test/...
- With nested AllNodes, `$Datum.AllNodes.psobject.Properties` returns environment sub-providers (FileProvider objects), not node data hashtables
- `(@{} + $FileProviderObject)` throws: "A hash table can only be added to another hash table"
- The Datum.yml examples in the docs show `ResolutionPrecedence: AllNodes\$($Node.Environment)\$($Node.NodeName)` which implies nested structure, but the iteration code assumed flat structure — an inconsistency

**Broken code samples fixed** (4 locations):
1. README.md — RSOP section: Added both flat and nested patterns with explanation
2. docs/RSOP.md — Basic Usage: Added both patterns with error message note
3. docs/RSOP.md — Comparing Nodes: Updated to nested pattern with comment
4. docs/AboutDatum.md — Example 2: Split into Example 2 (flat) and Example 2b (nested)

**Other code samples audited** (all OK):
- docs/CmdletReference.md — All parameter tables and examples correct
- docs/DatumYml.md — Configuration examples correct, file layout shows nested structure
- docs/DatumHandlers.md — Handler examples correct
- docs/Merging.md — Merge strategy examples correct
- docs/CodeLayers.md — Conceptual examples correct
- README.md — Getting started, lookup examples, handler examples all correct

**Verification**:
- Nested pattern tested against DscWorkshopConfigData: 9 nodes found (3 per environment)
- Flat pattern tested against MergeTestData: 3 nodes found
- Existing tests still pass: Rsop.tests.ps1 (23 passed), Demo3.tests.ps1 (3 passed)

## 2026-02-23 — RemoveSource / IncludeSource Documentation Fix
**Prompt**: "When calling Get-DatumRsop -IncludeSource -RemoveSource the source info is still visible. Examine the output objects, update docs only."
**Root Cause Found**:
- `Get-DatumRsop` uses `if ($IncludeSource) / elseif ($RemoveSource) / else` branching
- When BOTH switches are specified, `-IncludeSource` always wins (the `elseif` is never reached)
- `-RemoveSource` is designed to be used ALONE — it strips `__File` NoteProperties from output values via `.psobject.BaseObject`
- The doc example showed `-IncludeSource -RemoveSource` together as if they compose; they do not

**Documentation bugs fixed** (3 locations):
1. **docs/RSOP.md — "Example Output with Source"**: Replaced fake `__source: Roles/Role1.yml` key example with actual right-aligned inline annotations matching real output; documented `$env:DatumRsopIndentation`
2. **docs/RSOP.md — "Removing Source Data"**: Rewritten as "Removing Source Metadata" — clarified mutual exclusivity, explained `__File` NoteProperties, fixed example to `-RemoveSource` alone
3. **docs/RSOP.md — Troubleshooting**: Replaced nonexistent `$rsop.SomeKey.__source` with `ConvertTo-Yaml` inspection guidance
4. **docs/CmdletReference.md**: Updated `RemoveSource` parameter description to explain NoteProperty stripping and mutual exclusivity with IncludeSource

**No code changes** — docs-only update as requested.
