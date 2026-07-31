# I Turned My reMarkable Paper Pro Into a Fully Customized Power Tool — With Claude's Help

You know that scene in Harry Potter where Tom Riddle's diary writes back? That's now a real thing on my reMarkable Paper Pro. But that was just the beginning of a deep customization journey that transformed a simple e-ink tablet into something extraordinary.

## The Apps

Here's what's now running on my Paper Pro, all installed through a single session with Claude:

**Riddle — The AI Diary** (https://github.com/MaximeRivest/riddle)
Write anything with the stylus — a question, a thought, a journal entry. After a brief pause, your handwriting fades away, and an AI response writes itself back onto the page in flowing script. No keyboard. No screen glare. Just paper and ink, with something intelligent on the other side. It connects to any OpenAI-compatible API.

**Chromium — A Full Web Browser on E-Ink**
Tap to click, swipe to scroll, type on the built-in keyboard. It's a full Chromium browser running on an e-ink display. Browsing the web has never felt this calm.

**KOReader — The Ultimate E-Book Reader** (https://github.com/koreader/koreader)
Supports PDF, EPUB, MOBI, FB2, DOC, RTF, HTML, DjVu, and more. Built-in PDF reflow for scanned documents, dictionary lookup, customizable fonts and layouts. A massive upgrade over the native reader.

**PaperTerm — A Real Terminal**
A terminal emulator with an on-screen keyboard and fast partial e-ink updates. SSH into servers, run scripts, manage files — all from the tablet.

**Store — On-Device App Store**
Browse and install apps directly on the tablet without needing a computer.

**Bad Apple — E-Ink Benchmark**
The classic Bad Apple video playing at max FPS on e-ink. A fun benchmark that pushes the display to its limits.

## System Enhancements

Beyond apps, I also added:

- **reLuminate** — unlocks enhanced brightness levels beyond factory limits
- **CJK Fonts (Noto Sans)** — Chinese and Korean characters now display perfectly in file titles and documents, no more empty squares
- **rmtemplate** — a CLI tool for uploading custom notebook templates
- **Auto-updates disabled** — prevents OS updates from wiping all customizations

## Why This Feels Different

We interact with AI through chat boxes and keyboards all day. Writing by hand on the Riddle diary is something else entirely. It slows you down. You think before you write. The response appears the same way — ink forming on paper, not text streaming on a screen. It turns AI interaction into something contemplative instead of transactional.

And having a full browser, e-book reader, and terminal on a distraction-free e-ink device changes how you use a tablet. No notifications pulling your attention. No bright screen straining your eyes. Just tools that work.

## The Technical Journey

Getting all of this running required serious engineering. The reMarkable Paper Pro has a read-only root filesystem with an overlay, verified boot, and auto-updates that wipe your customizations. Here's what makes it work:

- **xovi** — an LD_PRELOAD-based extension loader that hooks into the tablet's UI process
- **AppLoad** — an app launcher that injects itself into the sidebar
- **remagic** — a companion CLI that automates the setup process

The trickiest parts:
- Systemd silently strips LD_PRELOAD from services, so you need a drop-in config written directly to the rootfs (after unmounting the /etc overlay)
- AppLoad versions are tied to specific OS versions — using the wrong one crashes xochitl with a cryptic hashed identifier error
- A Qt resource hashtable needs rebuilding every time the OS updates (~2 minutes of the tablet hashing thousands of QML files)
- My tablet auto-updated from OS 3.24 to 3.27 mid-installation, wiping everything — Claude adapted and rebuilt the entire setup for the new version
- CJK fonts need to be in `/usr/share/fonts/` for the system UI, with a persistent backup in `/home/root/.local/share/fonts/` to survive updates

Claude didn't just run commands — it diagnosed ELF binary symbol mismatches, wrote background scripts that survive SSH disconnections, parsed overlay filesystems, and adapted on the fly when the OS updated underneath us. This is what AI-assisted engineering looks like when the problems are genuinely hard.

## Get Started

If you have a reMarkable Paper Pro in developer mode:

1. Install remagic: `curl -fsSL https://raw.githubusercontent.com/maximerivest/remagic/main/get.sh | sh`
2. Or just tell Claude what you want — it'll handle the rest

**Project links:**
- Riddle: https://github.com/MaximeRivest/riddle
- remagic: https://github.com/maximerivest/remagic
- KOReader: https://github.com/koreader/koreader

#reMarkable #AI #Claude #OpenSource #Journaling #eInk #Productivity #HarryPotter #KOReader #Chromium #DIY
