# Termux Backup & Restore

## What is backed up

- `~/.termux/` — font, colors, `termux.properties`, extra-keys/services config
- Shell dotfiles: `~/.bashrc`, `~/.profile`, `~/.gitconfig`
- App configs in `~/.config/` — htop, micro, mpv, opencode, rclone
- `packages.txt` — list of all installed packages (`dpkg --get-selections`)

## How to create a backup

```bash
DEST=/storage/emulated/0/Download/termux-backup-$(date +%Y%m%d-%H%M%S)
mkdir -p "$DEST"
cp -r ~/.termux "$DEST/"
cp ~/.bashrc ~/.profile ~/.gitconfig "$DEST/"
cp -r ~/.config "$DEST/"/ \
  --exclude 'pulse' --exclude 'node_modules'
dpkg --get-selections > "$DEST/packages.txt"
```

Notes:
- `--exclude 'pulse'` skips the runtime socket dir and `--exclude 'node_modules'`
  skips app deps (incl. their `node_modules/.bin` symlinks). Both are
  regenerated automatically and cannot be copied to `/storage/emulated/0`
  anyway (no symlink support).
- If you still see `Permission denied` for symlinks, it is harmless: `cp`
  just skips them and the real config files are copied fine.

Then copy `$DEST` somewhere safe (PC, SD card, cloud).

## How to restore

Config files live in the app's private storage, so they must be copied back,
not just opened from Download.

```bash
DEST=/storage/emulated/0/Download/termux-backup-YYYYMMDD-HHMMSS
cp -r "$DEST/.termux" ~/
cp "$DEST/.bashrc" "$DEST/.profile" "$DEST/.gitconfig" ~/
cp -r "$DEST/.config" ~/
termux-reload-settings   # apply termux.properties/font/colors
```

## How to reinstall all packages

```bash
pkg update
pkg install $(cat "$DEST/packages.txt")
```

or, if the package list is mangled:

```bash
while read pkg state; do if [ "$state" = install ]; then pkg install -y "$pkg"; fi; done < "$DEST/packages.txt"
```

## Notes

- Config lives in the app's private storage (`/data/data/com.termux/files/home`),
  not on the shared storage — that is why the copy to Download is needed.
- `/storage/emulated/0` does not support symlinks, so any symlinks
  (e.g. runtime sockets, `node_modules/.bin`) are skipped during backup.
  These are throwaway runtime artifacts, not real config.
- `~/.ssh` keys and `~/.git-credentials` are NOT included by default
  for security; copy them separately if wanted.

---

For an overview of all my other projects, see https://adegard.github.io/blog/
