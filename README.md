# sudo-askpass

Tiny helper for no-TTY shells to run sudo on-demand.

Pops a GUI password dialog on the desktop instead of a terminal prompt. When
an agentic tool needs sudo, run the command through this and the user just
authenticates in the dialog.

## Usage

```sh
SUDO_ASKPASS="$HOME/.local/bin/sudo-askpass" sudo -A <command>
# or, with the wrapper installed:
sudo-gui <command>
```

A password dialog appears (yad, falling back to zenity or kdialog). The
password goes straight from the dialog to sudo — the agentic tool never sees it.

## Install

```sh
mkdir -p ~/.local/bin
cp sudo-askpass sudo-gui ~/.local/bin/
chmod +x ~/.local/bin/sudo-askpass ~/.local/bin/sudo-gui
```

Requires a graphical session and one of: `yad`, `zenity`, `kdialog`.

## Files

- `sudo-askpass` — askpass program for `sudo -A`; prints the dialog password to stdout
- `sudo-gui` — wrapper: `sudo-gui <cmd>` runs `SUDO_ASKPASS=... sudo -A <cmd>`
