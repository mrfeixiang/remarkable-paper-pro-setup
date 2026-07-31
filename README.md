# reMarkable Paper Pro Setup Guide

A complete guide to transforming the reMarkable Paper Pro into a fully customized power tool — apps, browser, e-book reader, terminal, CJK fonts, and an AI diary that writes back.

Everything here was set up in a single session with [Claude](https://claude.ai).

## What's Installed

### Apps (via AppLoad)

| App | Description |
|-----|-------------|
| [Riddle](https://github.com/MaximeRivest/riddle) | AI diary — write with the stylus, your handwriting fades, AI responds in flowing script |
| Chromium | Full e-ink web browser with tap, swipe, and built-in keyboard |
| [KOReader](https://github.com/koreader/koreader) | E-book reader supporting PDF, EPUB, MOBI, FB2, DOC, DjVu, CBZ, and more |
| PaperTerm | Terminal emulator with on-screen keyboard |
| Store | On-device app store — install apps without a computer |
| Bad Apple | E-ink benchmark — the classic video at max FPS |

### System Enhancements

| Mod | Description |
|-----|-------------|
| [reLuminate](https://github.com/unreMarkableLabs/reLuminate) | Unlock brightness levels beyond factory limits |
| CJK Fonts | Noto Sans CJK SC (Chinese) and KR (Korean) for proper display |
| [rmtemplate](https://github.com/zer0trip/rmtemplate) | CLI tool for uploading custom notebook templates |
| Auto-updates disabled | Prevents OS updates from wiping customizations |

## Prerequisites

- reMarkable Paper Pro in **developer mode**
- USB connection to your computer
- SSH password (Settings > Help > Copyrights and licenses)

## Quick Start

The easiest path is [remagic](https://github.com/maximerivest/remagic):

```bash
curl -fsSL https://raw.githubusercontent.com/maximerivest/remagic/main/get.sh | sh
remagic install riddle
remagic config riddle
```

## Key Technical Notes

The Paper Pro has a **read-only rootfs** with an `/etc` overlay backed by volatile storage. This means:

- **Systemd drop-ins** must be written after `umount -R /etc` to persist on the real rootfs
- **LD_PRELOAD** set via systemd `Environment=` is the only reliable way to inject xovi into xochitl
- **AppLoad versions** are tied to OS versions (v0.4.2 for OS 3.24, v0.5.3 for OS 3.27+)
- **Qt resource hashtable** must be rebuilt per OS version (~2 minutes)
- **OS updates wipe rootfs** — disable `swupdate` service, or keep recovery scripts ready
- **CJK fonts** need to be in `/usr/share/fonts/` for system UI, with backup copies in `/home/root/.local/share/fonts/`

### After an OS Update

```bash
# SSH into the tablet, then:
mount -o remount,rw /
umount -R /etc

# Re-create systemd drop-in
mkdir -p /etc/systemd/system/xochitl.service.d
cat << EOF > /etc/systemd/system/xochitl.service.d/xovi.conf
[Service]
Environment="LD_PRELOAD=/home/root/xovi/xovi.so"
Environment="XOVI_ROOT=/home/root/xovi/services/xochitl.service/"
Environment="QML_DISABLE_DISK_CACHE=1"
Environment="QML_XHR_ALLOW_FILE_WRITE=1"
Environment="QML_XHR_ALLOW_FILE_READ=1"
EOF

# Re-enable tripletap
bash /home/root/xovi-tripletap/enable.sh

# Rebuild hashtab
bash /home/root/build-hashtab3.sh

# Restore CJK fonts
cp /home/root/.local/share/fonts/NotoSansCJK*.otf /usr/share/fonts/
fc-cache -f

# Disable auto-updates again
systemctl mask swupdate.socket swupdate.service

# Restart
systemctl daemon-reload
systemctl restart xochitl
```

## The Stack

```
xochitl (reMarkable UI)
  └── xovi (LD_PRELOAD extension loader)
       ├── qt-resource-rebuilder (QML patching)
       └── AppLoad (sidebar app launcher)
            ├── Riddle
            ├── Chromium
            ├── KOReader
            ├── PaperTerm
            ├── Store
            └── Bad Apple
```

## Blog Posts

- [English — I Turned My reMarkable Paper Pro Into a Fully Customized Power Tool](riddle-linkedin-post.md)
- [中文 — 我如何用 Claude 把 reMarkable Paper Pro 打造成终极生产力工具](remarkable-blog-chinese.md)

## Credits

- [Riddle](https://github.com/MaximeRivest/riddle) by Maxime Rivest
- [remagic](https://github.com/maximerivest/remagic) by Maxime Rivest
- [xovi](https://github.com/asivery/rm-xovi-extensions) by asivery
- [AppLoad](https://github.com/asivery/rm-appload) by asivery
- [KOReader](https://github.com/koreader/koreader) by KOReader community
- [reLuminate](https://github.com/unreMarkableLabs/reLuminate) by unreMarkableLabs
- [rmtemplate](https://github.com/zer0trip/rmtemplate) by zer0trip
- [xovi-tripletap](https://github.com/rmitchellscott/xovi-tripletap) by rmitchellscott
- Setup assisted by [Claude](https://claude.ai)
