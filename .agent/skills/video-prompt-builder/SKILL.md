---
name: video-prompt-builder
description: Build production-ready video generation prompts from storyboard worksheet images, character sheets, background references, and Japanese or English stage directions. Use when asked to create Seedance2.0 or other video AI prompts from 絵コンテ, storyboard sheets, text storyboards, ト書き, camera notes, acting notes, dialogue, or reference images in the ScarletEchoes workflow.
metadata:
  short-description: Storyboard-to-video prompt builder
---

# Video Prompt Builder

Use this skill to create video generation prompts that follow the project's video prompt rules.

For ScarletEchoes, always read the current project rule first:

- `common/動画生成ルール.md`

If the user also provides storyboard-generation context, consult:

- `common/絵コンテルール.md`

The repository rule files are authoritative. If they conflict with this skill, follow the rule files.

## Required Inputs

Ask only for missing inputs that cannot be inferred from the request.

- `storyboard_image`: path to the filled storyboard worksheet image for the scene.
- `characters`: character sheet image paths for all appearing characters.
- `backgrounds`: background or stage image paths when available.
- `stage_directions`: text storyboard, script excerpt, ト書き, dialogue, acting notes, camera notes, or pasted scene description.

Optional:

- target video model, default `Seedance2.0`
- output language, default English prompt with Japanese names/dialogue preserved when provided
- work title, episode number, scene number, scene duration
- whether speech/lip-sync is supported by the target model
- output file path for the prompt

Default assumptions:

- Image A is the storyboard worksheet.
- Image B and following are character and background references in the order listed in the prompt.
- The storyboard image defines staging, cut order, framing, and visual progression.
- Character sheets define identity and appearance more strongly than the storyboard drawing.
- Background references define the environment when they are available.
- Dialogue is used for acting and speaker assignment; it must not appear as subtitles or captions unless the user explicitly asks.

## Workflow

1. Read `common/動画生成ルール.md`.
2. Inspect the provided file paths and identify reference order:
   - Image A: storyboard worksheet.
   - Image B and following: character sheets.
   - Remaining images: background or stage references.
3. Read the user's stage directions and extract:
   - appearing characters
   - dialogue and speaker assignment
   - acting beats
   - camera instructions
   - cut numbers or timing if present
   - mood, lighting, pacing, and scene objective
4. Map every acting beat and camera instruction to the closest storyboard cut number.
   - Preserve the cut order from the storyboard.
   - If text directions have no cut numbers, distribute them across the visible or implied cut sequence.
   - Do not invent unrelated cuts.
5. Build a prompt using the structure in `common/動画生成ルール.md`.
6. Make `[CHARACTERS]`, `[BACKGROUND]`, and `[CUT PLAN]` scene-specific.
7. Keep visual constraints explicit:
   - do not render the storyboard worksheet itself
   - do not show worksheet borders, boxes, labels, notes, arrows, or text
   - do not add BGM
   - do not add subtitle super
   - do not display captions, subtitles, or lyrics
8. Verify that the final prompt includes:
   - reference image order
   - character cast
   - character appearance reference mapping
   - background reference mapping when available
   - cut-by-cut acting, camera, emotion, and motion instructions
   - dialogue handling policy
   - adaptation policy from the rule file

## Prompt Requirements

The final prompt must:

- follow `common/動画生成ルール.md`
- identify the storyboard image as the source of staging and shot design
- identify character sheets as the priority source for face, hair, outfit, accessories, and proportions
- identify background images as the priority source for environment details
- convert the worksheet into an actual cinematic scene
- start from the initial blocking shown in the storyboard main visual
- follow storyboard cuts in numerical order
- preserve acting, emotion, pacing, and camera intent from the stage directions
- keep characters visually consistent through the whole scene
- avoid on-screen text unless explicitly requested

## Stage Direction Parsing

Support Japanese and English labels such as:

```markdown
- 時間:
- 内容:
- セリフ:
- カメラ:
- 画面:
- 演出:
- SE:
- 備考:
- Dialogue:
- Camera:
- Action:
- Direction:
```

Dialogue handling:

- Include dialogue only as spoken dialogue when the target model supports speech or lip-sync.
- If the target model does not support speech, use dialogue only as facial expression, timing, and emotional cues.
- Never turn dialogue into subtitles, captions, speech balloons, or on-screen text by default.

SE and music handling:

- Include SE only when the target model supports sound effects or when it affects visual acting.
- Do not add BGM unless the user explicitly asks and the project rule allows it.

## Cut Plan Format

Use one block per storyboard cut.

```text
CUT 1
- Source: storyboard cut 1.
- Camera:
- Acting / motion:
- Character focus:
- Dialogue / lip-sync:
- Environment / lighting:
- Transition / pacing:
```

Keep camera language compact and production-oriented, for example:

- `WIDE / STATIC`
- `MEDIUM / TRACKING`
- `CLOSE / DOLLY IN`
- `POV / PAN`
- `LOW ANGLE`
- `OVER THE SHOULDER`
- `HANDHELD`

## Prompt Skeleton

Adapt this skeleton per scene while preserving the rule-file intent.

```text
Generate a video scene from the provided reference images for Seedance2.0.

REFERENCE IMAGES
- Image A: [storyboard_image] - storyboard worksheet for this scene.
- Image B: [character_name] character sheet - [path].
- Image C: [character_name] character sheet - [path].
- Image D: [location_name] background reference - [path].

Interpret the references as follows:
- The large upper scene visual in Image A shows the initial state before the acting begins.
- It defines starting blocking, character positions, environment, and overall composition.
- The lower cut list defines the shot-by-shot progression in time order.
- Each cut image shows intended framing, visual action, and progression.

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
- Start from the initial state shown in the main scene visual.
- Follow the cuts in numerical order.
- Use the storyboard for staging and shot design.
- Use character references for appearance fidelity.
- Use background references for environment fidelity.
- Keep all characters visually consistent throughout the scene.
- Dialogue information below is for acting and speaker assignment.
- Do not add BGM.
- Do not add subtitle super.
- Do not display captions or subtitles on screen.

If the model supports speech or lip-sync:
- use the dialogue lines as spoken dialogue by the specified speaker.

If the model does not support speech generation:
- use the dialogue lines only as acting cues and emotional cues.
- do not show the dialogue as on-screen text.

[STYLE]

[VISUAL STYLE]
cinematic anime, 総予算5億円

[CAMERA FEEL]
stable cinematic camera

[MOOD]
...

[CHARACTER CAST]
...

[CHARACTERS]
...

[BACKGROUND]
...

[CUT PLAN]
Follow the storyboard faithfully and interpret each cut in order.

CUT 1
- Source:
- Camera:
- Acting / motion:
- Character focus:
- Dialogue / lip-sync:
- Environment / lighting:
- Transition / pacing:

CUT 2
...

ADAPTATION POLICY
- Faithfully reconstruct the storyboard.
- Faithfully preserve character appearance from the character references.
- Start from the initial blocking in the storyboard main visual.
- Follow the cuts in order.
- Preserve the intended acting, emotion, and pacing.
- Dialogue is assigned by the speaker field above.
- Do not add BGM.
- Do not add subtitle super.
- Do not add on-screen captions or lyrics.
- If interpolation between cuts is needed, make it smooth and natural while preserving storyboard intent.
```

## Saving Outputs

If the user asks for a file, save the prompt as Markdown or plain text in the requested output path. Use a scene-specific filename when no filename is provided, for example:

- `scene_01_video_prompt.md`
- `02_01_seedance_prompt.md`

## Final Response

Report:

- whether the prompt was written to a file or returned inline
- source storyboard image
- character and background references used
- any missing inputs or assumptions

Keep the final answer concise.
