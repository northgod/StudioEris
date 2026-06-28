---
name: gpt-image2-storyboard
description: Generate Japanese anime storyboard worksheet images with the current runtime's native image-generation tool from character sheet images, background images, scene visual images, and an already prepared Markdown text storyboard, following the project's common/絵コンテルール.md and common/絵コンテ仕様.md v2 format. Also revise an arbitrary cut inside an already-created storyboard worksheet by regenerating only that cut with the native image-generation tool and pasting it back into the original cut cell. Use after text-storyboard when asked to create 絵コンテ, storyboard sheets, cut sheets, scene worksheet PNGs, or cut-level corrections for ScarletEchoes. Do not use this skill to create or normalize text storyboards from scripts or prose.
metadata:
  short-description: Native image storyboard sheet generator
---

# Native Image Storyboard

Use this skill to generate filled storyboard worksheet PNGs from an already prepared text storyboard.

This is a ScarletEchoes project-local skill. Resolve project files relative to `{PROJECT_ROOT}`, the repository root that contains `README.md`, `common/`, `stories/`, and `.agent/`.

Role boundary:

- This skill's job is to understand the story progression, scene state, character psychology, blocking, camera intent, environment continuity, and visual priorities, then communicate them accurately to the runtime's native image-generation model.
- Use the current runtime's native image-generation tool through the same implicit-reference behavior used by the stage-sketch workflow. In Codex, this means GPT Image2 via the native image-generation route; in Gemini-based runtimes, this means Nano Banana Pro via that runtime's native route or equivalent. Before generating, load the worksheet template, character sheets, background references, stage sketch, and any scene visual with `view_image` or the runtime's equivalent visible-image mechanism so they are visible in the conversation context.
- Generate storyboard worksheet images and replacement cut illustrations only through the runtime's native image-generation tool directly. Do not call image APIs, CLI wrappers, PowerShell scripts, Python scripts, or any other custom generation route as a fallback.
- Template fidelity must be achieved by loading the worksheet template with `view_image` or the runtime's equivalent visible-image mechanism as visible image context and by writing a precise prompt that identifies it as `worksheet_template`.
- For full worksheet generation, do not repair or create worksheet structure by external post-compositing, drawing template borders, rebuilding grids, or pasting generated art onto the template after generation.
- For partial cut correction of an already-created worksheet, use the dedicated "Partial Cut Correction Workflow" below: generate only the replacement cut illustration with the native image-generation tool, then perform deterministic crop/resize/paste back into the existing cut-image cell. This is the only mode where post-compositing into a worksheet is allowed.
- Do not use prompt-only image generation when references are required. Do not treat prompt paths as substitutes for images loaded into the conversation context with `view_image` or the runtime's equivalent visible-image mechanism.
- If template fidelity is poor, improve the prompt and regenerate with the native image-generation tool after reloading the same visible references. Only non-creative file operations such as copying, renaming, and verification are allowed after generation.
- If the runtime's native image-generation tool cannot be used, or references cannot be loaded into the visible conversation context, stop and report the blocked condition. Do not create a workaround script, do not use `.agent/skills/gpt-image2-storyboard/scripts/*`, and do not switch to an API call.

Inputs:

- character sheet image paths resolved from `{ANIME_ROOT}/references/REFERENCE_MAP.md`
- stage/background image paths resolved from `{ANIME_ROOT}/background/REFERENCE_MAP.md`
- a Markdown text storyboard produced by the user or by `text-storyboard`
- optional output directory and filename pattern

Do not use this skill to convert scripts, prose, novels, scenario notes, or legacy cut lists into text storyboards. For that stage, use `text-storyboard` first, then pass the saved Markdown to this skill.

For ScarletEchoes, always read the current project rules first. Resolve story-relative paths with these variables:

- `{PROJECT_ROOT}`: repository root, for example `D:/Github/ScarletEchoes/ScarletEchoes`
- `{STORY_ROOT}`: target story root folder, for example `stories/01_緋色の魔法遣い`
- `{ANIME_ROOT}`: `{STORY_ROOT}/02_Anime`

Text storyboard file paths are always relative to `{ANIME_ROOT}`, not relative to the Markdown file location. When a text storyboard contains paths such as `舞台スケッチ: storyboards/01/scene_034_stage_sketch.png`, `シーンビジュアル: storyboards/01/Scene_037_Blocking.png`, `references/alvina_sheet.png`, or `background/講堂_1.png`, resolve them by joining the path to `{ANIME_ROOT}` before checking existence or passing the image to the native image-generation tool.

Path normalization must also absorb legacy project-root-relative paths found in `REFERENCE_MAP.md` or older storyboard files:

- Expand `{PROJECT_ROOT}`, `{STORY_ROOT}`, and `{ANIME_ROOT}` variables first.
- Use absolute paths as-is.
- Resolve `stories/...` and `common/...` from `{PROJECT_ROOT}`.
- Resolve `02_Anime/...` from `{STORY_ROOT}`.
- Resolve `references/...`, `background/...`, `storyboards/...`, and `designs/...` from `{ANIME_ROOT}`.
- Treat any other relative path as ambiguous and stop for correction instead of resolving it relative to the Markdown file.

For output and new references written back into text storyboards or prompts, prefer `{ANIME_ROOT}`-relative paths.

- `{PROJECT_ROOT}/common/絵コンテルール.md`
- `{PROJECT_ROOT}/common/絵コンテ仕様.md`
- `{PROJECT_ROOT}/common/絵コンテテンプレート_V2.jpg` as the v2 worksheet template image reference with role `worksheet_template`, unless the project rule file explicitly selects another template
- `{PROJECT_ROOT}/{ANIME_ROOT}/references/REFERENCE_MAP.md`
- `{PROJECT_ROOT}/{ANIME_ROOT}/background/REFERENCE_MAP.md`

The repository rule file is authoritative. If it conflicts with this skill, follow `絵コンテルール.md`.

## Required Inputs

Ask only for missing inputs that cannot be inferred from the request.

- `characters`: paths to character sheets for all appearing characters.
- `backgrounds`: paths to stage/background images. Multiple images are allowed.
- `text_storyboard`: Markdown file path or pasted Markdown content that is already normalized. If the source is a script/prose/legacy conte, stop and use `text-storyboard` first.
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
- One worksheet can contain up to 5 cuts.
- If a scene has more than 5 cuts, split it into multiple worksheet pages and suffix filenames predictably, for example `_01`, `_02`.
- Output should match the template aspect and page layout.
- The result is a production storyboard worksheet, not a finished key visual.

## Workflow

1. Read `{PROJECT_ROOT}/common/絵コンテルール.md`, `{PROJECT_ROOT}/common/絵コンテ仕様.md`, `{PROJECT_ROOT}/{ANIME_ROOT}/references/REFERENCE_MAP.md`, `{PROJECT_ROOT}/{ANIME_ROOT}/background/REFERENCE_MAP.md`, and inspect the selected worksheet template dimensions. Treat the worksheet template as a required image reference, not as prompt text.
2. Read the text storyboard and identify target scenes.
3. For each scene in the prepared text storyboard, read:
   - scene number
   - duration
   - summary/content
   - `セリフ` inside each `CUT PLAN` item as staging metadata only, never as worksheet dialogue text
   - camera
   - screen description
   - direction
   - SE and notes only when visually relevant
   - `舞台スケッチ:` as the required continuity image for fixed or recurring stages
4. Use the existing `CUT PLAN` when present. If a prepared text storyboard has no `CUT PLAN`, ask the user to run or allow `text-storyboard`; do not silently author a new text storyboard here.
5. Before generating storyboard sheets, enforce stage-sketch continuity from the text storyboard:
   - Read `舞台スケッチ:` for every target scene.
   - Treat the value of `舞台スケッチ:` as an `{ANIME_ROOT}`-relative path. Resolve it to `{PROJECT_ROOT}/{ANIME_ROOT}/<path>` before use.
   - For recurring concrete stages, `舞台スケッチ:` is mandatory and must point to an existing PNG saved by `text-storyboard` under `{ANIME_ROOT}`.
   - The stage sketch must already have been generated by `text-storyboard` using the project two-pass sketch flow: prompt-only `LOCATION / BLOCKING OVERVIEW` / `BLOCKING MAP` draft, then a native image-generation reference-consistency pass with the draft sketch plus resolved character and background references loaded as visible image context.
   - If the sketch is missing, only a prompt-only draft, or lacks required named-character identity cues needed for continuity, stop and return to `text-storyboard` to regenerate the sketch with the second-pass reference consistency step.
   - Do not create a new stage sketch in this skill and do not silently ignore a missing sketch. If a required sketch is absent, stop and ask to run `text-storyboard` so it can generate and record the sketch.
   - Group consecutive or recurring scenes by the same `場所状態` and `舞台スケッチ:` value.
   - Load the same stage sketch image with `view_image` or the runtime's equivalent visible-image mechanism before calling the native image-generation tool for every scene in that stage group.
   - Use the sketch to lock seats, standing positions, facing directions, crowd density, recurring supporting figures, simplified mobs, lighting, key props, and movement paths.
   - Prevent character-design leakage: background students and mobs must remain generic and must not inherit Alvina's white/red hair, Justina's blue braid and glasses, or Patrona's green hair/purple robe.
6. Read supporting-figure and mob continuity notes from the prepared text storyboard when present. Do not rewrite the text storyboard to add them.
7. Prepare v2 storyboard-sheet references:
   - Treat `シーンビジュアル:` and `舞台スケッチ:` values as separate scene visual or continuity references, not as worksheet areas to redraw.
   - Resolve those paths relative to `{PROJECT_ROOT}/{ANIME_ROOT}` and load the actual images with `view_image` or the runtime's equivalent visible-image mechanism before calling the native image-generation tool.
   - The worksheet to generate is the cut-design sheet only: header, scene summary, and up to 5 left-to-right cut cells.
   - Do not create a large in-sheet SCENE VISUAL area.
8. Build a native image-generation prompt using the template in `絵コンテルール.md`.
9. Resolve every character and stage name through the fixed reference maps, then include reference image intent in the prompt:
   - character sheet paths from `{ANIME_ROOT}/references/REFERENCE_MAP.md` and concise appearance notes
   - background paths from `{ANIME_ROOT}/background/REFERENCE_MAP.md` and concise stage notes
   - stage sketch paths from `舞台スケッチ:` resolved relative to `{PROJECT_ROOT}/{ANIME_ROOT}` and concise continuity notes; mandatory for recurring concrete stages
   - worksheet template image, loaded with `view_image` and identified as role `worksheet_template`; do not write its path as a substitute for visible image context
   - Any image paths written directly in the text storyboard must be resolved relative to `{PROJECT_ROOT}/{ANIME_ROOT}` and loaded with `view_image` when available.
10. Generate the worksheet only after all references that apply to the scene have been loaded into the conversation context:
   - worksheet template
   - character sheets
   - background references
   - stage sketch / scene visual continuity references
   - prior generated sheet only when iterating or preserving a layout correction
11. Call the native image-generation tool with a prompt that explicitly maps visible images to roles, for example: `Image 1: worksheet_template`, `Image 2: Alvina character_reference`, `Image 3: stage_sketch`.
12. Copy the newest generated PNG from `$CODEX_HOME/generated_images/...` into `output_dir` using `filename_pattern`. Leave the original generated image in place.
13. Verify every output with file listing and visual inspection when possible.
14. If readability, template fidelity, or cut count is visibly wrong, reload the same references and regenerate with a stricter prompt. Do not fix template fidelity by external compositing or rebuilding the worksheet layout after generation.

## Partial Cut Correction Workflow

Use this mode when the user asks to revise, replace, regenerate, adjust, or fix a specific cut inside an already-created storyboard worksheet, even if the user does not explicitly describe the implementation flow.

This workflow is part of `gpt-image2-storyboard`; do not create a separate skill unless the requested operation becomes a generic image-editing task unrelated to storyboard worksheets.

1. Identify the target worksheet PNG, scene number, cut number, and requested change.
2. Load the target worksheet with `view_image` or the runtime's equivalent visible-image mechanism.
3. Load the relevant character, background, scene visual, and prior cut references that are needed to preserve continuity.
4. Generate a replacement cut illustration only, not a worksheet:
   - Use the runtime's native image-generation tool.
   - The prompt must say `single storyboard cut illustration panel only, not a worksheet`.
   - Match the existing worksheet's rough storyboard style, line weight, contrast, and limited accent colors.
   - Include no borders, labels, text, subtitles, speech bubbles, watermark, actor names, or credits.
   - State what must be excluded from the replacement cut, especially characters or faces that the user asked not to show.
5. Copy the generated replacement cut PNG from the runtime's generated-image directory into the project, leaving the original generated file in place.
6. Determine the target cut-image cell bounds from the existing worksheet. Use the worksheet's actual pixel coordinates, not a guessed template remake.
7. Deterministically crop and resize the generated replacement cut to cover the target cut-image cell, then paste it into the original worksheet image.
8. Redraw only the minimal existing worksheet elements that the paste overlaps, such as the cut-image frame and `CUT IMAGE` label.
9. If the requested change alters camera type or cut summary, update only those text fields in the target cut. Do not rewrite unrelated cuts, the scene summary, title, duration, or other worksheet layout unless the user asks.
10. Save the edited worksheet as a sibling versioned file by default, for example `scene_10_storyboard_sheet_cut2_gpt-image2.png`. Overwrite the original only when the user explicitly requests replacement.
11. Verify the final worksheet visually and check dimensions. Also keep or report the standalone replacement cut PNG when it is useful for traceability.

Allowed deterministic local operations in this mode:

- file copy/rename from the native generation output
- image dimension inspection
- crop/cover-fit/resize
- paste into the existing cut-image cell
- redraw the pasted-over cell border or label
- limited text-field replacement for the corrected cut only

Not allowed in this mode:

- generating the replacement cut through an API, CLI wrapper, or custom model route
- rebuilding the whole worksheet from scratch with code
- changing unrelated cuts
- using crop-only reuse from the old cut as the final correction when the user asked for GPT Image2 regeneration
- deleting the original worksheet or the original generated image unless explicitly requested

## Prompt Requirements

The generated image must:

- be a direct native image-generation result using the visible references, not a post-composited worksheet
- use the visible `worksheet_template` image reference as the exact worksheet layout template
- preserve borders, headings, spacing, and the cut grid
- fill up to 5 cut cells
- leave unused cut cells blank
- use a rough storyboard drawing style
- prioritize readable layout and text
- include no speech balloons
- include no script dialogue, monologue, lyrics, or narration in the worksheet
- describe dialogue visually through posing, gaze, distance, and emotion
- use scene visual and stage sketch references only as external continuity references
- never place a large SCENE VISUAL area inside the worksheet
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
- 場所状態:
- 舞台スケッチ:
- 登場人物:
- カメラ:
- 画面:
- 演出:
- SE:
- 備考:
```

Dialogue handling:

- Use `セリフ` inside each `CUT PLAN` item only to infer facial expression, mouth movement, interaction, emotional turn, and timing.
- Do not write dialogue into worksheet cells unless the user explicitly asks for script annotations and the current `絵コンテルール.md` allows it.
- Do not delete, summarize away, or rewrite dialogue metadata in the source text storyboard. This skill reads the per-cut metadata; `text-storyboard` owns creating and preserving it for downstream video prompts.

## Cut Plan Handling

Read the existing `CUT PLAN` from the prepared text storyboard. Do not author a new text conte here.

When the text storyboard already contains a coarse visual plan and the user explicitly asks for worksheet-friendly subdivision, keep the changes local to the image-generation prompt and do not rewrite the source Markdown. Use these visual heuristics only for interpreting the worksheet cells:

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

## Native Image Prompt Skeleton

Adapt this skeleton per scene and send it to the native image-generation tool after all required image references have been loaded with `view_image` or the runtime's equivalent visible-image mechanism. Keep it concise enough for image generation but explicit about layout fidelity and visible-image roles.

```text
Generate a filled storyboard worksheet image for AI video generation using the current runtime's native image-generation model.

Use the visible worksheet template reference as the exact layout template. Do not infer the template from a file path in this prompt.

[VISIBLE IMAGE ROLES]
- Image 1: worksheet_template.
- Image 2: ...
- Image 3: ...

Follow these rules:
- Preserve the template layout exactly.
- Do not redesign the sheet.
- Use a 21:9 v2 storyboard worksheet with up to 5 left-to-right CUT LIST cells.
- Do not include a large SCENE VISUAL area in the worksheet.
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
- Character name: canonical name, resolved path from REFERENCE_MAP.md, appearance, outfit, role in this scene, starting position.

[BACKGROUND REFERENCES]
- Location name: canonical name, resolved image reference from REFERENCE_MAP.md, relevant stage details.

[WORKSHEET TEMPLATE REFERENCE]
- Role: worksheet_template. Use the visible template image exactly; preserve its borders, headings, spacing, and cut grid. Do not use a file path in the prompt as a substitute for this visible image reference.

[STAGE SKETCH / CONTINUITY REFERENCES]
- Stage sketch from `舞台スケッチ:`: actual image reference, shared by all scenes in the same stage state.
- Scene visual from `シーンビジュアル:` when present: actual image reference for stage layout and initial blocking.
- Preserve from sketch: exact seats/standing marks, facing directions, character distances, crowd density, generic mob designs, key props, windows/doors/furniture, and movement paths.
- If the stage sketch is a blocking map, preserve its map format, labels, legend, entrance/exit marks, arrows, seating banks, and camera angle unless the scene's cut plan explicitly changes blocking.
- Character-design isolation: only Alvina has white hair with red inner tips and red flower; only Justina has blue braid and round glasses; only Patrona has green hair, purple hat/robe, and blue-crystal staff. Do not copy these traits to background students.

[SCENE CONTENT]
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

Save final worksheet PNGs under the target story's anime folder, usually `{PROJECT_ROOT}/{ANIME_ROOT}/storyboards/<episode>/`.

- Treat `output_dir` as `{ANIME_ROOT}`-relative unless the user gives an explicit project-root-relative path.
- Copy or save generated images to the requested project path using the requested `filename_pattern`.
- Do not hardcode user-home or global Codex generated-image paths in this project-local skill.
- If the selected native image-generation tool returns a generated file path, copy that file to the project output path and verify the project output file.
- In Codex, if the native tool stores images under `$CODEX_HOME/generated_images/...`, locate the newest relevant generated image, copy it to the project output path, and verify the project output file.
- If the generated image cannot be located or saved through the runtime's native output mechanism, report that saving is blocked instead of switching to API or guessing another location.

## Final Response

Report:

- output directory
- generated filenames
- count of sheets
- any scenes split across multiple sheets
- verification result, especially dimensions

Keep the final answer concise.
