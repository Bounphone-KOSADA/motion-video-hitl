# Motion Video Studio (Claude skill)

A [Claude](https://claude.ai) skill for making **original animated motion videos entirely in code**. Frames are drawn on a canvas, music and sound effects are synthesized with numpy, and the voices come from free text-to-speech. Claude stops after every stage so you can approve the work or ask for changes before it continues (human-in-the-loop).

**Stages:** topic → script → character sheet → storyboard → motion draft → music & sound effects → English / Lao (ພາສາລາວ) dub → final MP4

## Install

### Claude app (claude.ai / desktop)
1. Download `motion-video-hitl.zip` from [Releases](https://github.com/Bounphone-KOSADA/claude-video-skills/releases).
2. In Claude, open **Settings → Capabilities → Skills**, click **Upload skill**, and pick the zip.
3. Turn the skill on.

### Claude Code (plugin)
```
/plugin marketplace add Bounphone-KOSADA/claude-video-skills
/plugin install motion-video-hitl@bounphone-video-skills
```

### Claude Code (manual)
```bash
git clone https://github.com/Bounphone-KOSADA/claude-video-skills.git
cp -r claude-video-skills/skills/motion-video-hitl ~/.claude/skills/
```

## Use it
Ask in plain words, and the skill starts on its own:
- *"Make a 40-second motion video about how rice grows"*
- *"ສ້າງວິດີໂອອະນິເມຊັນ ເລົ່າປະຫວັດ [ຫົວຂໍ້] ມີສຽງພາກລາວ ແລະ ຄຳບັນຍາຍ"*

At each stage Claude shows one result (a table, an image or a video) and waits for you to approve it or ask for changes.

## What you need
- Claude with code execution. The skill uses Python, Node, Playwright/Chromium and ffmpeg in Claude's workspace.
- **English voices:** [Kokoro](https://github.com/thewh1teagle/kokoro-onnx), an open-weight model that runs in the workspace.
- **Lao voices:** [edge-tts](https://github.com/rany2/edge-tts), run on your own computer. Claude writes a small script and gives you one command to paste. This needs the Claude desktop app linked to a folder on your computer.
- A few steps use tools that only exist in the Claude app (asking approval questions, sending files, writing to your computer). In other environments, Claude handles those steps in plain chat.

## Rules the skill follows
- Characters are original. No known characters, mascots or logos.
- Facts about real people, places and history are looked up first, with sources.
- edge-tts is for personal use. For commercial work, use a licensed service such as Azure Speech.

## License
MIT. See [LICENSE](LICENSE).
