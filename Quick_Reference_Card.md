# VS Code Extension Development — Quick Reference Card
## Keep This Open While Building GitCritters

---

## 🚀 Command Quick Links

```bash
# Project Setup (one time)
npm install -g yo generator-code
yo code

# Daily Development
npm run watch              # Watch + auto-compile TypeScript
npm run compile            # One-time compile
F5 (in VS Code)            # Launch extension in debug host

# Testing
npm run lint               # Check code quality
npm run test               # Run unit tests

# Publishing
vsce package               # Create .vsix file
vsce publish               # Upload to Marketplace
```

---

## 🏗️ File Structure

```
gitcritters/
├── src/
│   ├── extension.ts       ← Main entry point
│   ├── gitDetector.ts     ← Git API hooks
│   ├── animationManager.ts ← WebviewPanel logic
│   └── gameEngine.ts      ← Mini-game logic
├── media/
│   ├── panel.html         ← Animation webview
│   ├── sprites/           ← PNG sprite sheets
│   └── sounds/            ← Audio files
├── .vscode/
│   ├── launch.json        ← Debug config
│   └── tasks.json         ← Build tasks
├── out/                   ← Compiled JS (auto)
├── package.json
├── tsconfig.json
└── webpack.config.js
```

---

## 📝 Essential Code Snippets

### 1. Activate Extension & Register Command

```typescript
// src/extension.ts
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  console.log('Extension activated!');

  let cmd = vscode.commands.registerCommand(
    'gitcritters.showPanel',
    () => showAnimationPanel()
  );
  
  context.subscriptions.push(cmd);
}

export function deactivate() {}
```

### 2. Create WebviewPanel (Animation Display)

```typescript
function showAnimationPanel(command: string = 'push') {
  const panel = vscode.window.createWebviewPanel(
    'gitcritters',
    'GitCritters',
    vscode.ViewColumn.Beside,
    { enableScripts: true }
  );

  const nonce = getNonce(); // Generate CSP nonce
  panel.webview.html = getWebviewContent(panel.webview, nonce, command);

  setTimeout(() => panel.dispose(), 6000); // Close after 6s
}

function getNonce(): string {
  let text = '';
  const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  for (let i = 0; i < 32; i++) {
    text += chars.charAt(Math.floor(Math.random() * chars.length));
  }
  return text;
}
```

### 3. Git API Hook (Detect Push)

```typescript
function setupGitDetection() {
  const gitExt = vscode.extensions.getExtension('vscode.git');
  
  if (!gitExt) {
    vscode.window.showErrorMessage('Git extension not found');
    return;
  }

  const activate = gitExt.isActive 
    ? Promise.resolve(gitExt.exports)
    : gitExt.activate();

  activate.then((api) => {
    const git = api.getAPI(1);
    
    git.repositories.forEach((repo: any) => {
      let lastAhead = 0;
      
      repo.state.onDidChange(() => {
        const currentAhead = repo.state.HEAD?.ahead ?? 0;
        
        if (lastAhead > 0 && currentAhead === 0) {
          showAnimationPanel('push');
        }
        
        lastAhead = currentAhead;
      });
    });
  });
}
```

### 4. WebView HTML (Canvas Animation)

```typescript
function getWebviewContent(
  webview: vscode.Webview,
  nonce: string,
  command: string
): string {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta http-equiv="Content-Security-Policy"
            content="default-src 'none'; 
                     img-src ${webview.cspSource}; 
                     script-src 'nonce-${nonce}';">
      <style>
        body { margin: 0; padding: 20px; background: #1e1e1e; color: #00ff00; }
        canvas { image-rendering: pixelated; border: 1px solid #00ff00; }
      </style>
    </head>
    <body>
      <h1>${command.toUpperCase()}</h1>
      <canvas id="canvas" width="340" height="160"></canvas>
      
      <script nonce="${nonce}">
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        let frame = 0;
        function animate() {
          ctx.fillStyle = '#1e1e1e';
          ctx.fillRect(0, 0, 340, 160);
          
          // Draw your animation here
          ctx.fillStyle = '#ff9933';
          ctx.fillRect(frame * 2, 70, 20, 20);
          
          frame++;
          if (frame < 120) {
            requestAnimationFrame(animate);
          }
        }
        
        animate();
      </script>
    </body>
    </html>
  `;
}
```

### 5. 8-Bit Sound Effect

```javascript
// Inside WebView script
function play8BitSound(frequency = 523, duration = 200) {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  
  osc.type = 'square'; // 8-bit waveform
  osc.frequency.value = frequency;
  
  osc.connect(gain);
  gain.connect(ctx.destination);
  
  gain.gain.setValueAtTime(0.3, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + duration / 1000);
  
  osc.start(ctx.currentTime);
  osc.stop(ctx.currentTime + duration / 1000);
}

// Usage
play8BitSound(523, 200);   // C5 note, 200ms
play8BitSound(659, 200);   // E5 note
play8BitSound(784, 200);   // G5 note
```

### 6. Read Extension Config

```typescript
function getExtensionSettings() {
  const config = vscode.workspace.getConfiguration('gitcritters');
  
  const soundEnabled = config.get('soundEnabled') as boolean;
  const animationSpeed = config.get('animationSpeed') as string;
  
  return { soundEnabled, animationSpeed };
}

// Use it
const { soundEnabled, animationSpeed } = getExtensionSettings();
if (soundEnabled) {
  play8BitSound(); // Play from webview
}
```

### 7. Save/Load State (Progression)

```typescript
// Save to global state
context.globalState.update('gitcritters.xp', 150);

// Load from global state
const xp = context.globalState.get('gitcritters.xp') ?? 0;

// Watch for changes
context.globalState.onDidChange((e) => {
  if (e.affectsConfiguration('gitcritters')) {
    console.log('Settings changed!');
  }
});
```

---

## 🎨 Canvas Drawing Patterns

### Draw Pixel Art (Simple Grid)

```javascript
const SCALE = 2; // 2x zoom
const pixelData = [
  '  XX  ', // Row 0
  ' XXXX ',  // Row 1
  'XXXXXX',  // Row 2
  ' XXXX ',  // Row 3
  '  XX  '   // Row 4
];

ctx.fillStyle = '#ff9933';
for (let row = 0; row < pixelData.length; row++) {
  for (let col = 0; col < pixelData[row].length; col++) {
    if (pixelData[row][col] === 'X') {
      ctx.fillRect(col * SCALE, row * SCALE, SCALE, SCALE);
    }
  }
}
```

### Draw Sprite from PNG

```javascript
const spriteSheet = new Image();
spriteSheet.src = '...'; // URL to PNG
spriteSheet.onload = () => {
  const frameIndex = 0;
  const frameWidth = 32;
  const frameHeight = 32;
  
  ctx.drawImage(
    spriteSheet,
    frameIndex * frameWidth, 0,     // Source x, y
    frameWidth, frameHeight,         // Source width, height
    100, 50,                        // Dest x, y
    frameWidth * 2, frameHeight * 2 // Dest width, height (scaled)
  );
};
```

### Animation Loop with FPS Control

```javascript
const FPS = 12; // 12 frames per second
const frameTime = 1000 / FPS;
let lastFrameTime = 0;
let frameIndex = 0;

function animate(currentTime) {
  if (currentTime - lastFrameTime >= frameTime) {
    // Update frame
    frameIndex++;
    lastFrameTime = currentTime;
  }
  
  // Draw current frame
  drawFrame(frameIndex);
  
  requestAnimationFrame(animate);
}

animate(0);
```

---

## 🔍 Debugging Checklist

| Issue | Solution |
|---|---|
| Extension doesn't load | Check `activationEvents` in package.json |
| Command not in palette | Check `contributes.commands` in package.json |
| TypeScript errors | Run `npm run compile` to see full errors |
| WebView won't show | Check CSP nonce in HTML meta tag |
| Git detection fails | Ensure vscode.git extension is active |
| Sound doesn't play | Check AudioContext permissions, use try-catch |
| Animation jittery | Use `requestAnimationFrame()` instead of setInterval |

---

## 📊 Git Commands You'll Detect

| Command | Detection Method | Property to Watch |
|---|---|---|
| **push** | HEAD.ahead drops to 0 | `repo.state.HEAD.ahead` |
| **pull** | HEAD.ahead decreases | `repo.state.HEAD.ahead` |
| **commit** | HEAD.commit changes | `repo.state.HEAD.commit` |
| **branch** | HEAD.name changes | `repo.state.HEAD.name` |
| **merge** | Conflict state or ahead count | Combine multiple checks |

---

## 🎮 Game Mechanic Formulas

### Push Timing Game

```javascript
const needlePosition = Math.sin(frameCount * 0.1) * 100; // Oscillate
const greenZoneMin = 40;
const greenZoneMax = 60;

if (needlePosition >= greenZoneMin && needlePosition <= greenZoneMax) {
  score = 150; // Perfect hit
} else if (needlePosition > 30 && needlePosition < 70) {
  score = 50;  // Good hit
} else {
  score = 0;   // Miss
}
```

### XP & Leveling

```typescript
function earnXP(amount: number) {
  let xp = context.globalState.get('xp') as number ?? 0;
  xp += amount;
  
  const level = Math.floor(xp / 100) + 1; // 100 XP per level
  const xpNeeded = (level * 100) - xp;
  
  context.globalState.update('xp', xp);
  context.globalState.update('level', level);
}
```

### Critter Unlock Logic

```typescript
function checkUnlocks(commitCount: number, pushCount: number) {
  if (commitCount >= 100) unlockCritter('fox');
  if (pushCount >= 50) unlockCritter('dragon');
  if (commitCount >= 500) unlockCritter('wizard');
  // etc...
}
```

---

## 📦 Publishing Credentials

```bash
# Create Publisher Account
https://dev.azure.com

# Generate Personal Access Token (PAT)
- Scopes: Marketplace (manage)

# Login to vsce
vsce login <publisher-name>
# Paste PAT when prompted

# Then publish
vsce publish
```

---

## 🔗 Common VS Code API Patterns

### Show Notification

```typescript
vscode.window.showInformationMessage('Hello!');
vscode.window.showWarningMessage('Warning');
vscode.window.showErrorMessage('Error');
```

### Input Dialog

```typescript
const answer = await vscode.window.showInputBox({
  placeHolder: 'Enter something...'
});
```

### Quick Pick (Dropdown)

```typescript
const selected = await vscode.window.showQuickPick(['Cat', 'Frog', 'Chick']);
```

### Open File/URL

```typescript
vscode.env.openExternal(vscode.Uri.parse('https://example.com'));
```

### Terminal Integration

```typescript
const terminal = vscode.window.createTerminal('GitCritters');
terminal.sendText('git push');
```

---

## 💾 TypeScript Types Reference

### Common Extension Types

```typescript
vscode.ExtensionContext    // Context passed to activate()
vscode.WebviewPanel        // Animation panel
vscode.Webview             // Panel's webview API
vscode.ExtensionContext    // For storing state/subscriptions
vscode.commands.registerCommand()  // Register custom commands
vscode.window.onDidChangeActiveTextEditor  // Listen to editor changes
vscode.workspace.getConfiguration()  // Read user settings
```

---

## ✅ MVP Completion Checklist

- [ ] Extension scaffolded with `yo code`
- [ ] Git detection working (detects push/pull)
- [ ] WebviewPanel opens and displays animation
- [ ] Canvas animation renders without errors
- [ ] 8-bit sound plays on command
- [ ] Auto-closes panel after 6 seconds
- [ ] No console errors (F5 debug works)
- [ ] Settings in package.json are readable
- [ ] Command shows in Command Palette
- [ ] README.md written with instructions

---

## 🚀 Next Phase: Games & Progression

```javascript
// Phase 1 Done ✅

// Phase 2: Add mini-games
- Push timing game
- Catch-the-commit game
- Commit streak counter

// Phase 3: Add progression
- XP system
- Critter unlocks
- Achievement badges
- Repo pet (grows with commits)
```

---

## 📚 Key Docs (Bookmark These)

- [VS Code Extension API](https://code.visualstudio.com/api/)
- [Git Extension API](https://code.visualstudio.com/api/references/vscode-api#GitExtension)
- [WebviewPanel Docs](https://code.visualstudio.com/api/extension-guides/webview)
- [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

---

**Print this page and keep it on your second monitor while coding!** 📋
