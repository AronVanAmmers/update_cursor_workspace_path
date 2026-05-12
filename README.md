# Cursor Workspace Path Updater

Small script: after you rename a project folder, it moves Cursor’s workspace SQLite data so chat/agent history can keep working.

## Usage

1. Rename the folder (`old_folder` → `new_folder`).

2. Open **new_folder** in Cursor once so it creates a workspace entry.

3. **Quit Cursor completely** (all windows). If Cursor is running, it may lock files or overwrite them.

4. Open a normal terminal **outside** Cursor (not the integrated terminal). See **Windows** or **Linux / WSL** below.

5. **`cd` into the renamed folder**, then run the script. It asks for the **old** folder name. It copies `state.vscdb` (plus `-wal` / `-shm` when present) from the old workspace folder into the new one and rewrites stored paths.

6. Start Cursor again.

### Windows

CMD or PowerShell, outside Cursor:

```bash
cd C:\Users\Guzelbilen\Desktop\new_folder
python update_cursor_workspace_path_EN.py
```

### Linux / WSL

bash or another system shell, outside Cursor:

```bash
cd /home/youruser/develop/new_folder
python3 update_cursor_workspace_path_EN.py
```

### Where Cursor stores workspaces

The script checks paths that already exist on disk; you only need this list when something fails.

- **Cursor on Windows + folder in WSL:** Cursor often writes under Windows. From WSL that looks like  
  `/mnt/c/Users/<WindowsUser>/AppData/Roaming/Cursor/User/workspaceStorage`.  
  If `/mnt/c` is there, the script adds this automatically (username from `CURSOR_WINDOWS_USERNAME`, then `$USER`, then `$LOGNAME`). If your Linux login name differs from your Windows profile folder, set **`CURSOR_WINDOWS_USERNAME`** to that folder name under `C:\Users\`.
- **Linux “server” layout:** often `~/.cursor-server/data/User/workspaceStorage`
- **Linux desktop layout:** often `~/.config/Cursor/User/workspaceStorage`

Override: set **`CURSOR_USER_DATA_DIR`** to the directory that **contains** the `User` folder (the script uses `User/workspaceStorage` under it).

## What it does

- Asks for the old folder name (same parent directory as the new name).
- Locates old and new workspace entries under `workspaceStorage`.
- Copies the SQLite files that hold workspace state.
- Rewrites path strings (including `vscode-remote://…` for WSL when needed).
- Backs up an existing target `state.vscdb` as `state.vscdb.backup`.

## Examples

### Windows

```bash
# Rename: old_folder → new_folder
# Open new_folder in Cursor once, then quit Cursor
# Run from CMD or PowerShell outside Cursor:

cd C:\Users\Guzelbilen\Desktop\new_folder
python update_cursor_workspace_path_EN.py
# Old folder name: old_folder
```

### Linux / WSL

```bash
# Rename: old_folder → new_folder
# Open new_folder in Cursor once, then quit Cursor
# Run from bash outside Cursor:

cd /home/youruser/develop/new_folder
python3 update_cursor_workspace_path_EN.py
# Old folder name: old_folder
```

## Flow

1. Find workspace tied to the **old** path (from the name you type).
2. Find workspace tied to the **current** directory (the **new** path).
3. Copy `state.vscdb` (and WAL sidecars if present).
4. Patch paths inside the copied DB and `workspace.json` when needed.

## Rules

- Run from an external shell: **Windows** → CMD/PowerShell; **Linux / WSL / macOS** → e.g. bash. Not Cursor’s integrated terminal.
- Quit Cursor before you run it.
- Chat message text is not edited; only path-like strings get replaced.
- Workspace keys (hashes) stay as Cursor assigned them.
- If the new workspace is missing, open the renamed folder in Cursor once, quit, and retry.

## Troubleshooting

**Workspace not found**

- You never opened the renamed folder in Cursor, or Cursor hasn’t written storage yet — open it once, quit, retry.
- Storage lives somewhere unexpected — check the paths above or set `CURSOR_USER_DATA_DIR`.

**History still missing**

- You ran it from Cursor’s terminal, or Cursor was still open — use an external shell and quit Cursor first.
- Confirm the script printed a successful copy (including `ItemTable` on the target DB).

**Paths look stale**

- Quit Cursor fully and reopen the project.
- On Windows, try running the terminal as Administrator if files stayed locked.
