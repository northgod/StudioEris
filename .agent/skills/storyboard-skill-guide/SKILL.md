---
name: storyboard-skill-guide
description: Explain how to use the ScarletEchoes custom animation workflow skills text-storyboard, gpt-image2-storyboard, and video-prompt-builder in chat. Use when the user asks how to call, choose, sequence, or provide inputs for these storyboard/video prompt skills, asks for examples of prompts for them, or asks which skill to use for text storyboards, GPT Image2 storyboard worksheets, scene visuals, or video generation prompts.
---

# Storyboard Skill Guide

Use this skill to answer usage questions about the project-specific animation workflow skills:

- `text-storyboard`
- `gpt-image2-storyboard`
- `video-prompt-builder`

Give the user a concise chat explanation. Do not run the three target skills unless the user asks to perform the actual production work.

## Quick Answer Pattern

When asked how to use the skills, explain the workflow in this order:

1. Use `$text-storyboard` to convert source text, script, or scene notes into normalized text storyboards.
2. Use `$gpt-image2-storyboard` to turn the text storyboard and resolved references into scene visuals and 21:9 storyboard worksheets.
3. Use `$video-prompt-builder` to combine the scene visual, storyboard worksheet, character references, and CUT PLAN into a video generation prompt.

Mention that this project uses `{ANIME_ROOT}`-relative paths in text storyboards, storyboard sheets, and video prompts. `{ANIME_ROOT}` means the target story's `02_Anime/` folder.

## Skill Purposes

### text-storyboard

Use `$text-storyboard` when the user has prose, script, scene notes, or rough cuts and wants a text storyboard.

Expected inputs:

- Story title or target story folder.
- Episode or scene range.
- Source manuscript, script, or scene memo.
- Known character, background, and prop references if available.

Expected outputs:

- Markdown text storyboard.
- Scenes kept within the 15-second limit.
- `CUT PLAN` with cut timings based on dialogue length, acting beats, reactions, pauses, and cut purpose rather than equal division.
- Dialogue written inside the corresponding CUT item.
- `シーンビジュアル:` / `舞台スケッチ:` planned or generated path when needed.
- Reference roles such as `scene_visual`, `storyboard_sheet`, `character_reference`, and `background_reference`.

Example user prompt:

```text
Use $text-storyboard for stories/01_緋色の魔法遣い episode 03. Convert this script excerpt into 15-second text storyboard scenes and record all reference paths relative to 02_Anime/.
```

### gpt-image2-storyboard

Use `$gpt-image2-storyboard` when the text storyboard exists and the user wants visual storyboard assets.

Expected inputs:

- Normalized text storyboard.
- `common/絵コンテ仕様.md` as the v2 worksheet authority.
- Scene visual or stage sketch.
- Character, background, prop, and template references resolved to real files.

Expected outputs:

- Scene visual image if needed.
- 21:9 storyboard worksheet image.
- Left-to-right time flow.
- Maximum 5 cuts per sheet.
- Sheet 2 split when more than 5 cuts are truly needed.

Important constraints:

- Do not put dialogue, monologue, BGM, or subtitles into the worksheet image.
- Do not place the scene visual inside the worksheet; keep it as a separate image.
- Do not use path text alone as an image reference. Resolve files and show/load actual references before image generation when the environment supports it.

Example user prompt:

```text
Use $gpt-image2-storyboard to create the v2 storyboard worksheet for storyboards/03/scene_012.md. Use the scene visual and character references listed in the text storyboard, all relative to 02_Anime/.
```

### video-prompt-builder

Use `$video-prompt-builder` when the user has the visual references and wants a video generation prompt, such as for Seedance2.0.

Expected inputs:

- Image A: scene visual reference.
- Image B: storyboard worksheet.
- Image C and following: character references.
- Optional background references, prior frames, or keyframes.
- Text storyboard `CUT PLAN` and acting/camera details.

Expected outputs:

- Video generation prompt with reference roles separated.
- Cut-by-cut timing and action instructions.
- Dialogue assigned inside the corresponding cut when needed.
- Explicit instructions not to render worksheet borders, labels, boxes, notes, captions, subtitles, or BGM.

Core reference interpretation:

```text
Image A is the scene visual reference.
Image B is the storyboard worksheet.
Image C and following are character references.
Do not render the worksheet itself.
Use references only as production references.
```

Example user prompt:

```text
Use $video-prompt-builder for Scene 012. Image A is Scene_012_Blocking.png, Image B is Scene_012_StoryboardSheet.png, and Image C onward are the character sheets. Build a Seedance2.0 prompt from the CUT PLAN.
```

## Choosing the Right Skill

- If the user asks "文章からコンテにしたい": recommend `$text-storyboard`.
- If the user asks "絵コンテ画像を作りたい": recommend `$gpt-image2-storyboard`.
- If the user asks "動画生成AI用プロンプトにしたい": recommend `$video-prompt-builder`.
- If the user asks for the full workflow: recommend all three in order.
- If the user asks only how to call them: give the explicit `$skill-name` examples and stop.

## Project Rules to Mention

Mention these when relevant:

- Text storyboard scene length is 15 seconds or less.
- v2 storyboard worksheets are 21:9.
- Cuts run left to right in time order.
- One worksheet has at most 5 cuts.
- Scene visual and storyboard worksheet are separate images.
- Dialogue belongs in the text prompt/CUT PLAN, not in the worksheet image.
- New paths inside storyboard/video workflow files should be relative to `{ANIME_ROOT}`.
