# 📸 webcam-snap for Agents

Give Agents eyes. **webcam-snap** lets Agents capture a still photo from your machine's local webcam (`/dev/video0`) using `ffmpeg`, then read the image with a multimodal model to describe or analyze what it sees — or hand the photo back to you.

Ask *"take a selfie"*, *"拍张照片看看现在的环境"*, or *"what's in front of the camera?"* and Agents runs the capture and looks at the result.

```
you  ▸ 拍张照片看看房间乱不乱
Agents ▸ (captures /tmp/selfie.jpg from /dev/video0, reads it)
       ▸ 拍到了：桌面上有一个马克杯和几本书，椅子上搭着一件外套，整体还算整齐……
```

- 🐧 **Linux + V4L2** — works with built-in and USB webcams exposed as `/dev/video*`
- 🎛 **Exposure-aware** — skips the first 30 frames so auto-exposure settles before the shot
- 🧩 **Three install formats** — standalone skill, plugin, or plugin marketplace
- 🪶 **Tiny & dependency-light** — one `ffmpeg` call, one shell script, no services

---

## Requirements

- **Linux** with a V4L2 webcam (`ls /dev/video*` shows a device)
- **ffmpeg** — `sudo apt install ffmpeg` (Debian/Ubuntu) / `sudo dnf install ffmpeg` (Fedora)
- Read access to the video device (see [Troubleshooting](docs/HOWTOUSE.md#troubleshooting))
- Agents (for skill/plugin use) — the raw `ffmpeg` command works anywhere

---

## Install

Pick the method that fits how you use Agents. Full walkthrough in **[docs/HOWTOUSE.md](docs/HOWTOUSE.md)**.

### 1. As a plugin marketplace (recommended)

In an interactive Agents session:

```
/plugin marketplace add longsizhuo/Agents-webcam-snap
/plugin install webcam-snap@webcam-snap-marketplace
```

### 2. As a standalone skill (no plugin system)

Copy the skill folder into your personal skills directory:

```bash
git clone https://github.com/longsizhuo/Agents-webcam-snap.git
cp -r Agents-webcam-snap/plugins/webcam-snap/skills/webcam-snap ~/.Agents/skills/
```

Restart Agents — the `webcam-snap` skill is now available.

### 3. Just the script (no Agents at all)

```bash
bash plugins/webcam-snap/skills/webcam-snap/scripts/take_selfie.sh /tmp/selfie.jpg 1280x720
```

---

## How it works

1. A skill (`SKILL.md`) tells Agents *when* to reach for the camera and *how* to capture a frame.
2. Capture is a single `ffmpeg` V4L2 grab that discards the first 30 frames (auto-exposure warm-up) and writes one JPEG.
3. Agents reads the JPEG directly — modern multimodal models "see" the image without any extra OCR/vision service — and describes or acts on it, or sends it to you.

```
ffmpeg -y -f v4l2 -video_size 1280x720 -i /dev/video0 \
       -vf "select=gte(n\,30)" -frames:v 1 -vsync 0 -f image2 /tmp/selfie.jpg
```

---

## Repository layout

```
Agents-webcam-snap/
├── .Agents-plugin/
│   └── marketplace.json              # marketplace manifest (method 1)
├── plugins/
│   └── webcam-snap/                  # the plugin
│       ├── .Agents-plugin/
│       │   └── plugin.json           # plugin manifest
│       ├── skills/
│       │   └── webcam-snap/          # the skill (method 2 — copy this dir)
│       │       ├── SKILL.md
│       │       ├── README.md
│       │       └── scripts/
│       │           └── take_selfie.sh
│       └── README.md
├── docs/
│   └── HOWTOUSE.md                   # detailed usage + troubleshooting
├── LICENSE                           # MIT
└── README.md
```

---

## Scope & non-goals

**In scope:** local USB / built-in webcams on Linux via V4L2.

**Out of scope (by design):** network/IP cameras (RTSP/ONVIF), macOS/Windows capture, and video recording. These need different tooling; keeping the skill to one job keeps it reliable.

---

## License

[MIT](LICENSE) © longsizhuo
