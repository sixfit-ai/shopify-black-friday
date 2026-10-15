# Demo GIF: shot list (20–30 s)

Record in claude.ai on the **test store** with the skill uploaded and Shopify
connected. Use a clean chat, browser at ~1280 px wide, zoom 110–125% so text
reads at 800 px. Hide bookmarks and personal tabs. Answer the interview off
camera and cut it.

| # | Time | On screen | Notes |
|---|---|---|---|
| 1 | 0–3 s | Type: *"Is 30% off everything a good idea for Black Friday?"* and send | The hook. |
| 2 | 3–7 s | Claude names the store and the days to Black Friday, and starts the pricing scan ("your prices include 21% VAT…") | Pause on the VAT line: it's the surprise for most merchants. |
| 3 | 7–15 s | The Hero / Clearance tables: max safe discount, € kept per sale, "needs +49% units to pay off" | Scroll slowly. Stop on one product where 30% would lose money. |
| 4 | 15–21 s | The three-scenario table and the recommendation line | Hold ~2 s on "Recommended: Balanced, because your #1 priority is profit". |
| 5 | 21–24 s | The discount plan table → type *"Yes, create them"* | The approval moment. |
| 6 | 24–28 s | *"5 discounts created and verified, all scheduled for Fri 27 Nov 00:00 – Mon 30 Nov 23:59"* plus the rollback line | End on the rollback sentence. |

Afterwards, roll back on the test store (ask Claude: *"undo the discounts you just created"*).

## Convert .mov → docs/demo.gif (~800 px, under ~8 MB)

**gifski** (best quality; `brew install gifski ffmpeg`):

```bash
mkdir -p /tmp/demo-frames docs
ffmpeg -i demo.mov -vf "fps=12,scale=800:-1:flags=lanczos" /tmp/demo-frames/f%04d.png
gifski --fps 12 --width 800 --quality 80 -o docs/demo.gif /tmp/demo-frames/f*.png
ls -lh docs/demo.gif   # if > 8 MB: lower --quality to 70 or fps to 10
```

**ffmpeg only** (two-pass palette):

```bash
ffmpeg -i demo.mov -vf "fps=12,scale=800:-1:flags=lanczos,palettegen=stats_mode=diff" -y /tmp/palette.png
ffmpeg -i demo.mov -i /tmp/palette.png \
  -lavfi "fps=12,scale=800:-1:flags=lanczos [x]; [x][1:v] paletteuse=dither=bayer:bayer_scale=5:diff_mode=rectangle" \
  -y docs/demo.gif
ls -lh docs/demo.gif   # if > 8 MB: fps=10, or trim with -ss 0 -t 28
```
