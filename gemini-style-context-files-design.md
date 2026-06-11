# Gemini-Style Context Files for Pi

**Date**: 2026-05-08  
**Status**: Approved by user, ready for implementation  
**Goal**: Allow Pi to load multiple context files from config (mimicking Gemini CLI's `context.fileName`)

---

## Section 1: Architecture Overview

### What Changes

Modify `DefaultResourceLoader.loadProjectContextFiles()` in `resource-loader.js` to:

1. At each directory level during the walk, **check for config file** (`.pi/context-files.json` or `.gemini/settings.json`)
2. **If config exists** at that level → load only files from config (skip `loadContextFileFromDir()` for that level)
3. **If no config** → existing behavior (`loadContextFileFromDir()`)
4. Walk **continues past config directory** to root (doesn't stop)

### Data Flow

```
Directory Walk
    ↓
Check currentDir/.pi/context-files.json
    ↓ yes         ↓ no
Load config files    Check currentDir/.gemini/settings.json
    ↓ yes         ↓ no
Load config files    Use loadContextFileFromDir() (existing)
    ↓
Continue walk to parent dir
```

### Key Principle

Config **replaces AGENTS.md discovery at that level only** - it doesn't stop the walk or affect other levels.

---

## Section 2: Implementation Details

### Files to Modify

1. **`src/core/resource-loader.ts`** (source) OR **`dist/core/resource-loader.js`** (compiled)
   - Add `loadContextFilesFromConfig(dir)` method
   - Modify `loadProjectContextFiles()` while-loop

### New Method: `loadContextFilesFromConfig(dir)`

```typescript
function loadContextFilesFromConfig(dir) {
  const configPaths = [
    join(dir, ".pi", "context-files.json"),
    join(dir, ".gemini", "settings.json"),
  ];

  for (const configPath of configPaths) {
    if (!existsSync(configPath)) continue;

    try {
      const config = JSON.parse(readFileSync(configPath, "utf-8"));
      const fileList = config.contextFiles || config.context?.fileName || [];

      if (!Array.isArray(fileList) || fileList.length === 0) continue;

      const contextFiles = [];
      const seenPaths = new Set();

      for (const file of fileList) {
        const fullPath = isAbsolute(file) ? file : join(dir, file);
        if (!existsSync(fullPath) || seenPaths.has(fullPath)) continue;

        seenPaths.add(fullPath);
        contextFiles.push({
          path: fullPath,
          content: readFileSync(fullPath, "utf-8"),
        });
      }

      return contextFiles; // Return first found config
    } catch (e) {
      console.error(`Error reading config ${configPath}:`, e);
    }
  }

  return null; // No config found
}
```

### Modified While-Loop in `loadProjectContextFiles()`

```typescript
while (true) {
  // Check for config at this directory level FIRST
  const configFiles = loadContextFilesFromConfig(currentDir);

  if (configFiles && configFiles.length > 0) {
    // Config exists - use it, skip AGENTS.md discovery at this level
    for (const file of configFiles) {
      if (!seenPaths.has(file.path)) {
        ancestorContextFiles.unshift(file);
        seenPaths.add(file.path);
      }
    }
  } else {
    // No config - normal AGENTS.md discovery
    const contextFile = loadContextFileFromDir(currentDir);
    if (contextFile && !seenPaths.has(contextFile.path)) {
      ancestorContextFiles.unshift(contextFile);
      seenPaths.add(contextFile.path);
    }
  }

  // Continue walk to parent
  if (currentDir === root) break;
  const parentDir = resolve(currentDir, "..");
  if (parentDir === currentDir) break;
  currentDir = parentDir;
}
```

### Key Points

- `loadContextFilesFromConfig()` returns `null` if no config found
- Config **replaces** `loadContextFileFromDir()` at that level only
- Walk **continues to parent** regardless of config

---

## Section 3: Error Handling & Edge Cases

### What Can Go Wrong

| Scenario                                                            | Behavior                                                                                          |
| ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| **Config file is invalid JSON**                                     | Catch error, log to console, fall back to `loadContextFileFromDir()` (normal AGENTS.md discovery) |
| **File in config doesn't exist**                                    | Skip that file, continue loading others. Log warning.                                             |
| **Config has empty array**                                          | Treat as "no config found", fall back to AGENTS.md discovery                                      |
| **Duplicate files in config**                                       | Use `seenPaths` Set to deduplicate (same as existing behavior)                                    |
| **Both `.pi/context-files.json` AND `.gemini/settings.json` exist** | Use `.pi/context-files.json` first (Pi-native takes priority)                                     |
| **Config specifies AGENTS.md + other files**                        | Load all files from config (user explicitly included AGENTS.md)                                   |

### Error Handling Code (in `loadContextFilesFromConfig`)

```typescript
try {
  const config = JSON.parse(readFileSync(configPath, "utf-8"));
  const fileList = config.contextFiles || config.context?.fileName || [];

  if (!Array.isArray(fileList) || fileList.length === 0) {
    continue; // Try next config path
  }

  // ... load files
} catch (e) {
  console.error(`Error reading config ${configPath}:`, e);
  continue; // Try next config path or fall back to AGENTS.md
}
```

### What We Do NOT Change

- ✅ Directory walk logic (while loop structure unchanged)
- ✅ `loadContextFileFromDir()` function (still used when no config)
- ✅ Global `~/.pi/agent/AGENTS.md` loading (unchanged)
- ✅ `seenPaths` deduplication (reused for config files)

---

## Section 4: Testing Strategy

### Test Cases

| #   | Scenario                                                               | Expected Result                                                          |
| --- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 1   | **No config**                                                          | Normal directory walk (existing behavior unchanged)                      |
| 2   | **`.pi/context-files.json` exists** with valid files                   | Load ONLY those files, skip AGENTS.md at that level                      |
| 3   | **`.gemini/settings.json` exists** with `context.fileName`             | Load ONLY those files, skip AGENTS.md at that level                      |
| 4   | **Both configs exist**                                                 | Use `.pi/context-files.json` (Pi-native priority)                        |
| 5   | **Config has invalid JSON**                                            | Log error, fall back to AGENTS.md discovery                              |
| 6   | **File in config doesn't exist**                                       | Skip that file, load others, log warning                                 |
| 7   | **Monorepo: config at `/project/`, AGENTS.md at `/project/frontend/`** | Config used at `/project/`, walk continues, `/frontend/AGENTS.md` loaded |
| 8   | **Config file has relative paths**                                     | Resolve relative to config file's directory                              |
| 9   | **Config has duplicate files**                                         | Deduplicate via `seenPaths`                                              |
| 10  | **Config specifies AGENTS.md + other files**                           | Load all files from config (user explicitly included AGENTS.md)          |

### Test Implementation Approach

- Add unit tests to existing `resource-loader.test.ts` (or create it)
- Mock `existsSync`, `readFileSync` for config file scenarios
- Test `loadContextFilesFromConfig()` in isolation
- Test `loadProjectContextFiles()` integration with config

---

## Summary

**Goal**: Mimic Gemini CLI's `context.fileName` behavior in Pi

**Approach**: Modify `DefaultResourceLoader.loadProjectContextFiles()` to check for config at each directory level during walk

**Config formats supported**:

- `.pi/context-files.json` with `contextFiles` array (Pi-native)
- `.gemini/settings.json` with `context.fileName` array (Gemini-compatible)

**Key behavior**:

- Config **replaces** AGENTS.md discovery at that level only
- Walk **continues past config directory** to root
- Subdirectories still get normal AGENTS.md discovery
- Global `~/.pi/agent/AGENTS.md` unaffected

**Next steps**:

1. Fork `pi-coding-agent`
2. Implement changes per this design
3. Add tests
4. Submit PR upstream
5. Once merged, users can use `pi install` or upgrade to get feature
