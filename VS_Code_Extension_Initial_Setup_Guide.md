# Building a VS Code Extension: Complete Initial Setup Guide
## For GitCritters (Pixel-Art Git Animations)

---

## 📋 Phase 0: Prerequisites & Environment

Before you start, ensure you have:
- **Node.js** v14+ (`node --version`)
- **npm** v6+ (`npm --version`)
- **VS Code** (latest version)
- **Git** installed and configured

Verify everything:
```bash
node --version   # v18+ recommended
npm --version    # v8+ recommended
git --version    # v2.30+
code --version   # VS Code version
```

---

## 🔧 Tech Stack Overview

| Technology | Purpose | Why? |
|---|---|---|
| **TypeScript** | Extension logic | Type-safe, prevents runtime bugs in core logic |
| **Node.js** | Runtime environment | Powers the extension host |
| **VS Code Extension API** | VS Code integration | Hooks into Git API, WebviewPanels, file system |
| **HTML/Canvas** | Animation rendering | Canvas 2D for pixel art, lightweight |
| **Web Audio API** | 8-bit sound | No external libraries needed |
| **Webpack/esbuild** | Code bundling | Minifies and tree-shakes for fast startup |

---

## 🎯 Step 1: Scaffold Your Extension Project

### Option A: Using the VS Code Generator (Recommended)

```bash
# Install the Yeoman generator globally
npm install -g yo generator-code

# Run the generator
yo code

# When prompted:
# → Extension name: GitCritters
# → Extension identifier: gitcritters
# → Description: "Pixel-art animations for every git command"
# → Initialize a git repository: Yes
# → Bundle the extension: Yes
# → Which bundler: webpack
# → Which package manager: npm
# → Enable TypeScript: Yes (strongly recommended)
```

This scaffolds the directory structure automatically.

### Option B: Manual Setup (Advanced)

```bash
mkdir gitcritters && cd gitcritters
npm init -y

# Install core dependencies
npm install --save-dev vscode yo generator-code
npm install --save-dev typescript @types/node @types/vscode
npm install --save-dev webpack webpack-cli ts-loader
```

---

## 📁 Project Structure (After Scaffolding)

```
gitcritters/
├── .vscode/
│   ├── launch.json         # Debug configuration
│   ├── settings.json       # Workspace settings
│   └── tasks.json          # Build tasks
│
├── src/
│   ├── extension.ts        # Main extension file (entry point)
│   ├── gitDetector.ts      # Git command detection logic
│   ├── animationManager.ts # WebviewPanel lifecycle
│   └── gameEngine.ts       # Mini-game logic
│
├── media/
│   ├── sprites/            # Aseprite-exported PNG files
│   ├── sounds/             # 8-bit audio (MP3/OGG)
│   └── panel.html          # WebView template for animations
│
├── out/                    # Compiled JavaScript (auto-generated)
├── node_modules/
├── package.json            # Dependencies and scripts
├── tsconfig.json           # TypeScript configuration
├── webpack.config.js       # Bundler config
├── .vscodeignore           # Files to exclude from packaging
└── README.md
```

---

## ⚙️ Step 2: Configure package.json

Update `package.json` with the extension metadata and commands:

```json
{
  "name": "gitcritters",
  "displayName": "GitCritters",
  "description": "Pixel-art animations for every git command",
  "version": "0.0.1",
  "author": "Your Name",
  "license": "MIT",
  "repository": "https://github.com/yourusername/gitcritters",
  "engines": {
    "vscode": "^1.70.0"
  },
  "categories": [
    "Other",
    "Visualization"
  ],
  "activationEvents": [
    "onStartupFinished"
  ],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "gitcritters.showPanel",
        "title": "GitCritters: Show Panel"
      },
      {
        "command": "gitcritters.viewProgress",
        "title": "GitCritters: View Progress"
      }
    ],
    "configuration": {
      "title": "GitCritters",
      "properties": {
        "gitcritters.soundEnabled": {
          "type": "boolean",
          "default": true,
          "description": "Enable/disable sound effects"
        },
        "gitcritters.animationSpeed": {
          "type": "string",
          "enum": ["slow", "normal", "fast"],
          "default": "normal",
          "description": "Animation speed"
        }
      }
    }
  },
  "scripts": {
    "vscode:prepublish": "webpack --mode production",
    "webpack": "webpack --mode development",
    "webpack-watch": "webpack --mode development --watch",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./",
    "pretest": "npm run compile && npm run lint",
    "lint": "eslint src --ext ts",
    "test": "node ./out/test/runTest.js"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/vscode": "^1.70.0",
    "@typescript-eslint/eslint-plugin": "^5.0.0",
    "@typescript-eslint/parser": "^5.0.0",
    "eslint": "^8.0.0",
    "ts-loader": "^9.0.0",
    "typescript": "^4.7.0",
    "webpack": "^5.75.0",
    "webpack-cli": "^5.0.0"
  }
}
```

---

## ✍️ Step 3: Write Your First Extension (entry.ts)

This is the **activation entry point** — runs when VS Code loads your extension.

```typescript
// src/extension.ts
import * as vscode from 'vscode';

// Store the extension context
let context: vscode.ExtensionContext;

export function activate(extensionContext: vscode.ExtensionContext) {
  context = extensionContext;
  
  console.log('GitCritters activated!');

  // Register command: Show Panel
  let showPanelCommand = vscode.commands.registerCommand(
    'gitcritters.showPanel',
    () => {
      showAnimationPanel('push');
    }
  );
  context.subscriptions.push(showPanelCommand);

  // Hook into Git API
  setupGitDetection();
}

export function deactivate() {
  console.log('GitCritters deactivated');
}

// ────────────────────────────────────────────────────────────
// Git Detection: Watch for repository state changes
// ────────────────────────────────────────────────────────────

function setupGitDetection() {
  const gitExtension = vscode.extensions.getExtension('vscode.git');
  
  if (!gitExtension) {
    vscode.window.showErrorMessage(
      'Git extension not found. Please install VS Code\'s built-in Git extension.'
    );
    return;
  }

  if (!gitExtension.isActive) {
    gitExtension.activate().then(onGitApiReady, onGitApiError);
  } else {
    onGitApiReady(gitExtension.exports);
  }
}

function onGitApiReady(gitApi: any) {
  const git = gitApi.getAPI(1);

  // Monitor each repository
  git.repositories.forEach((repo: any) => {
    monitorRepository(repo);
  });

  // When new repositories are opened
  git.onDidOpenRepository((repo: any) => {
    monitorRepository(repo);
  });
}

function monitorRepository(repo: any) {
  let lastAhead = repo.state.HEAD?.ahead ?? 0;

  // Watch for state changes
  repo.state.onDidChange(() => {
    const currentAhead = repo.state.HEAD?.ahead ?? 0;
    const branchName = repo.state.HEAD?.name ?? 'unknown';

    // Detect push: ahead count drops to 0
    if (lastAhead > 0 && currentAhead === 0) {
      showAnimationPanel('push', branchName);
    }

    // Detect pull: ahead count decreases
    if (currentAhead < lastAhead && currentAhead >= 0) {
      showAnimationPanel('pull', branchName);
    }

    lastAhead = currentAhead;
  });
}

function onGitApiError(err: any) {
  vscode.window.showErrorMessage(
    `GitCritters error: Could not access Git API - ${err.message}`
  );
}

// ────────────────────────────────────────────────────────────
// WebviewPanel: Display the animation
// ────────────────────────────────────────────────────────────

function showAnimationPanel(
  command: string,
  branchName: string = 'main'
) {
  const panel = vscode.window.createWebviewPanel(
    'gitcritters', // Panel ID
    'GitCritters', // Title
    vscode.ViewColumn.Beside, // Show beside the editor
    {
      enableScripts: true,
      localResourceRoots: [
        vscode.Uri.joinPath(context.extensionUri, 'media')
      ]
    }
  );

  // Generate a unique nonce for CSP
  const nonce = getNonce();

  // Load HTML content
  panel.webview.html = getWebviewContent(
    panel.webview,
    nonce,
    command,
    branchName
  );

  // Auto-close after 6 seconds
  setTimeout(() => {
    if (panel) panel.dispose();
  }, 6000);

  // Message listener (for game interactions)
  panel.webview.onDidReceiveMessage((message) => {
    switch (message.command) {
      case 'log':
        console.log(message.text);
        break;
      case 'gameScore':
        vscode.window.showInformationMessage(`Score: ${message.score}`);
        break;
    }
  });
}

// ────────────────────────────────────────────────────────────
// WebView HTML: Animation Rendering
// ────────────────────────────────────────────────────────────

function getWebviewContent(
  webview: vscode.Webview,
  nonce: string,
  command: string,
  branchName: string
): string {
  return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <meta http-equiv="Content-Security-Policy" 
            content="default-src 'none'; 
                     img-src ${webview.cspSource}; 
                     script-src 'nonce-${nonce}'; 
                     style-src 'nonce-${nonce}';">
      <title>GitCritters</title>
    </head>
    <body style="margin: 0; padding: 0; background: #1e1e1e; overflow: hidden;">
      <canvas id="animationCanvas" width="340" height="160" 
              style="image-rendering: pixelated; display: block; margin: auto;">
      </canvas>

      <script nonce="${nonce}">
        const vscode = acquireVsCodeApi();
        const canvas = document.getElementById('animationCanvas');
        const ctx = canvas.getContext('2d');

        // Basic animation loop
        let frame = 0;
        function animate() {
          // Clear canvas
          ctx.fillStyle = '#1e1e1e';
          ctx.fillRect(0, 0, canvas.width, canvas.height);

          // Draw command info
          ctx.fillStyle = '#00ff00';
          ctx.font = '16px monospace';
          ctx.fillText('Command: ${command}', 20, 40);
          ctx.fillText('Branch: ${branchName}', 20, 60);

          // Draw animated frame indicator
          ctx.fillText('Frame: ' + frame, 20, 80);
          frame++;

          // Simple animation loop
          if (frame < 120) {
            requestAnimationFrame(animate);
          } else {
            // Animation finished
            vscode.postMessage({ command: 'log', text: 'Animation complete!' });
          }
        }

        // Start animation
        animate();
      </script>
    </body>
    </html>
  `;
}

// ────────────────────────────────────────────────────────────
// Utilities
// ────────────────────────────────────────────────────────────

function getNonce(): string {
  let text = '';
  const possible = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  for (let i = 0; i < 32; i++) {
    text += possible.charAt(Math.floor(Math.random() * possible.length));
  }
  return text;
}
```

---

## 🚀 Step 4: Build & Run Your Extension

### Compile TypeScript to JavaScript:

```bash
# One-time compile
npm run compile

# Or watch mode (recompile on file change)
npm run webpack-watch
```

### Run the Extension in Debug Mode:

1. Open your project folder in VS Code
2. Press **F5** (or **Cmd+Shift+D** on Mac)
3. A new VS Code window opens with your extension loaded
4. Open the Command Palette (**Ctrl+Shift+P** / **Cmd+Shift+P**)
5. Type "GitCritters: Show Panel" and press Enter

You should see a panel appear with your animation running!

---

## 🎨 Step 5: Basic Canvas Animation Example

Replace the `animate()` function in the webview with a pixel-art cat:

```javascript
// Basic pixel art: draw a cat
function drawPixelCat(ctx, x, y, frame) {
  const scale = 2; // 2x zoom
  
  // Define cat sprite as pixel data (simplified)
  const catPixels = [
    '        ',
    '  ●   ● ',  // eyes
    '  ●   ● ',
    '    ●    ', // nose
    '  █████  ', // mouth
    '        ',
  ];

  ctx.fillStyle = '#ff9933'; // Orange
  for (let row = 0; row < catPixels.length; row++) {
    for (let col = 0; col < catPixels[row].length; col++) {
      if (catPixels[row][col] === '█') {
        ctx.fillRect(
          x + col * scale,
          y + row * scale,
          scale,
          scale
        );
      }
    }
  }

  // Animate blinking
  if (Math.floor(frame / 10) % 2 === 0) {
    ctx.fillStyle = '#000'; // Closed eyes
    ctx.fillRect(x + 2 * scale, y + 1 * scale, scale, scale);
    ctx.fillRect(x + 6 * scale, y + 1 * scale, scale, scale);
  }
}

let frame = 0;
function animate() {
  ctx.fillStyle = '#1e1e1e';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // Draw animated cat
  drawPixelCat(ctx, 100, 50, frame);

  frame++;
  if (frame < 120) {
    requestAnimationFrame(animate);
  }
}
```

---

## 🔊 Step 6: Add 8-Bit Sound Effects

Add this to your webview script:

```javascript
function play8BitFanfare() {
  const audioContext = new (window.AudioContext || window.webkitAudioContext)();
  const notes = [523, 659, 784, 1047]; // C5, E5, G5, C6
  const gain = audioContext.createGain();
  gain.connect(audioContext.destination);

  notes.forEach((freq, index) => {
    const osc = audioContext.createOscillator();
    osc.type = 'square'; // 8-bit sound
    osc.frequency.value = freq;
    osc.connect(gain);

    const startTime = audioContext.currentTime + index * 0.1;
    osc.start(startTime);
    osc.stop(startTime + 0.2);
  });

  // Fade out
  gain.gain.setValueAtTime(0.3, audioContext.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5);
}

// Call when animation succeeds
play8BitFanfare();
```

---

## 📝 Step 7: Add Configuration Support

Access settings defined in `package.json` contributions:

```typescript
// In extension.ts
function getExtensionConfig() {
  const config = vscode.workspace.getConfiguration('gitcritters');
  
  const soundEnabled = config.get('soundEnabled') as boolean;
  const animationSpeed = config.get('animationSpeed') as string;

  console.log(`Sound: ${soundEnabled}, Speed: ${animationSpeed}`);
  
  return { soundEnabled, animationSpeed };
}

// Use it before showing the panel
const { soundEnabled, animationSpeed } = getExtensionConfig();
showAnimationPanel('push', 'main', soundEnabled);
```

---

## 🎯 Step 8: Debug & Test

### Run with Debugging:

1. **F5** to launch the extension host
2. Open DevTools in the extension host: **Ctrl+Shift+I** (Windows/Linux) or **Cmd+Shift+I** (Mac)
3. Set breakpoints in `extension.ts` and step through code
4. Check the Debug Console for `console.log()` outputs

### Test Git Commands:

In the extension host window:
```bash
# Initialize a test git repo
mkdir test-repo
cd test-repo
git init
git config user.email "test@example.com"
git config user.name "Test User"

# Create a file and commit
echo "Hello" > file.txt
git add file.txt
git commit -m "Initial commit"

# Trigger a push (even if it fails, your extension detects the attempt)
git push
```

---

## 📦 Step 9: Package & Publish

### Create a Publisher Account:

1. Go to [Azure DevOps](https://dev.azure.com)
2. Sign up or log in
3. Create a Personal Access Token (PAT) with scopes: `Marketplace (manage)`

### Install vsce (VS Code Extension Publisher):

```bash
npm install -g @vscode/vsce
```

### Login to Marketplace:

```bash
vsce login <publisher-name>
# Paste your PAT when prompted
```

### Package the Extension:

```bash
vsce package
# Creates: gitcritters-0.0.1.vsix
```

### Publish to Marketplace:

```bash
vsce publish
```

You can now install from VS Code Marketplace!

---

## ✅ Quick Checklist: MVP Launch

- [ ] Scaffold project with `yo code`
- [ ] Wire Git API hooks in `extension.ts` (detect push/pull/commit)
- [ ] Create a basic canvas animation in `panel.html`
- [ ] Test git detection with real repo
- [ ] Add one command: `showPanel` with keyboard shortcut (optional)
- [ ] Add 8-bit audio using Web Audio API
- [ ] Create extension icon (128×128px PNG)
- [ ] Write a basic `README.md`
- [ ] Test in extension host (F5)
- [ ] Create Azure DevOps publisher account
- [ ] Package and publish to Marketplace

---

## 🔗 Key Resources

- **[VS Code Extension API](https://code.visualstudio.com/api/)** — full documentation
- **[Git Extension API](https://code.visualstudio.com/api/references/vscode-api#GitExtension)** — hooking into git commands
- **[WebviewPanel API](https://code.visualstudio.com/api/references/vscode-api#WebviewPanel)** — displaying content
- **[Content Security Policy](https://code.visualstudio.com/api/extension-guides/webview#content-security-policy)** — CSP for webviews
- **[Publishing Extensions](https://code.visualstudio.com/api/working-with-extensions/publishing-extension)** — marketplace submission

---

## 📚 Next Steps After MVP

1. **Add more commands** (merge, branch, stash, rebase) — map to different critters
2. **Build mini-games** — push timing, catch-the-commit, etc.
3. **Implement XP/progression** — level up critters, unlock cosmetics
4. **Add persistence** — save progress to VS Code's global storage API
5. **Create premium pack** — cosmetic critters, exclusive animations, monetize via Gumroad

---

**Happy coding! Your GitCritters extension is ready to ship.** 🐾
