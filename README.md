# pi-coding-agent-termux

Termux port of [pi-coding-agent](https://github.com/badlogic/pi-mono) - a terminal-based coding agent with multi-model support.

**For user documentation, features, and usage instructions, please see the [upstream repository](https://github.com/badlogic/pi-mono/tree/main/packages/coding-agent).**

This repository maintains a Termux-compatible fork of pi-coding-agent. The port removes problematic dependencies and provides minimal Termux-specific replacements to enable running pi on Android devices via Termux.

## Installation on Termux

```bash
npm install -g @vaclav-synacek/pi-coding-agent-termux
```

Or build from source:

```bash
pkg install git nodejs-lts
git clone https://github.com/VaclavSynacek/pi-coding-agent-termux.git
cd pi-coding-agent-termux
npm install
npm run build
```

For clipboard support, install Termux:API:

```bash
pkg install termux-api
```

## Changes in this Port

This port modifies the upstream pi-coding-agent to work on Termux by:

### 1. Clipboard Integration
- **Changed**: `@mariozechner/clipboard` made optional (native bindings incompatible with Termux)
- **Added**: Termux clipboard support via `termux-clipboard-set` and `termux-clipboard-get` commands
- **Behavior**: Text clipboard works via Termux:API; image clipboard operations are disabled on Termux
- **Files modified**: 
  - `packages/coding-agent/src/utils/clipboard.ts`
  - `packages/coding-agent/src/utils/clipboard-image.ts`
  - `packages/coding-agent/package.json` (clipboard moved to optionalDependencies)

### 2. Update Notification
- **Changed**: Version check and update notification system
- **Behavior**: Checks for updates against `@vaclav-synacek/pi-coding-agent-termux` on npm (instead of upstream package)
- **Display**: Shows correct command: `npm install -g @vaclav-synacek/pi-coding-agent-termux`
- **Files modified**: 
  - `packages/coding-agent/src/modes/interactive/interactive-mode.ts` (npm registry URL and install command)

### 3. Optional Dependencies
- **Changed**: `canvas` moved to optionalDependencies in `packages/ai/package.json`
- **Reason**: Canvas cannot build on Termux (requires pixman-1), but is only used for ai tests
- **Behavior**: npm install succeeds on Termux even if canvas build fails
- **Files modified**: `packages/ai/package.json`

### 4. TypeScript Target
- **Changed**: `tsconfig.base.json` target set to ES2024
- **Reason**: Support for regex `v` flag (required by some dependencies)
- **Files modified**: `tsconfig.base.json`

### 5. GitHub Workflows
- **Replaced**: All GitHub Actions workflows (`.github/workflows/`)
- **Reason**: Building tailored to the termux port
- **Files modified**: `.github/workflow/*`

### 6. Top level README
- **Replaced**: full content of the readme to termux specific readme with only
  a link/reference to the general upstream readme
- **Reason**: Serves as documentation of the termux specific porting process
  for agents and humans
- **Files modified**: `README.md`

### 7. Agent README
- **Replaced**: full content of the readme for the npm package replaced with
  termux specific content. The content is fixed and minimal.
- **Reason**: this is the readme that gets displayed on npm - needs to be
  termux specific with reference to upstream. But it has to contain as little
  details as possible to minimize need for double updates when content in top
  level readme is updated.
- **Files modified**: `packages/coding-agent/README.md`

### 8. Build System Limitation
- **Limitation**: tsgo (native TypeScript compiler) doesn't support Android/Termux
- **Impact**: Builds must be done on Linux/macOS/Windows, not on Termux
- **Reason**: tsgo only provides binaries for standard platforms, not android-arm64
- **Workaround**: CI/CD builds on Linux and publishes to npm

## Repository Structure & Maintenance

This repository follows a structured branching strategy to track upstream releases while maintaining Termux-specific patches:

### Branches

- **`master`** - The main Termux port branch
  - Contains Termux-specific modifications applied on top of upstream
  - Rebased onto new upstream versions when they are released
  - Force-pushed after each rebase (history is rewritten)
  - Content of all commits when done should exactly match chapter "Content in
    this Port"
  
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

**Tags are immutable** - once published to npm and to github, a tag is never changed or deleted. This preserves the complete version history. Both publish to npm and push tag to github should happen at the same time in the workflow, once the testing has been done and the build is ready.

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

4. **Commit and push for testing**
   - commit and force push the master, but do not add tags yet
   ```bash
   git push origin master --force-with-lease
   ```

5. **Test on Termux machine (do NOT build)**
   - **Warning**: Building will fail on Termux due to tsgo. Only `npm install` and test.
   - If previous steps were done on non-termux machine, move to termux now and
     git clone or git pull the master. If we are already on termux machine just continue
     here.
   - Install dependencies: `npm install` (canvas will fail to build - this is expected)
   - Verify clipboard operations (with Termux:API installed)
   - Test that image operations work correctly
   - Check that all core features work
   - **If issues are found**: Fix them, commit, and repeat steps 3-5
   - **If everything is messed up and you cannot backtrack**: revert master to previous version tag and start over from the beginning.

**Do following steps only if all previous done and everything seems to work and
is tested**

7. **Update package.json version**
   ```bash
   cd packages/coding-agent
   # Update version to {UPSTREAM_VERSION}-{PORT_REVISION} (e.g., "0.46.0-0")
   ```
   - commit and force push again

8. **Create immutable release tag**
   ```bash
   git tag -a v0.46.0-0 -m "Termux port of upstream v0.46.0"
   git push origin v0.46.0-0
   ```

9. **Check it was published to npm**
    - when new tag is pushed to github, the CI/CD should build and push it to
      npm
    - check on github that the CI/CD pipeline has not failed
    - check on npm that the latest version is the expected new one
    - if failed, inspect the CI/CD for errors, fix them by manipulating history
      and fixing them in the commit that rewrote CI/CD workflows
    - repeat from step 7

## Commit Strategy for Rebasing

To make rebasing easier, Termux-specific changes should be organized as **minimal, focused commits**:

- One commit per logical change (e.g., "Make clipboard package optional", "Add Termux clipboard support")
- Clear commit messages explaining why the change is needed for Termux
- Avoid mixing unrelated changes
- Keep patches as small as possible while maintaining functionality

This approach ensures that when rebasing onto new upstream versions, conflicts are:
- Easier to understand and resolve
- Less likely to occur
- Clearly attributable to specific Termux requirements
- easy to rebase and check against this README for completeness

## Development Notes

### Building

**Note:** Building requires Linux/macOS/Windows. Termux is not supported due to tsgo platform limitations. On Termux, use pre-built packages from npm or test from CI/CD builds.

This is a monorepo. To build all packages:

```bash
# From repository root
npm install
npm run build
```

To build only the coding-agent package:

```bash
cd packages/coding-agent
npm run build
```

### Testing on Termux

1. Install Termux from F-Droid
2. Install dependencies:
   ```bash
   pkg update && pkg install git nodejs-lts
   pkg install termux-api  # For clipboard support
   ```
3. Clone and build this repository
4. Run: `./packages/coding-agent/dist/cli.js`

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
3. Make focused commits with clear messages - ideally by manipulating existing
   commits if it is altering one of the listed termux specific differneces
4. Test on actual Termux device
5. Submit pull request

For general pi-coding-agent features/bugs, contribute to the [upstream repository](https://github.com/badlogic/pi-mono) instead.

## License

Same as upstream: MIT License

See [LICENSE](LICENSE) file for details.
