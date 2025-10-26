## User Project Setup
Copy entire `nut-js-deploy` folder to a place in your source folder structure.
### New Project
```bash
cd \projects
mkdir my-automation-project
cd my-automation-project
pnpm init
```

### package.json
Adjust the file path to nut-js to align with your folder structure.
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
