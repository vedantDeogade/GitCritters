# Essential Files Checklist: Building a VS Code Extension
## Minimal Setup to Get From Zero to Working

---

## 📋 The 5 Essential Files (Bare Minimum)

When you scaffold with `yo code`, you get everything. But here's what's **actually required** for a working extension:

### 1️⃣ **package.json** — Extension Metadata & Configuration

This is the brain of your extension. It tells VS Code:
- What the extension is called
- What commands it exposes
- When to activate
- What settings are configurable

**Minimal package.json:**

```json
{
  "name": "gitcritters",
  "displayName": "GitCritters",
  "description": "Pixel-art animations for every git command",
  "version": "0.0.1",
  "author": "Your Name",
  "license": "MIT",
  "engines": {
    "vscode": "^1.70.0"
  },
  "activationEvents": [
    "onStartupFinished"
  ],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "gitcritters.showPanel",
        "title": "GitCritters: Show Panel"
      }
    ],
    "configuration": {
      "title": "GitCritters",
      "properties": {
        "gitcritters.soundEnabled": {
          "type": "boolean",
          "default": true
        }
      }
    }
  },
  "scripts": {
    "vscode:prepublish": "tsc -p ./",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./"
  },
  "devDependencies": {
    "@types/node": "^18.0.0",
    "@types/vscode": "^1.70.0",
    "typescript": "^4.7.0"
  }
}
```

**Key Sections:**
- `activationEvents`: When to load the extension (`onStartupFinished` = on VS Code startup)
- `main`: Path to compiled extension (typically `./out/extension.js`)
- `contributes.commands`: Commands users can run
- `contributes.configuration`: Settings users can customize

---

### 2️⃣ **src/extension.ts** — Main Extension Code

The entry point. This file runs when VS Code loads your extension.

**Minimal extension.ts:**

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  console.log('GitCritters is now active!');

  // Register a command
  let disposable = vscode.commands.registerCommand(
    'gitcritters.showPanel',
    () => {
      vscode.window.showInformationMessage('GitCritters panel opened!');
      showAnimationPanel();
    }
  );

  context.subscriptions.push(disposable);
}

export function deactivate() {
  console.log('GitCritters deactivated');
}

function showAnimationPanel() {
  const panel = vscode.window.createWebviewPanel(
    'gitcritters',
    'GitCritters Animation',
    vscode.ViewColumn.Beside,
    { enableScripts: true }
  );

  panel.webview.html = getWebviewContent();

  setTimeout(() => panel.dispose(), 6000);
}

function getWebviewContent(): string {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { margin: 0; padding: 20px; background: #1e1e1e; color: #00ff00; }
        h1 { font-family: monospace; }
        canvas { border: 1px solid #00ff00; image-rendering: pixelated; }
      </style>
    </head>
    <body>
      <h1>GitCritters Animation</h1>
      <canvas id="canvas" width="340" height="160"></canvas>
      <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        let frame = 0;
        function animate() {
          ctx.fillStyle = '#1e1e1e';
          ctx.fillRect(0, 0, 340, 160);
          
          // Draw a simple rectangle that moves
          ctx.fillStyle = '#ff9933';
          ctx.fillRect(frame * 2, 70, 20, 20);
          
          frame = (frame + 1) % 170;
          requestAnimationFrame(animate);
        }
        animate();
      </script>
    </body>
    </html>
  `;
}
```

**What it does:**
- `activate()` — Runs when extension loads
- `registerCommand()` — Creates a command users can run
- `createWebviewPanel()` — Opens the animation panel
- `getWebviewContent()` — Returns the HTML for the panel

---

### 3️⃣ **tsconfig.json** — TypeScript Configuration

Tells TypeScript how to compile your code.

**Minimal tsconfig.json:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./out",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  },
  "include": ["src"],
  "exclude": ["node_modules"]
}
```

---

### 4️⃣ **.vscode/launch.json** — Debug Configuration

Tells VS Code how to launch the extension in debug mode (F5).

**Minimal launch.json:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": ["--extensionDevelopmentPath=${workspaceFolder}"],
      "outFiles": ["${workspaceFolder}/out/**/*.js"],
      "preLaunchTask": "npm: compile"
    }
  ]
}
```

---

### 5️⃣ **.vscode/tasks.json** — Build Tasks

Allows VS Code to run `npm run compile` before launching the debugger.

**Minimal tasks.json:**

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "npm: compile",
      "type": "shell",
      "command": "npm",
      "args": ["run", "compile"],
      "isBackground": true
    }
  ]
}
```

---

## 📁 File Structure After Setup

```
gitcritters/
├── .vscode/
│   ├── launch.json          ✅ Debug config
│   └── tasks.json           ✅ Build tasks
│
├── src/
│   └── extension.ts         ✅ Main extension code
│
├── out/                     📦 (Auto-generated by tsc)
│   └── extension.js         (Compiled JavaScript)
│
├── node_modules/            📦 (Auto-created by npm)
├── package.json             ✅ Extension metadata
├── tsconfig.json            ✅ TypeScript config
└── README.md                (Documentation)
```

---

## 🚀 Quick Start: Get It Running in 5 Minutes

### Step 1: Create the files

```bash
mkdir gitcritters && cd gitcritters
npm init -y
```

Create the **5 files above** manually or use:
```bash
npm install -g yo generator-code
yo code
```

### Step 2: Install dependencies

```bash
npm install --save-dev @types/vscode @types/node typescript
```

### Step 3: Compile

```bash
npm run compile
```

### Step 4: Debug (F5 in VS Code)

- Open the project folder in VS Code
- Press **F5**
- A new VS Code window opens with your extension loaded
- Open Command Palette (**Ctrl+Shift+P**)
- Type "GitCritters: Show Panel"
- You see the animation panel!

---

## 📝 Adding More Features: Step-by-Step

### Feature 1: Detect Git Push

Update `src/extension.ts`:

```typescript
function setupGitDetection(context: vscode.ExtensionContext) {
  const gitExt = vscode.extensions.getExtension('vscode.git');
  if (!gitExt) return;

  if (!gitExt.isActive) {
    gitExt.activate().then((api) => setupGitHooks(api, context));
  } else {
    setupGitHooks(gitExt.exports, context);
  }
}

function setupGitHooks(gitApi: any, context: vscode.ExtensionContext) {
  const git = gitApi.getAPI(1);
  
  git.repositories.forEach((repo: any) => {
    let lastAhead = 0;
    
    repo.state.onDidChange(() => {
      const currentAhead = repo.state.HEAD?.ahead ?? 0;
      
      // Detect push: ahead count drops to 0
      if (lastAhead > 0 && currentAhead === 0) {
        vscode.window.showInformationMessage('Push detected! 🚀');
        showAnimationPanel();
      }
      
      lastAhead = currentAhead;
    });
  });
}

export function activate(context: vscode.ExtensionContext) {
  console.log('GitCritters is now active!');
  
  // Detect git commands
  setupGitDetection(context);
  
  // Register command
  let cmd = vscode.commands.registerCommand('gitcritters.showPanel', showAnimationPanel);
  context.subscriptions.push(cmd);
}
```

### Feature 2: Add Different Animations

Update `getWebviewContent()`:

```typescript
function getWebviewContent(command: string = 'push'): string {
  const animations: {[key: string]: string} = {
    push: 'A cat pushing a crate →',
    pull: 'A frog casting a rod ↙',
    commit: 'A chick stamping a seal ✓',
    merge: 'A dragon merging rivers ≋'
  };

  return `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { margin: 0; padding: 20px; background: #1e1e1e; color: #00ff00; font-family: monospace; }
        canvas { border: 1px solid #00ff00; image-rendering: pixelated; display: block; margin: 20px 0; }
      </style>
    </head>
    <body>
      <h1>GitCritters: ${command.toUpperCase()}</h1>
      <p>${animations[command] || 'Unknown command'}</p>
      <canvas id="canvas" width="340" height="160"></canvas>
      <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        // Draw based on command type
        ctx.fillStyle = '#1e1e1e';
        ctx.fillRect(0, 0, 340, 160);
        
        ctx.fillStyle = '#ff9933';
        ctx.font = '20px monospace';
        ctx.fillText('${command.toUpperCase()}', 100, 80);
        
        // Simple animation
        let frame = 0;
        function animate() {
          ctx.fillStyle = '#1e1e1e';
          ctx.fillRect(0, 120, 340, 40);
          
          ctx.fillStyle = '#00ff00';
          ctx.fillText('Frame: ' + frame, 50, 140);
          
          frame++;
          if (frame < 120) requestAnimationFrame(animate);
        }
        animate();
      </script>
    </body>
    </html>
  `;
}
```

### Feature 3: Add Sound

Add to webview HTML:

```javascript
function play8BitSound(frequency = 523, duration = 200) {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  
  osc.type = 'square';
  osc.frequency.value = frequency;
  
  osc.connect(gain);
  gain.connect(ctx.destination);
  
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + duration / 1000);
  
  osc.start(ctx.currentTime);
  osc.stop(ctx.currentTime + duration / 1000);
}

// Call on success
play8BitSound(523, 200); // C5 note
```

---

## ✅ Testing Checklist

Before publishing, verify:

- [ ] Extension activates on startup (no errors in Debug Console)
- [ ] Command shows in Command Palette
- [ ] Animation panel opens and closes after 6 seconds
- [ ] Canvas animates without flickering
- [ ] Sound plays when triggered
- [ ] Git detection works (push/pull/commit detected)
- [ ] No TypeScript compilation errors (`npm run compile` succeeds)

---

## 📦 Publishing Checklist

Once your MVP works:

1. **Create publisher account:** https://dev.azure.com
2. **Create Personal Access Token (PAT)** with Marketplace scope
3. **Install vsce:** `npm install -g @vscode/vsce`
4. **Login:** `vsce login your-publisher-name`
5. **Update version in package.json:** `"version": "0.1.0"`
6. **Package:** `vsce package`
7. **Publish:** `vsce publish`

Your extension is now on the VS Code Marketplace!

---

## 🔗 Essential Docs

- **[VS Code Extension Anatomy](https://code.visualstudio.com/api/get-started/extension-anatomy)** — what each file does
- **[Your First Extension](https://code.visualstudio.com/api/get-started/your-first-extension)** — official tutorial
- **[activation Events](https://code.visualstudio.com/api/references/activation-events)** — when your extension loads

---

## 💡 Pro Tips

1. **Use `npm run watch`** during development — auto-recompiles on save
2. **Use F5 + Ctrl+Shift+I** to debug with DevTools open
3. **Check the Debug Console** for `console.log()` output
4. **Set breakpoints** in TS files and step through code
5. **Use `context.subscriptions.push()`** to register all cleanup resources

---

**Now you have everything needed to build and publish a VS Code extension!** 🎉
