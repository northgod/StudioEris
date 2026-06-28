---
name: video-prompt-builder
description: Build production-ready video generation prompts from Japanese or English stage directions, scene blocking, dialogue, camera notes, and selected visual references such as character sheets, background art, storyboard sheets, concept art, or prior frames. Use when Codex needs to combine user-provided ト書き, camera/action text, and canvas-selected images into a structured prompt for video generation AI, including anime cuts, scene shots, image-to-video requests, and reference-aware motion prompts.
---

# Video Prompt Builder

## Purpose

Use this skill to transform ト書き, scene notes, cut notes, camera instructions, and selected visual references into a single video generation prompt.

This is a ScarletEchoes project-local skill. Resolve project files relative to `{PROJECT_ROOT}`, the repository root that contains `README.md`, `common/`, `stories/`, and `.agent/`.

The prompt should be ready to paste into Seedance2.0 or another video generation tool. For ScarletEchoes work, follow `{PROJECT_ROOT}/common/動画生成ルール.md` as the authoritative prompt format.

Project path rules:

- Use project-root-relative paths for rule files, such as `common/動画生成ルール.md`.
- Text storyboard asset paths are relative to `{ANIME_ROOT}` (`stories/[Story_Title]/02_Anime`), not relative to the Markdown file location.
- Resolve text storyboard asset paths to `{PROJECT_ROOT}/{ANIME_ROOT}/<path>` before treating them as image references.
- The final video-generation prompt must identify reference images by filename only, for example `@Image: stage_sketch_auditorium_lecture_panic.png`, not by absolute path or `{ANIME_ROOT}`-relative path. Resolve full paths internally for existence checks, but strip directory names in the prompt.

## Inputs To Accept

Accept any combination of:

- `stage_directions`: ト書き, scenario text, cut description, dialogue, acting notes, camera notes, SE, duration.
- `character_references`: selected character sheets or images from the canvas.
- `background_references`: selected location, prop, stage, or environment images.
- `stage_sketch_references`: stage sketches named in the text storyboard, used for spatial continuity.
- `storyboard_references`: selected storyboard worksheet sheets, cut panels, keyframes, or previous generated frames.
- `output_preferences`: target model, duration, aspect ratio, language, prompt style, negative prompt needs.

If the user says the images are selected on the canvas, treat those selected images as active references. Do not ask for file paths unless the images are unavailable in the current context.

## Workflow

1. Identify the target unit: scene, cut, shot, or short sequence.
2. Extract from the stage directions:
   - duration and aspect ratio if present
   - location and time of day
   - characters present
   - action beats in chronological order
   - camera size, angle, lens feel, and movement
   - acting, expression, emotion, and relationship changes
   - lighting, atmosphere, color, and VFX
   - dialogue from the corresponding `CUT PLAN` item only as performance context unless the user wants lip-sync text
3. Inspect selected references and summarize only production-relevant details:
   - character design: hair, eyes, outfit, accessories, body scale, silhouette
   - background: architecture, layout, props, lighting, weather, time of day
   - storyboard/keyframe: composition, blocking, camera direction, motion arrows
4. When using v2 storyboard materials, keep scene visual and worksheet roles separate:
   - `scene_visual`: stage, initial blocking, spatial continuity, character placement.
   - `storyboard_sheet`: shot-by-shot progression, camera, performance, timing.
   - `character_reference`: character appearance and identity.
5. When building from a text storyboard, read the scene's `舞台スケッチ:` value and include that image as `Image A`. Include the corresponding storyboard worksheet PNG as `Image B` when available. Resolve character and background references through the reference maps, but write only filenames in the final prompt.
6. Build the prompt using the section order from `{PROJECT_ROOT}/common/動画生成ルール.md`.
7. Add constraints inside `ADAPTATION POLICY` and, when useful, an `Avoid:` line under the relevant section.
8. If information conflicts, prioritize in this order:
   1. explicit user instruction in the current request
   2. selected storyboard/keyframe for composition and blocking
   3. selected character sheet for character design
   4. selected background for stage design
   5. text stage directions for acting and timing

## Output Format

Default to the Seedance2.0 prompt template from `{PROJECT_ROOT}/common/動画生成ルール.md`. Keep section labels in English and use concise production English camera terms.

```markdown
Generate a video scene from the provided
REFERENCE IMAGES
- Image A: scene visual / stage sketch reference for this scene (@Image: <stage_sketch_filename.png>)
- Image B: storyboard worksheet for this scene (@Image: <storyboard_sheet_filename.png>)
- Image C and following: character reference images (@Image: <character_sheet_filename.png>)
- Next images after character references: background references (@Image: <background_filename.png>)
- Additional images: prior frames or keyframes if provided (@Image: <filename.png>)

Interpret the references as follows:
- Image A defines stage structure, character placement, spatial continuity, initial blocking, mob density, entrances, exits, furniture, key props, movement paths, and overall environment layout.
- Image B defines shot-by-shot progression, camera, action, performance, and timing.
- Read the storyboard worksheet strictly from left to right.
- Each cut image shows the intended framing, visual action, and progression of the scene.

Character reference images are the priority source for:
- character identity
- face
- hairstyle
- hair color
- eye color
- costume
- accessories
- body proportions

Important:
- Do not render the storyboard worksheet itself.
- Do not show the page, borders, boxes, labels, notes, arrows, or text from the worksheet.
- Convert the storyboard into an actual cinematic scene.
- Use Image A to understand spatial continuity and starting blocking.
- Use Image B to understand cut order, camera, performance, and timing.
- Follow the cuts in numerical order.
- Use the storyboard for staging and shot design.
- Use the character references for appearance fidelity.
- Keep all characters visually consistent throughout the scene.
- Dialogue written inside each CUT PLAN item is for acting, speaker assignment, mouth movement, and lip-sync timing.
- Do not add BGM.
- Do not add subtitle super.
- Do not display captions or subtitles on screen.

If the model supports speech or lip-sync:
- use the dialogue lines written inside each CUT PLAN item as spoken dialogue by the specified speaker.

If the model does not support speech generation:
- use the dialogue lines inside each CUT PLAN item only as acting cues and emotional cues.
- do not show the dialogue as on-screen text.

[STYLE]

[VISUAL STYLE]
cinematic anime, 総予算5億円

[CAMERA FEEL]
stable cinematic camera

[MOOD]
...

[CHARACTER CAST]
List all characters appearing in this scene.

[CHARACTERS]
- Character name(@Image: character_sheet_filename.png): appearance locks and role.

[BACKGROUND]
- Background(@Image: background_filename.png): stage layout, lighting, props.

[SCENE VISUAL REFERENCE]
- Stage sketch(@Image: stage_sketch_filename.png)
- Starting positions:
- Stage structure:
- Mob density and basic appearance:
- Entrances/exits/furniture/key props:
- Movement paths:

[CUT PLAN]
Follow the storyboard faithfully and interpret each cut in order.
Do not divide the duration evenly by default.
Assign each cut duration based on dialogue length, acting beats, reaction time, held tension, quick insert shots, and the narrative purpose of the cut.
Write dialogue inside the corresponding cut only.
Do not create a separate dialogue section.
CUT 1: ...
CUT 2: ...

ADAPTATION POLICY
- Faithfully reconstruct the storyboard.
- Faithfully preserve character appearance from the character references.
- Use Image A as the scene visual reference for spatial layout, starting positions, crowd density, and movement paths.
- Use Image B as the storyboard worksheet for cut order, camera, action, performance, and timing.
- Follow the cuts in order.
- Preserve the intended acting, emotion, and pacing.
- Dialogue is assigned inside the corresponding CUT PLAN item.
- Do not add BGM.
- Do not add subtitle super.
- Do not add on-screen captions or lyrics.
- If interpolation between cuts is needed, make it smooth and natural while preserving storyboard intent.
```

When the user asks for a shorter output, keep the same section names but make each section compact. Do not switch to a different prompt format for ScarletEchoes.

## Prompt Construction Rules

### Main Prompt

Write a structured prompt using the sections above. Include these elements when known:

- duration: `15 seconds`, `8 seconds`, etc.
- aspect ratio: `16:9`, `9:16`, `1:1`, etc.
- style: `Japanese TV anime`, `rough storyboard to final anime`, `cinematic anime`, etc.
- subject and setting
- chronological action beats
- camera and composition
- lighting and mood
- required continuity from references

### Character Consistency

When character sheets are provided, state visible identity locks explicitly:

```text
Preserve the character sheet exactly: hairstyle, hair color, eye color, outfit, accessories, body proportions, silhouette, and facial style.
Do not change costume, age, hairstyle, eye color, or accessories between frames.
```

For ScarletEchoes Alvina, if `alvina_sheet.png` is selected or named, lock:

```text
Alvina: white bob hair with red inner hair accents, red flower hair ornament, red eyes, white blouse, large red ribbon, white dress/tunic, black long cloak with red lining, black tights, black loafers.
Avoid brown cloak, blue eyes, gold hair ornament, boots, or outfit simplification.
```

### Background Consistency

When background art is selected, describe the stage layout, not just the mood:

- foreground, middle ground, background
- entrances/exits
- character starting positions
- important props
- time of day and lighting
- camera-facing direction

### Storyboard / Keyframe Consistency

When storyboard worksheet or cut panels are selected, assign the scene visual as `Image A` and the worksheet as `Image B`. Use them primarily for:

- framing
- camera angle
- blocking
- movement direction
- shot order
- emotional beat

Do not render the worksheet itself. Never show the page, borders, boxes, labels, notes, arrows, or worksheet text in the video. Convert the worksheet into an actual cinematic scene.

### Reference Filename Policy

For ScarletEchoes prompts, list every visual reference with filename only:

- stage sketch from `舞台スケッチ:`: `@Image: stage_sketch_....png`
- storyboard worksheet: `@Image: 06-08_scene_007_storyboard.png`
- character sheet: `@Image: alvina_sheet.png`
- background image: `@Image: 講堂_2.png`

Do not write absolute paths such as `D:/...` or `{ANIME_ROOT}`-relative paths such as `storyboards/03_1/...` in the final video prompt. Path resolution is an internal validation step only.

### Dialogue Handling

Use dialogue as acting context by default. Follow the rule from `{PROJECT_ROOT}/common/動画生成ルール.md`:

- If the model supports speech or lip-sync, use dialogue lines inside each `CUT PLAN` item as spoken dialogue by the specified speaker.
- If the model does not support speech generation, use dialogue inside each `CUT PLAN` item only as acting and emotional cues.
- Do not show dialogue as on-screen text.

Translate dialogue into facial expression, mouth movement, reaction timing, gaze, and body language as needed.

### Camera Terms

Use compact, model-friendly terms:

- `WIDE / STATIC`
- `MEDIUM / TRACKING`
- `CLOSE / DOLLY IN`
- `POV / PAN`
- `LOW ANGLE`
- `HIGH ANGLE`
- `OVER THE SHOULDER`
- `HANDHELD`
- `TILT UP`
- `SLOW PUSH IN`

Then explain the movement in plain language.

## Constraint Patterns

Put avoid constraints in the `Important` block or `ADAPTATION POLICY` rather than changing the template. Include these when character references are important:

```text
Avoid changing character design, changing outfit, changing hairstyle, changing eye color, extra characters, duplicated limbs, distorted hands, unreadable face, flickering costume details, unstable background layout, camera jitter, excessive blur, text overlays, subtitles, speech balloons, watermarks.
```

For storyboard-to-video:

```text
Avoid ignoring storyboard composition, changing camera angle, reversing movement direction, adding unrelated actions, cutting away too early.
```

## Quality Checklist

Before finalizing, verify the prompt answers:

- Who is on screen?
- Where are they in the stage layout?
- What changes over time?
- What does the camera do?
- What must match the selected references?
- What must be avoided?
- Is the action feasible within the requested duration?

If a required reference is missing from context, state the gap and build the best prompt from available inputs.
