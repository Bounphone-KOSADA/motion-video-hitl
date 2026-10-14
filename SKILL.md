---
name: "motion-video-hitl"
description: "Make an original animated motion video in code from a topic, with human approval at each stage: topic, script, character sheet, scenes, sound effects, English/Lao dub."
---

# Motion video studio (human-in-the-loop)

Build a short animated film entirely in code: frames drawn on a canvas, music and sound effects synthesized, voices from text-to-speech. Every stage ends at an approval gate. **Never start the next stage until the user approves the current one.** On a change request, apply it and show the same gate again. If the project has a CLAUDE.md with motion-video house rules, follow those too; where they conflict, the user's latest instruction wins.

## How to run a gate
- Show ONE deliverable per gate (one PNG, one MP4, or one table in chat), plus 2-4 short lines: what it is, any choice worth questioning, the next decision.
- Ask for approval with AskUserQuestion when options are discrete (e.g. Approve / Change something); otherwise ask in one line.
- Record each approval in the task list so a returning user sees where things stand.
- One self-critique pass per gate, using a single image (a contact sheet), never frame-by-frame stills.

## Gate 0 - Accept the topic
Restate the topic as a one-paragraph pitch:
- title
- length (default 40 s), 16:9, 30 fps, 120 BPM
- main character in one sentence
- 5-8 beats with timestamps
- look: palette mood, outline weight, 2 typefaces (display + mono/text)

Ask: accept, adjust, or pick from 2-3 alternative angles. Characters must be original: no known characters, mascots, logos, or copyrighted displays (e.g. use an unlit silhouette for a landmark whose light show is protected).

## Gate 1 - Draft script
A table in chat with columns: time range | scene | on-screen action | camera/transition | key SFX | narration line | character line.
- Time major actions to the beat (0.5 s at 120 BPM).
- Keep each spoken line inside its scene. English runs about 2.5-3 words/s; Lao runs about 20-40% longer, so leave slack.
- Plan designed transitions (iris, match cut, dive), not plain crossfades.
- End on the title.

## Gate 2 - Character sheet
Write the character as a code module (`<name>.js` exporting `drawCharacter(ctx, opts)`), with parameters for turn angle, expression, arms, lean, squash, `t`, and line width. Render ONE 1920x1080 PNG with:
- turnaround: front, 3/4, side, back
- 5 expressions
- palette of 5-8 named colours
- props
- 4-5 character rules

Flag any design trade-off you notice (e.g. a feature that hides in profile).

## Gate 3 - Scenes
1. **Storyboard:** write `film.js` with `render(ctx, t)`, which is pure and keeps no state between frames, plus `events()`, which returns every timed cue (pops, stamps, transitions, title). Render ONE PNG of 6-9 key frames from the real film code, each with a number, title, one-line action and timestamp. Stop for approval.
2. **Motion draft:** the full length at 960x540, 30 fps, no sound. Check it with ONE ffmpeg contact sheet (`select='not(mod(n,30))',scale=320:-1,tile=8x5`). Send the MP4. Stop for approval.

Build notes:
- Render with headless Chromium (Playwright). If a browser download is blocked, find an existing binary (`python3 -c "from playwright.sync_api import sync_playwright as s;p=s().start();print(p.chromium.executable_path)"`).
- Pipe JPEG frames into ffmpeg (libx264, yuv420p, +faststart). Split the timeline into 1-second chunks across workers, then concat.
- Fonts come from `@fontsource/*`, inlined as base64.
- Maps come only from public-domain Natural Earth data via `world-atlas`, `d3-geo` and `topojson-client`. Pre-project to path strings, and keep lon/lat rings for a code-drawn globe.
- Quality: ease in/out, overshoot on arrivals, anticipation before big moves, parallax, 4px ink outlines kept constant under camera zoom, paper grain, and text on backing cards.
- Export `events.json` from `events()`. It is the single timing source for audio.

## Gate 4 - Sound effects (yes / no)
Ask: "Add music and sound effects? (yes / no)".
- **yes:** write `sound.py` (numpy + scipy) that synthesizes everything, with no samples.
  - Music: 120 BPM chord pad, bass, a lead instrument that changes per scene, light percussion.
  - SFX: one per event in `events.json`.
  - Check numerically: RMS per 2 s window, peak below -1 dBFS, and the level jump on 2-3 key hits.
- **no:** keep silence or a music bed only, whichever the user says.

Start the 1080p render in the background (`render-all.mjs out.mp4 30 1`) while composing audio. Render full resolution only once.

## Gate 5 - Dubbing voice (English / Lao / both / none)
Ask which. Show the script lines for that language and get approval of the wording before synthesis; for Lao, the user checks the translation.

**English: Kokoro** (open-weight, runs in the workspace):
- `pip install --break-system-packages kokoro-onnx soundfile`
- Model files come from the GitHub releases of `thewh1teagle/kokoro-onnx` (`kokoro-v1.0.onnx`, `voices-v1.0.bin`).
- Narrator `bf_emma` at speed 0.95; a character such as `am_puck` at speed about 1.1, with a light robot tint for robots (12 ms comb plus a 55 Hz ring-mod).

**Lao: edge-tts** (free, no key). Voices: `lo-LA-KeomanyNeural` (female, narrator) and `lo-LA-ChanthavongNeural` (male; raise pitch about +35 Hz for a cute character).
- The workspace usually cannot reach this service, so run it on the user's computer: request access to one folder, write `lao_tts_free.py` plus a launcher there with `device_commit_files`, and ask the user to paste ONE command.
- The launcher must create a private venv (`python3 -m venv .venv`) because Homebrew Python blocks pip installs (PEP 668), and must stop on the first error (`set -e`; don't pipe pip through `tail`).
- If a file needs replacing on the device, write it under a NEW filename and confirm the new size with `device_list_dir`, because an overwrite can silently fail.
- Stage the resulting zip back yourself and convert MP3 to WAV with ffmpeg.
- Azure Speech with a key is the official alternative for commercial use.

**Mixing (any language):**
- Trim silence on each clip.
- Re-time scene by scene: a line stays inside its scene window with a 0.12 s gap. Prefer up to 0.45 s early or 0.6 s late; if a scene's lines still don't fit, compress them evenly with an overlap-add stretch down to no less than 75%.
- Duck music about -11 dB under the voice (120 ms ramps). Set the voice about 6 dB above the bed RMS.
- Verify voice-over-bed of at least 6 dB on every line, using the projection of the mix onto the voice clip. Fix any quiet line before delivering.
- Mux the new audio onto the existing 1080p picture with `-c:v copy`; never re-render the picture for audio-only changes.
- Keep one MP4 per language and keep each language's `vo.wav`/`score.wav` under language-suffixed names.

## Delivery
- MP4 in H.264 + AAC at 1080p. If it's over 30 MB, re-encode with a 2-pass bitrate to fit.
- Source zip containing all scripts, `package.json`, `events.json`, and a README with the commands: storyboard, draft, sound, voice, final.
- Send files with SendUserFile, with a one-line caption.
- Close each gate in 3-5 lines: what changed, the file, the next decision.