# nut.js Custom Deployment - README for LLMs

## Critical Context (Post-January 2025)

**IMPORTANT:** nut.js was removed from the npm registry after January 2025. The developer moved to a subscription model but kept the project open-source. This means:
- nut.js is NOT available via `npm install @nut-tree/nut-js`
- nut.js must be built from source (GitHub repository)
- All deployment uses custom-built packages, not npm packages

## Project Structure Overview

```
\projects\
├── nut.js\                           # Development workspace (monorepo)
│   ├── core\
│   │   ├── nut.js\                   # Main nut.js package
│   │   ├── shared\                   # Shared utilities
│   │   ├── provider-interfaces\      # Provider interfaces
│   │   └── configs\                  # Build configurations
│   ├── providers\
│   │   ├── libnut\                   # Native module wrapper
│   │   └── clipboardy\               # Clipboard provider (named default-clipboard-provider)
│   └── examples\                     # Test examples
│
├── libnut-core\                      # Native C++ code (separate repo)
│   └── build\Release\libnut.node     # Compiled Windows binary
│
├── libnut-native\                    # Platform-specific binary wrapper
│   ├── package.json                  # Name: @nut-tree/libnut-native
│   ├── index.js                      # Platform detection loader
│   └── bin\
│       ├── win32\libnut.node         # Windows binary
│       ├── linux\libnut.node         # Linux binary (when built)
│       └── darwin\libnut.node        # macOS binary (future)
│
├── nut-js-deploy\                    # Deployment package
│   └── node_modules\@nut-tree\
│       ├── nut-js\                   # Main package (with modified package.json)
│       ├── libnut\                   # Native wrapper
│       ├── libnut-native\            # Platform binaries
│       ├── shared\                   # Shared utilities
│       ├── provider-interfaces\      # Interfaces
│       ├── configs\                  # Configs
│       └── default-clipboard-provider\ # Clipboard provider
│
└── [user-projects]\                  # User's automation projects
    └── package.json                  # References: "file:../nut-js-deploy/node_modules/@nut-tree/nut-js"
```

## Key Technical Details

### 1. Package Manager
- **pnpm@10** is used throughout
- The original repo referenced pnpm@8 but has been upgraded
- User removed the `packageManager` field from root package.json to use global pnpm@10

### 2. Native Module (libnut.node)
- Built from separate `libnut-core` repository (C++ code)
- Platform-specific: Windows, Linux, and macOS have different source files
- CMake build system with conditional compilation based on platform
- Output is always named `libnut.node` regardless of platform
- Location: `libnut-native/bin/{platform}/libnut.node`

### 3. Platform Detection
The `libnut-native` package automatically loads the correct binary:
```javascript
// libnut-native/index.js
const path = require('path');
const platform = process.platform;
const binaryPath = path.join(__dirname, 'bin', platform, 'libnut.node');
module.exports = require(binaryPath);
```

### 4. Critical Package Modifications
The `nut-js-deploy` folder contains packages with **modified package.json files**:

**All `workspace:*` references must be changed to `file:../[package-name]`**

Example - `nut-js/package.json`:
```json
{
  "dependencies": {
    "@nut-tree/libnut": "file:../libnut",
    "@nut-tree/shared": "file:../shared",
    "@nut-tree/provider-interfaces": "file:../provider-interfaces",
    "@nut-tree/default-clipboard-provider": "file:../default-clipboard-provider",
    "jimp": "0.22.10",
    "node-abort-controller": "3.1.1"
  }
}
```

**Critical:** The `@nut-tree/default-clipboard-provider` dependency was added manually (not in original source).

### 5. Folder vs Package Name Mismatch
- Folder name: `providers/clipboardy`
- Package name: `@nut-tree/default-clipboard-provider`
- In deployment, folder MUST be named `default-clipboard-provider` to match package name

### 6. Peer Dependencies
All `peerDependencies` were removed from packages in the deployment folder to avoid resolution conflicts.

## Development Workflow

### Building from Source
```bash
cd \projects\nut.js
pnpm install
pnpm run compile
```

### Testing
```bash
cd \projects\nut.js\examples\mouse-test
pnpm test
```

### Creating Deployment Package
1. Copy compiled packages to `nut-js-deploy/node_modules/@nut-tree/`
2. Manually edit all package.json files to replace `workspace:*` with `file:../[package-name]`
3. Add `@nut-tree/default-clipboard-provider` to nut-js dependencies
4. Remove all `peerDependencies` sections

### Required Packages in Deployment
- `@nut-tree/nut-js` (main package)
- `@nut-tree/libnut` (native wrapper)
- `@nut-tree/libnut-native` (platform binaries)
- `@nut-tree/shared`
- `@nut-tree/provider-interfaces`
- `@nut-tree/configs`
- `@nut-tree/default-clipboard-provider`

## User Project Setup

### New Project
```bash
cd \projects
mkdir my-automation-project
cd my-automation-project
pnpm init
```

### package.json
```json
{
  "name": "my-automation-project",
  "version": "1.0.0",
  "dependencies": {
    "@nut-tree/nut-js": "file:../nut-js-deploy/node_modules/@nut-tree/nut-js"
  }
}
```

### Install and Run
```bash
pnpm install
node index.js
```

## Cross-Platform Deployment

### Windows (Current)
- Binary located at: `libnut-native/bin/win32/libnut.node`
- Built from libnut-core using CMake on Windows

### Linux (Future)
- Will be located at: `libnut-native/bin/linux/libnut.node`
- Must be built on Linux machine from libnut-core
- Same deployment package works for both platforms (automatic platform detection)

### Deployment to Other Machines
1. Copy entire `nut-js-deploy` folder
2. Copy user project folder
3. Adjust file path in package.json if folder structure differs
4. Run `pnpm install`
5. Run the automation scripts

## Common Issues and Solutions

### Issue: "Cannot find module '@nut-tree/[package-name]'"
**Solution:** Check that:
1. Package exists in `nut-js-deploy/node_modules/@nut-tree/`
2. Folder name matches package name in package.json
3. `workspace:*` references are changed to `file:../[package-name]`
4. Package is listed in dependencies of the parent package

### Issue: Workspace dependency errors
**Solution:** All `workspace:*` must be converted to `file:../[package-name]` in deployment packages

### Issue: Peer dependency conflicts
**Solution:** Remove all `peerDependencies` sections from deployment packages

### Issue: Module not found during compilation
**Solution:** Check import statements - there was a typo in original source: `@nut-tree/core/shared` should be `@nut-tree/shared`

## Development vs Deployment

### For Active Development
Reference the workspace directly:
```json
"dependencies": {
  "@nut-tree/nut-js": "file:../nut.js/core/nut.js"
}
```
Pros: Immediate updates, no repacking needed
Cons: Requires full workspace

### For Deployment
Use the deployment package:
```json
"dependencies": {
  "@nut-tree/nut-js": "file:../nut-js-deploy/node_modules/@nut-tree/nut-js"
}
```
Pros: Self-contained, portable, true deployment test
Cons: Must rebuild deployment package after changes

## Important Notes for LLMs

1. **Never suggest** `npm install @nut-tree/nut-js` - it's not in npm registry
2. **pnpm is required** - this is a pnpm workspace project
3. **File references, not versions** - all dependencies use `file:` protocol in deployment
4. **Platform-specific binary** - libnut.node must be compiled for each OS
5. **Manual package.json edits** - workspace references must be manually converted for deployment
6. **Folder naming matters** - folder names must exactly match package names in deployment
7. **Default clipboard provider** - must be explicitly added to nut-js dependencies

## User's Environment
- **OS:** Windows 10 Pro
- **Node.js:** v22.14.0
- **pnpm:** 10.18.3
- **Primary workspace:** `\projects\`
- **Compiled Windows binary:** Available and working
- **Linux binary:** Not yet built (future work)

## Version Information
- nut.js version: 4.2.0
- libnut version: 4.2.0
- libnut-core version: 2.7.1

## Example Usage
```javascript
const { mouse, straightTo, Point } = require("@nut-tree/nut-js");

async function test() {
    console.log("Moving mouse to (100, 100)...");
    await mouse.move(straightTo(new Point(100, 100)));
    console.log("Done!");
}

test().catch(console.error);
```

---

**Last Updated:** Based on conversation from October 25, 2025
**Status:** Fully functional Windows deployment, Linux deployment pending binary compilation
