---
name: gpt-image2-storyboard
description: Generate Japanese anime storyboard worksheet images with GPT Image2 from character sheet images, background images, and a Markdown text storyboard, following the project's 絵コンテルール.md and 絵コンテテンプレート.png format. Use when asked to create 絵コンテ, storyboard sheets, cut sheets, or scene worksheets as PNGs for ScarletEchoes or similar anime/video generation workflows.
metadata:
  short-description: GPT Image2 storyboard sheet generator
---

# GPT Image2 Storyboard

Use this skill to generate filled storyboard worksheet PNGs from:

- character sheet image paths
- stage/background image paths
- a Markdown text storyboard
- optional output directory and filename pattern

For ScarletEchoes, always read the current project rules first:

- `common/絵コンテルール.md`
- `common/絵コンテテンプレート.png`

The repository rule file is authoritative. If it conflicts with this skill, follow `絵コンテルール.md`.

## Required Inputs

Ask only for missing inputs that cannot be inferred from the request.

- `characters`: paths to character sheets for all appearing characters.
- `backgrounds`: paths to stage/background images. Multiple images are allowed.
- `text_storyboard`: Markdown file path or pasted Markdown content.
- `output_dir`: directory to save PNG sheets.
- `filename_pattern`: example `02_01_cut_sheet_[scene].png`, `03_[scene].png`, or `01_cut_sheet_01.png`.

Optional:

- work title
- episode number
- scene range
- scene-per-page policy
- color policy

Default assumptions:

- One worksheet image per scene.
- One worksheet can contain up to 8 cuts.
- If a scene has more than 8 cuts, split it into multiple worksheet pages and suffix filenames predictably, for example `_01`, `_02`.
- Output should match the template aspect and page layout.
- The result is a production storyboard worksheet, not a finished key visual.

## Workflow

1. Read `絵コンテルール.md` and inspect `絵コンテテンプレート.png` dimensions with `file`.
2. Read the text storyboard and identify target scenes.
3. For each scene, extract:
   - scene number
   - duration
   - summary/content
   - dialogue only as staging context, never as worksheet dialogue text
   - camera
   - screen description
   - direction
   - SE and notes only when visually relevant
4. Convert each scene into 1 to 8 visual cuts.
   - Preserve the storyboard's intent.
   - If a scene has only one block, divide it into natural visual beats.
   - Keep cut times continuous and equal to the scene duration.
5. Build a GPT Image2 prompt using the template in `絵コンテルール.md`.
6. Include reference image intent in the prompt:
   - character sheet paths and concise appearance notes
   - background paths and concise stage notes
   - template image path
7. Call the image generation tool.
8. Copy the generated PNG into `output_dir` using `filename_pattern`.
9. Verify every output with `file` and `ls -lh`.
10. If readability, template fidelity, or cut count is visibly wrong, regenerate with a stricter prompt.

## Prompt Requirements

The generated image must:

- use `common/絵コンテテンプレート.png` as the exact worksheet layout template
- preserve borders, headings, spacing, and the cut grid
- fill up to 8 cut cells
- leave unused cut cells blank
- use a rough storyboard drawing style
- prioritize readable layout and text
- include no speech balloons
- include no script dialogue, monologue, lyrics, or narration in the worksheet
- describe dialogue visually through posing, gaze, distance, and emotion
- show the scene visual as blocking plus establishing image
- keep cut summaries concise and visual

Prefer English inside worksheet fields when the rule template calls for English camera/cut fields. Japanese work titles and scene labels are allowed if already present in the template or requested.

## Scene Parsing

Support both headings:

```markdown
### シーン 01
### CUT 01
```

Recognize these fields when present:

```markdown
- 時間:
- 内容:
- セリフ:
- カメラ:
- 画面:
- 演出:
- SE:
- 備考:
```

Dialogue handling:

- Use dialogue to infer facial expression, interaction, and emotional turn.
- Do not write dialogue into worksheet cells unless the user explicitly asks for script annotations and the current `絵コンテルール.md` allows it.

## Cut Planning Heuristics

When turning one scene into up to 8 cuts:

- Cut 1: establish location and initial blocking.
- Cut 2: introduce primary subject or action.
- Middle cuts: show movement, reaction, reveal, or escalation.
- Final cut: show the scene's result or transition hook.

Use compact camera labels:

- `WIDE / STATIC`
- `MEDIUM / TRACKING`
- `CLOSE / DOLLY IN`
- `POV / PAN`
- `LOW ANGLE`
- `OVER THE SHOULDER`
- `HANDHELD`

Use time ranges relative to the scene:

- `00:00-00:03`
- `00:03-00:07`

## GPT Image2 Prompt Skeleton

Adapt this skeleton per scene. Keep it concise enough for image generation but explicit about layout fidelity.

```text
Generate a filled storyboard worksheet image for AI video generation using GPT Image2.

Use this exact layout template:
common/絵コンテテンプレート.png

Follow these rules:
- Preserve the template layout exactly.
- Do not redesign the sheet.
- Use one large SCENE VISUAL and up to 8 CUT LIST cells.
- Leave unused cut cells blank.
- Use rough storyboard sketch style, clean readable production worksheet.
- No speech balloons.
- No dialogue text or narration in cut summaries.
- Prioritize blocking, camera, character positions, movement arrows, background, lighting, emotion.

[WORK TITLE]
...

[EP]
...

[SCENE]
...

[DURATION]
...

[CHARACTER REFERENCES]
- Character name: path, appearance, outfit, role in this scene, starting position.

[BACKGROUND REFERENCES]
- Location name: path, relevant stage details.

[SCENE CONTENT]
...

[SCENE VISUAL]
Initial blocking:
...

[SCENE SUMMARY]
1. ...
2. ...
3. ...
4. ...

[CUT PLAN]
CUT1.
TIME:
CAMERA:
VISUAL ACTION:
SUMMARY:

CUT2.
...
```

## Saving Outputs

Generated images are saved by the image generation tool in its default generated-images directory. Always copy the latest generated PNG to the requested destination; leave the original generated image in place.

Use a safe copy pattern:

```bash
latest=$(find "${CODEX_HOME:-$HOME/.codex}/generated_images" -type f -name '*.png' -printf '%T@ %p\n' | sort -n | tail -1 | cut -d' ' -f2-)
cp "$latest" "<output_path>"
file "<output_path>"
```

If this command is unavailable on the current platform, find the newest generated PNG by another non-destructive method and copy it.

## Final Response

Report:

- output directory
- generated filenames
- count of sheets
- any scenes split across multiple sheets
- verification result, especially dimensions

Keep the final answer concise.
