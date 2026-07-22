# GitCritters

Pixel-art reactions for everyday git commands inside VS Code.

GitCritters listens to VS Code's built-in Git extension, detects common repository state changes, and opens a compact WebviewPanel with Canvas 2D animation, optional 8-bit sound, XP, levels, and unlock badges.

## Current MVP

- Detects `commit`, `push`, `pull`, `branch`, and merge-conflict activity through the VS Code Git API.
- Renders command-specific pixel-art scenes in a WebviewPanel.
- Plays optional square-wave sound effects with the Web Audio API.
- Tracks XP, levels, event counts, and unlocks in VS Code global state.
- Adds a status bar item for quick progress access.
- Provides a manual animation preview command for local testing.

## Project Layout

```text
GitCritters/
|-- README.md
|-- Essential_Files_Checklist.md
|-- Quick_Reference_Card.md
|-- Tech_Stack_Reference.md
|-- VS_Code_Extension_Initial_Setup_Guide.md
`-- gitcritters/
    |-- extension.js
    |-- package.json
    |-- README.md
    `-- test/
        `-- extension.test.js
```

## Commands

| Command | Description |
|---|---|
| `GitCritters: Show Panel` | Pick and preview an animation. |
| `GitCritters: View Progress` | Open XP, level, counts, and unlocks. |
| `GitCritters: Reset Progress` | Clear stored GitCritters progression data. |

## Settings

| Setting | Default | Description |
|---|---:|---|
| `gitcritters.soundEnabled` | `true` | Play retro sound effects with animations. |
| `gitcritters.animationSpeed` | `normal` | Animation speed: `slow`, `normal`, or `fast`. |
| `gitcritters.panelAutoCloseSeconds` | `6` | Seconds before animation panels close. Use `0` to keep them open. |
| `gitcritters.openPanelOnGitEvents` | `true` | Open an animation panel for detected git activity. |
| `gitcritters.notificationsEnabled` | `false` | Show a short XP status message for git events. |

## Development

Work from the generated extension folder:

```bash
cd gitcritters
npm install
npm run compile
npm run lint
npm test
```

On Windows PowerShell, use `npm.cmd` if script execution policy blocks the `npm.ps1` shim:

```bash
npm.cmd run compile
npm.cmd run lint
npm.cmd test
```

Press `F5` in VS Code from `gitcritters/` to launch an Extension Development Host.

## Notes

This scaffold is currently implemented in JavaScript because the generated project did not include TypeScript. The extension is structured so the Git event classifier and progression logic can be migrated to TypeScript later without changing the user-facing commands.
