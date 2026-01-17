# pi-coding-agent-termux

Termux port of [pi-coding-agent](https://github.com/badlogic/pi-mono) - a terminal-based coding agent with multi-model support.

**For user documentation, features, and usage instructions, please see the [upstream repository](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent).**

This repository maintains a Termux-compatible fork of pi-coding-agent. The port removes problematic dependencies and provides minimal Termux-specific replacements to enable running pi on Android devices via Termux.

## Installation on Termux

```bash
npm install -g @vaclav-synacek/pi-coding-agent-termux
```

Or build from source (on a development machine):

```bash
git clone https://github.com/VaclavSynacek/pi-coding-agent-termux.git
cd pi-coding-agent-termux
npm install
npm run build
cd packages/coding-agent
npm pack
# Transfer the .tgz file to Termux and install it
```

On Termux:
```bash
npm install -g /path/to/vaclav-synacek-pi-coding-agent-termux-0.45.7.tgz
```

**Note**: Building on Termux is not supported because the build tool (`tsgo`) doesn't have ARM64 binaries. Build on a regular machine and transfer the package, or install directly from npm.

For clipboard support, install Termux:API:

```bash
pkg install termux-api
```

## Functional Changes in this Port

This port modifies the upstream pi-coding-agent to work on Termux by:

### 1. Clipboard Integration
- **Changed**: `@mariozechner/clipboard` made optional (native bindings incompatible with Termux)
- **Added**: Termux clipboard support via `termux-clipboard-set` and `termux-clipboard-get` commands
- **Behavior**: Text clipboard works via Termux:API; image clipboard operations are disabled on Termux
- **Files modified**: 
  - `packages/coding-agent/src/utils/clipboard.ts`
  - `packages/coding-agent/src/utils/clipboard-image.ts`
  - `packages/coding-agent/package.json` (clipboard moved to optionalDependencies)

### 2. Image Processing
- **Changed**: Uses `wasm-vips` for image processing
- **Note**: Upstream v0.45.7+ migrated from `photon-node` to `wasm-vips` which is WebAssembly-based
- **Behavior**: Image resizing and conversion work correctly (wasm-vips is platform-independent)
- **Files modified**: None required - wasm-vips works on all platforms including Termux

### 3. Update Notification
- **Changed**: Version check and update notification system
- **Behavior**: Checks for updates against `@vaclav-synacek/pi-coding-agent-termux` on npm (instead of upstream package)
- **Display**: Shows correct command: `npm install -g @vaclav-synacek/pi-coding-agent-termux`
- **Files modified**: 
  - `packages/coding-agent/src/modes/interactive/interactive-mode.ts` (npm registry URL and install command)

### 4. Optional Dependencies
- **Changed**: `canvas` moved to optionalDependencies in `packages/ai/package.json`
- **Reason**: Canvas cannot build on Termux (requires pixman-1), but is only used for ai tests
- **Behavior**: npm install succeeds on Termux even if canvas build fails
- **Files modified**: `packages/ai/package.json`

### 5. TypeScript Target
- **Changed**: `tsconfig.base.json` target set to ES2024
- **Reason**: Support for regex `v` flag (required by some dependencies)
- **Files modified**: `tsconfig.base.json`

## Repository Structure & Maintenance

This repository follows a structured branching strategy to track upstream releases while maintaining Termux-specific patches:

### Branches

- **`master`** - The main Termux port branch
  - Contains Termux-specific modifications applied on top of upstream
  - Rebased onto new upstream versions when they are released
  - Force-pushed after each rebase (history is rewritten)
  
- **`upstream`** - Clean upstream tracking branch
  - Mirrors `https://github.com/badlogic/pi-mono.git` main branch
  - Used as the base for rebasing master
  - Never contains Termux-specific changes
  - Helps maintain transparency about what's changed in the port

### Tags

Tags follow the naming convention: `v{UPSTREAM_VERSION}-{PORT_REVISION}`

Examples:
- `v0.45.7-0` - First Termux port of upstream v0.45.7
- `v0.45.7-1` - Second Termux port of upstream v0.45.7 (port bugfix)
- `v0.46.0-0` - First Termux port of upstream v0.46.0

**Tags are immutable** - once published to npm, a tag is never changed or deleted. This preserves the complete version history.

### Remotes

- `origin` - This repository (`VaclavSynacek/pi-coding-agent-termux`)
- `upstream` - Upstream repository (`badlogic/pi-mono`)

## Maintenance Workflow

### When a New Upstream Version is Released

Example: Upstream releases `v0.46.0`

1. **Update upstream branch**
   ```bash
   git checkout upstream
   git fetch upstream
   git merge upstream/main  # or git reset --hard upstream/main
   git push origin upstream
   ```

2. **Rebase master onto new upstream version**
   ```bash
   git checkout master
   git rebase v0.46.0  # rebase onto the upstream tag
   ```

3. **Resolve conflicts**
   - Fix any rebase conflicts in Termux-specific patches
   - Pay special attention to files listed in "Functional Changes" section above
   - Ensure patches still apply cleanly and make sense

4. **Test locally on development machine**
   - Build the project: `npm install && npm run build`
   - Run checks: `npm run check`
   - Ensure everything compiles and passes linting

5. **Commit and push for testing**
   ```bash
   git push origin master --force-with-lease
   ```

6. **Test on actual Termux device**
   - Clone or pull the updated repository on Termux device
   - Navigate to the repository: `cd path/to/pi-coding-agent-termux`
   - Install dependencies: `npm install` (canvas build failure is expected and harmless)
   - Verify clipboard operations (with Termux:API installed)
   - Test that image operations work correctly
   - Check that all core features work
   - **If issues are found**: Fix them, commit, and repeat steps 4-6

7. **Update package.json version**
   ```bash
   cd packages/coding-agent
   # Update version to {UPSTREAM_VERSION}-{PORT_REVISION} (e.g., "0.46.0-0")
   ```

8. **Create immutable release tag**
   ```bash
   git tag -a v0.46.0-0 -m "Termux port of upstream v0.46.0"
   git push origin v0.46.0-0
   ```

9. **Publish to npm**
   ```bash
   cd packages/coding-agent
   npm publish
   ```

### When Port Needs a Bugfix (Without Upstream Change)

Example: Fix a bug in the Termux port of v0.46.0

1. **Make fixes on master branch**
   ```bash
   git checkout master
   # Make your fixes
   git commit -m "fix: describe the port-specific fix"
   ```

2. **Test locally and on Termux device**
   - Follow steps 4-6 from the upstream update workflow above
   - Repeat until fixes are confirmed working

3. **Update package.json version** (increment port revision)
   ```bash
   cd packages/coding-agent
   # Increment PORT_REVISION: "0.46.0-0" → "0.46.0-1"
   ```

4. **Create new port revision tag**
   ```bash
   git tag -a v0.46.0-1 -m "Termux port of upstream v0.46.0 (bugfix)"
   git push origin v0.46.0-1
   ```

5. **Publish to npm**
   ```bash
   cd packages/coding-agent
   npm publish
   ```

### Initial Setup (Already Done)

This was the initial setup of this repository:

1. Created new repository
2. Added upstream remote: `git remote add upstream https://github.com/badlogic/pi-mono.git`
3. Fetched upstream: `git fetch upstream --tags`
4. Created `upstream` branch: `git checkout -b upstream upstream/main`
5. Created `master` branch from upstream v0.45.7: `git checkout -b master v0.45.7`
6. Applied Termux patches as focused commits
7. Tagged first release: `v0.45.7-0`

## Termux Port Commits

The Termux port consists of these focused commits (applied on top of upstream):

1. **docs: Add comprehensive Termux port README** - Complete documentation and maintenance instructions
2. **feat: Update TypeScript target to ES2024** - Enable modern JavaScript features
3. **feat: Make @mariozechner/clipboard optional for Termux compatibility** - Optional require() for clipboard
4. **feat: Add Termux clipboard support** - Integration with termux-clipboard-set/get
5. **feat: Make @mariozechner/clipboard optional dependency** - Move to optionalDependencies in package.json
6. **feat: Update version check to use Termux port package** - Point to @vaclav-synacek/pi-coding-agent-termux
7. **feat: Update package.json for Termux port** - Package name and metadata
8. **feat: Make canvas optional dependency in ai package** - Allow clean build on Termux
9. **chore: Remove package-lock.json** - Platform-independent dependency resolution

## Commit Strategy for Rebasing

To make rebasing easier, Termux-specific changes are organized as **minimal, focused commits**:

- One commit per logical change
- Clear commit messages explaining why the change is needed for Termux
- Avoid mixing unrelated changes
- Keep patches as small as possible while maintaining functionality

This approach ensures that when rebasing onto new upstream versions, conflicts are:
- Easier to understand and resolve
- Less likely to occur
- Clearly attributable to specific Termux requirements

## Development Notes

### Building (Development Machine Required)

**Important**: Building requires `tsgo` which doesn't have Android ARM64 binaries. Build on a regular development machine (Linux/macOS/Windows).

This is a monorepo. To build all packages:

```bash
# From repository root (on development machine)
npm install
npm run build
```

To build only the coding-agent package:

```bash
cd packages/coding-agent
npm run build
```

**For Termux deployment**: Build on a development machine, then either:
1. Publish to npm and install from there
2. Use `npm pack` to create a .tgz file and transfer it to Termux

**Note**: The optional dependency `canvas` (in packages/ai) may fail to build due to missing native libraries (pixman-1). This is expected and harmless - npm will skip it and continue. Canvas is only used for ai package tests.

### Testing on Termux

1. Install Termux from F-Droid
2. Install dependencies:
   ```bash
   pkg update && pkg install git nodejs-lts
   pkg install termux-api  # For clipboard support
   ```
3. Install the package:
   ```bash
   npm install -g @vaclav-synacek/pi-coding-agent-termux
   ```
4. Run: `pi`

**For development/testing unreleased changes**: Build on a development machine, create a tarball with `npm pack`, transfer to Termux, and install with `npm install -g /path/to/package.tgz`.

### Comparison with Upstream

To see all Termux-specific changes:

```bash
git diff upstream master
```

To see changes in a specific file:

```bash
git diff upstream master packages/coding-agent/src/utils/clipboard.ts
```

## Publishing to npm

The package is published as `@vaclav-synacek/pi-coding-agent-termux` on npm.

**Publishing steps:**

```bash
cd packages/coding-agent
npm publish
```

**Key package.json fields:**
- `name`: `@vaclav-synacek/pi-coding-agent-termux` (different from upstream to avoid conflicts)
- `version`: Format `{UPSTREAM_VERSION}-{PORT_REVISION}` (e.g., `0.46.0-0`, `0.46.0-1`)
- `description`: "Termux port of pi-coding-agent - Coding agent CLI with read, bash, edit, write tools and session management"
- `repository`: Points to this repository (`VaclavSynacek/pi-coding-agent-termux`)
- `homepage`: https://github.com/VaclavSynacek/pi-coding-agent-termux#readme

## Why This Approach?

This maintenance strategy provides:

1. **Clean separation** - Clear distinction between upstream code and Termux patches
2. **Easy updates** - Rebasing makes it straightforward to adopt new upstream features
3. **Version history** - Immutable tags preserve every published version
4. **Transparency** - Easy to see exactly what's different from upstream
5. **Maintainability** - Future maintainers can understand the port structure
6. **No upstream dependency** - Port can continue indefinitely without upstream acceptance

## Contributing

When contributing Termux-specific changes:

1. Fork this repository
2. Create a feature branch from `master`
3. Make focused commits with clear messages
4. Test on actual Termux device
5. Submit pull request

For general pi-coding-agent features/bugs, contribute to the [upstream repository](https://github.com/badlogic/pi-mono) instead.

## License

Same as upstream: MIT License

See [LICENSE](LICENSE) file for details.
