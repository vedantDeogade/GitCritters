# GitCritters
GitCritters 🐾

Pixel-art critters that react to your git commands — right inside VS Code.

GitCritters turns everyday git actions into tiny animated moments. Commit, push, pull, merge — each triggers a hand-crafted pixel-art animation and 8-bit sound, with an XP and unlock system that rewards you for actually using git well (not just often).

Show Image


✨ Features


Animated reactions to ~10 core git commands (commit, push, pull, merge, branch, stash, etc.), each with its own critter animation
git push mini-game — a timing-based challenge that plays during pushes
XP & progression system — earn experience for git activity and level up
Unlockable critters — new characters unlock as you progress
Retro audio — 8-bit square-wave sound effects via the Web Audio API
Lightweight overlay — runs in a VS Code WebviewPanel without disrupting your normal git workflow



🎮 How It Works

GitCritters listens to VS Code's Git Extension API (repo.state.onDidChange()) to detect git operations in real time. When a tracked action fires, a WebviewPanel renders a Canvas 2D animation matched to that command, accompanied by a short sound cue. Progression and unlock state persist across sessions.


📦 Installation

From the VS Code Marketplace:


Open the Extensions panel in VS Code (Ctrl+Shift+X / Cmd+Shift+X)
Search for GitCritters
Click Install


From source:

bashgit clone https://github.com/<your-username>/gitcritters.git
cd gitcritters
npm install

Then press F5 in VS Code to launch an Extension Development Host with GitCritters loaded.


🛠 Tech Stack


TypeScript / Node.js — extension host logic
VS Code Extension API — Git Extension API for repo state changes
Canvas 2D — sprite rendering inside a WebviewPanel
Web Audio API — square-wave oscillator sound effects
Aseprite — sprite art authoring and export pipeline



🚀 Usage

Once installed, GitCritters activates automatically in any workspace with a git repository. No configuration needed — just use git as you normally would (via terminal, Source Control panel, or command palette) and watch for critter reactions in the panel.

Commands

CommandDescriptionGitCritters: Show PanelOpens the critter panel manuallyGitCritters: View ProgressShows XP, level, and unlocked crittersGitCritters: SettingsConfigure sound, animation speed, etc.

(Update this table with your actual contributed commands from package.json.)


⚙️ Configuration

SettingDefaultDescriptiongitcritters.soundEnabledtrueToggle sound effectsgitcritters.animationSpeednormalslow / normal / fast

(Adjust to match your actual contributes.configuration entries.)


🤝 Contributing

Contributions are welcome! This project is also meant as a friendly entry point into the VS Code extension ecosystem.


Fork the repo
Create a feature branch (git checkout -b feature/new-critter)
Commit your changes
Open a pull request


See CONTRIBUTING.md for details on adding new critters/animations via the Aseprite pipeline.


🗺 Roadmap


 Additional critter animations for remaining git commands
 Custom critter themes / skins
 Marketplace freemium tier (cosmetic unlocks)
 Multiplayer/leaderboard support for teams



📄 License

This project is licensed under the MIT License — (update if you choose a different license).


🙏 Acknowledgments

Built with pixel art crafted in Aseprite, and powered by the VS Code Extension API.