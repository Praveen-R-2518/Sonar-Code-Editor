# 🛠️ Development & Setup Guide

Welcome to the development guide for **Sonar Code Editor**. This document covers how to set up your local development environment, build the application, and package it for distribution.

## 📋 Prerequisites

Before you begin, ensure you have the following installed:
- **Node.js**: `v18.x` or higher
- **NPM**: `v9.x` or higher
- **Git**

## 🚀 Local Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/Sonar-Code-Editor.git
   cd Sonar-Code-Editor
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Configure Environment Variables:**
   - Create a `.env` file in the root directory.
   - Configure your **Appwrite** endpoint, project ID, and other required keys required for authentication and real-time database syncing.

## 💻 Running in Development

To start the application with hot-reloading for both the Vite React renderer and the Electron wrapper:

```bash
npm run start
```
*This utilizes `concurrently` to run the Vite dev server, TypeScript compiler for the main process, and the Electron executable simultaneously.*

## 📂 Project Structure

```text
├── assets/                   # App icons and static graphical assets
├── build/                    # Electron builder output rules and assets
├── release/                  # Compiled executable applications (generated)
├── src/
│   ├── main/                 # Electron main process source
│   │   ├── main.ts           # App lifecycle & window management
│   │   ├── ipcHandlers.ts    # Secure IPC communication handlers
│   │   ├── monitoring.ts     # System monitoring & activity tracking
│   │   └── ...               
│   ├── preload/              # Electron context bridge scripts
│   │   └── preload.ts        
│   ├── renderer/             # React frontend source
│   │   ├── components/       # Reusable UI (Editor, Sidebar, FileTree, AdminPanel)
│   │   ├── context/          # React Contexts (Auth, Collaboration)
│   │   ├── hooks/            # Custom Hooks (Network status, Activity)
│   │   ├── pages/            # Main Views (IDE, Login, Admin Dashboard)
│   │   ├── services/         # Appwrite, localStore, PDF reporting
│   │   ├── styles/           # Global CSS definitions
│   │   ├── types/            # TypeScript declaration files
│   │   └── App.tsx           
│   └── shared/               # Shared types and constants (main ↔ renderer)
├── appwrite.config.json      # Appwrite Collection schema definitions
├── package.json              # Dependencies & scripts
└── vite.renderer.config.ts   # Vite configuration for the renderer UI
```

## 📦 Building & Packaging

To compile the application for production and package it into distributable executables:

```bash
# 1. Build the React renderer, Main process, and Preload scripts
npm run build

# 2. Package for your specific OS
npm run package:win    # For Windows (.exe)
npm run package:mac    # For macOS (.dmg, .app)
npm run package:linux  # For Linux (.AppImage)
```
*Compiled artifacts will be located in the `release/` directory.*

## Collaboration Regression Playbook

Use this manual test before each beta/release to prevent stale-content regressions in collaborative mode.

### Test Case: Delete + Recreate Same File Name

Goal:
- Verify that deleting a file on one machine and recreating the same path on another machine does not resurrect stale collaborative content.

Setup:
1. Start host on PC-1 and join from PC-2.
2. Open the same shared workspace on both peers.
3. Ensure `index.html` exists and has recognizable content, for example `OLD-CONTENT`.

Steps:
1. On PC-1, delete `index.html` from the file tree.
2. On PC-2, create a new file named `index.html`.
3. On PC-2, open another file (do not keep `index.html` active).
4. On PC-1, open `index.html`, add `NEW-CONTENT`, and save.
5. Open/reopen `index.html` on both PCs.

Expected Result:
1. Both peers show the latest `NEW-CONTENT`.
2. Neither peer falls back to pre-delete `OLD-CONTENT`.
3. No silent revert happens when switching tabs or reopening the file.

Failure Symptoms (regression):
1. `index.html` reopens with old content on either peer.
2. Edits made after recreate do not propagate to the other peer.
3. Content appears correct briefly, then reverts after tab switch.

### Extra Validation Matrix

Run the same case with the following variants:
1. Recreated file starts empty, then edited by PC-1.
2. Recreated file starts with non-empty content from PC-2, then edited by PC-1.
3. Autosave ON and Autosave OFF.
4. Active tab on recreated file vs inactive tab during remote edits.

Pass Criteria:
1. Latest edit always wins and replicates to both peers.
2. Empty file state is preserved when intentionally empty.
3. Reopen of file never restores pre-delete content.