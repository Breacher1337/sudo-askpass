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

## How the password flows (and why the caller never sees it)

`sudo -A` does not read the password from a terminal. It runs the askpass
program named by `SUDO_ASKPASS` and reads the password from that program's
stdout over a private pipe:

    GUI dialog (yad/zenity/kdialog) -> askpass stdout -> pipe -> sudo only

Consequences, verified at runtime:

- The command run as root inherits only the caller's stdin/stdout/stderr.
  The askpass pipe is consumed by sudo itself and is never passed to the
  child process — the child's fd table contains only fds 0/1/2.
- The password is not in the child's environment. sudo sets only
  `SUDO_USER`, `SUDO_UID`, `SUDO_GID`, `SUDO_COMMAND` and `SUDO_HOME`.
- Nothing is written to disk: the askpass script stores nothing, the dialog
  tool prints only to stdout, and sudo never logs passwords.
- The scripts are installed mode `0711` (`rwx--x--x`): executable by all,
  readable only by the owner — and they contain no secret anyway, since the
  password exists only transiently inside the pipe.

The pipe between the dialog and sudo is the only channel; the calling shell
and anything it runs have no access to the password.
