---
name: nano-triple
version: "2.0.0"
description: "3 images, one prompt, instant A/B/C comparison. Style presets, iterative refinement, image editing, aspect ratios. Powered by Gemini Nano Banana Pro. The fastest way to find the right image."
author: mvanhorn
license: MIT
repository: https://github.com/mvanhorn/nano-triple
homepage: https://aistudio.google.com
triggers:
  - make me an image
  - generate an image
  - create an image
  - make an image
  - design an image
  - draw me something
  - create a picture
  - generate artwork
  - make a graphic
  - create a visual
  - design a logo
  - make a banner
  - create an illustration
  - generate variations
  - show me 3 options
metadata:
  openclaw:
    emoji: "🎨"
    tags:
      - image-generation
      - nano-banana-pro
      - creative
      - parallel
      - ai-art
      - style-presets
      - iterative-refinement
      - image-editing
      - aspect-ratios
      - batch-generation
      - prompt-enhancement
      - visual-design
      - content-creation
      - variations
---

# Nano Triple: 3 Images, Same Prompt, You Pick

Generate 3 image variations in parallel for every request. The user picks a winner or refines any version. Always 3 at a time - never just 1.

---

## Core Flow

### Step 1: Parse the Request

Extract these from the user's message:
- **Prompt** - What they want to see
- **Style** - Any style preference (default: none/natural)
- **Aspect ratio** - Any size preference (default: 1:1 square)
- **Mode** - New generation or editing an existing image
- **Refinement** - Are they giving feedback on a previous set?

### Step 2: Build the Generation Prompt

Start with the user's words. Apply style and aspect ratio modifiers if specified.

**Prompt construction rules:**
1. User's core description comes first
2. Style preset appended if requested
3. Never rewrite the user's intent - only append style/quality modifiers
4. For refinements, incorporate feedback into the base prompt

### Step 3: Generate 3 Images in Parallel

Run all 3 simultaneously. Same prompt, 3 calls. The model's inherent randomness produces 3 different results.

```bash
# Generate all 3 in parallel (run these concurrently, not sequentially)
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "[CONSTRUCTED PROMPT]"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageSizeOptions": {"aspectRatio": "[ASPECT_RATIO]"}
    }
  }' | python3 -c "
import sys, json, base64
resp = json.load(sys.stdin)
for part in resp.get('candidates', [{}])[0].get('content', {}).get('parts', []):
    if 'inlineData' in part:
        with open('option-1.png', 'wb') as f:
            f.write(base64.b64decode(part['inlineData']['data']))
        break
"

# Repeat for option-2.png and option-3.png with the SAME prompt
```

Alternative using the generate_image.py helper if available:

```bash
uv run ~/.npm-global/lib/node_modules/clawdbot/skills/nano-banana-pro/scripts/generate_image.py \
  --prompt "[CONSTRUCTED PROMPT]" \
  --filename "option-1.png" --resolution 1K

uv run ~/.npm-global/lib/node_modules/clawdbot/skills/nano-banana-pro/scripts/generate_image.py \
  --prompt "[CONSTRUCTED PROMPT]" \
  --filename "option-2.png" --resolution 1K

uv run ~/.npm-global/lib/node_modules/clawdbot/skills/nano-banana-pro/scripts/generate_image.py \
  --prompt "[CONSTRUCTED PROMPT]" \
  --filename "option-3.png" --resolution 1K
```

### Step 4: Present Options

Show all 3 images labeled A, B, C:

- **A** [image]
- **B** [image]
- **C** [image]

**No descriptions. No commentary. Just A, B, C and the images.**

After the images, add one line: "Pick a letter, or tell me what to change."

### Step 5: Handle Response

The user will do one of these:

| User says | Action |
|-----------|--------|
| "B" | Done. B is the winner. Save/deliver it. |
| "B but warmer" | Refinement. Generate 3 new with feedback applied. |
| "none, try again" | Regenerate 3 with same prompt. |
| "make it 16:9" | Regenerate 3 with new aspect ratio. |
| "try watercolor style" | Regenerate 3 with style applied. |
| "combine A and C" | Use both descriptions as inspiration for 3 new. |

---

## Style Presets

When the user asks for a style (or you detect one in their prompt), append the style modifier to the prompt. The user can say "photorealistic", "photo style", "make it look real" - match to the closest preset.

### Available Styles

| Preset | Append to prompt |
|--------|-----------------|
| **photorealistic** | ", ultra-realistic photograph, natural lighting, shot on Canon EOS R5, 8K detail" |
| **illustration** | ", digital illustration, clean lines, vibrant colors, professional editorial illustration style" |
| **watercolor** | ", watercolor painting, soft washes, visible brush texture, paper grain, artistic watercolor technique" |
| **minimalist** | ", minimalist design, clean composition, ample negative space, simple geometric forms, muted palette" |
| **gradient** | ", smooth gradient background, modern gradient design, color transitions, contemporary digital art" |
| **retro** | ", retro vintage style, 1970s color palette, film grain, warm tones, nostalgic aesthetic" |
| **neon** | ", neon glow effect, cyberpunk lighting, dark background with bright neon colors, electric atmosphere" |
| **sketch** | ", pencil sketch, hand-drawn style, crosshatching, graphite on paper, detailed line work" |
| **anime** | ", anime art style, cel-shaded, vibrant colors, expressive features, Japanese animation aesthetic" |
| **pixel-art** | ", pixel art style, 16-bit retro game aesthetic, crisp pixels, limited color palette, nostalgic gaming" |
| **oil-painting** | ", oil painting on canvas, rich impasto texture, classical painting technique, gallery quality" |
| **3d-render** | ", 3D rendered, cinema 4D style, smooth materials, studio lighting, product visualization quality" |

### Style Usage

User: "make me a cat in a spacesuit, watercolor style"

Constructed prompt: "a cat in a spacesuit, watercolor painting, soft washes, visible brush texture, paper grain, artistic watercolor technique"

User: "make it neon instead"

Constructed prompt: "a cat in a spacesuit, neon glow effect, cyberpunk lighting, dark background with bright neon colors, electric atmosphere"

---

## Aspect Ratios

Default is 1:1 (square). User can request any of these:

| Name | Ratio | Use case | Config value |
|------|-------|----------|--------------|
| **Square** | 1:1 | Social posts, profile pics, icons | "1:1" |
| **Banner** | 16:9 | YouTube thumbnails, hero images, presentations | "16:9" |
| **Story** | 9:16 | Instagram/TikTok stories, phone wallpapers | "9:16" |
| **Thumbnail** | 4:3 | Blog posts, article headers, thumbnails | "4:3" |
| **Wide** | 3:2 | Photography standard, prints | "3:2" |
| **Ultrawide** | 21:9 | Cinematic, desktop wallpapers | "21:9" |

Pass the aspect ratio in the API call's `imageSizeOptions`:

```json
{
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "imageSizeOptions": {"aspectRatio": "16:9"}
  }
}
```

### Aspect Ratio Detection

Match user language to ratios:
- "banner", "wide", "landscape", "youtube" -> 16:9
- "story", "vertical", "phone", "portrait", "tall" -> 9:16
- "square", "post", "instagram" -> 1:1
- "thumbnail", "thumb" -> 4:3
- "wallpaper", "cinematic", "ultrawide" -> 21:9

---

## Iterative Refinement

This is the core loop. The user never has to start over - they build on what they like.

### Refinement Rules

1. When user says "B but [feedback]", take the ORIGINAL prompt and ADD their feedback
2. Generate 3 NEW images with the modified prompt
3. Present as A, B, C again (new set, old set is gone)
4. Track refinement depth for context (but don't limit it)

### Refinement Examples

**Round 1:**
- Prompt: "a mountain landscape at sunset"
- Show A, B, C

**Round 2 (user says "A but more dramatic sky"):**
- Prompt: "a mountain landscape at sunset, more dramatic sky, intense cloud formations"
- Show A, B, C

**Round 3 (user says "C but add a lake reflection"):**
- Prompt: "a mountain landscape at sunset, more dramatic sky, intense cloud formations, add a lake in the foreground with reflections"
- Show A, B, C

### Compound Feedback

If user gives multiple pieces of feedback, incorporate all of them:
- "B but warmer colors and add some birds" -> append "warmer color palette, birds flying in the sky"
- "A but make it night and add city lights" -> change "sunset" to "night" and append "city lights in the valley"

---

## Image Editing Mode

When the user provides an existing image and asks to modify it, switch to editing mode.

### How to Detect Edit Mode

- User uploads/references an image AND gives modification instructions
- User says "edit this", "change this", "modify this image"
- User references a previously generated option: "take B and add a rainbow"

### Edit API Call

For editing, include the source image in the request:

```bash
# Read the source image and base64 encode it
SOURCE_B64=$(base64 -i source-image.png)

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [
      {"inlineData": {"mimeType": "image/png", "data": "'$SOURCE_B64'"}},
      {"text": "[EDIT INSTRUCTION]"}
    ]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"]
    }
  }'
```

### Edit Rules

1. Still generate 3 variations of the edit
2. Each variation applies the edit differently (model randomness)
3. Present as A, B, C like normal
4. User can chain edits: "B" -> "now remove the background" -> "add a gradient behind it"

---

## Batch Mode

When the user gives multiple prompts at once, generate 3 for each.

### Detection

- User provides a numbered list of prompts
- User says "make these images" with multiple descriptions
- User says "I need images for: X, Y, Z"

### Batch Execution

For each prompt in the batch:
1. Generate 3 images (same parallel flow)
2. Label by prompt: "Prompt 1: A, B, C" / "Prompt 2: A, B, C"
3. User can pick winners per prompt independently

### Batch Example

**User:** "I need 3 sets: (1) a sunset beach, (2) a coffee shop interior, (3) an abstract pattern"

**Response:**

**Prompt 1: sunset beach**
- A [image] B [image] C [image]

**Prompt 2: coffee shop interior**
- A [image] B [image] C [image]

**Prompt 3: abstract pattern**
- A [image] B [image] C [image]

"Pick winners for each, or give feedback on any."

---

## Prompt Enhancement

When enabled (user says "enhance my prompt", "make it better", or "improve this"), auto-improve the prompt before generating.

### Enhancement Rules

1. Keep the user's core intent intact
2. Add specificity: lighting, composition, mood, detail level
3. Add quality keywords: "high quality", "detailed", "professional"
4. Don't change the subject or scene
5. Show the enhanced prompt to the user before generating

### Enhancement Example

**User prompt:** "a dog in a park"

**Enhanced:** "a golden retriever playing in a sunlit city park, afternoon light filtering through trees, shallow depth of field, vibrant green grass, candid moment, high detail"

**Always show:** "Enhanced prompt: [enhanced version]. Generating 3 options..."

The user can say "no, use my original" to skip enhancement.

---

## Error Recovery

### API Errors

| Error | Recovery |
|-------|----------|
| 429 (rate limit) | Wait 5 seconds, retry. If 2nd failure, wait 15 seconds. Max 3 retries. |
| 500 (server error) | Retry once after 3 seconds. If persistent, tell user to try again in a minute. |
| 400 (bad request) | Check prompt for issues. Simplify and retry. |
| Network timeout | Retry with 60-second timeout. |
| Empty response | Retry once. If still empty, report to user. |

### Content Policy

If the API returns a content policy block:
1. Tell the user: "That prompt was blocked by content policy. Try rephrasing."
2. Don't retry the same prompt
3. Don't try to work around the policy
4. Suggest a milder alternative if obvious

### Partial Failures

If only 1 or 2 of the 3 images succeed:
1. Show what you got: "Got 2 of 3 (one failed). Here are A and B:"
2. Offer to retry the failed one: "Want me to try for a third?"
3. Don't block the user from picking from what's available

---

## Complete Examples

### Example 1: Basic Generation

**User:** make me an image of a lighthouse on a cliff

**Agent generates 3 images with prompt "a lighthouse on a cliff", presents:**
- A [image]
- B [image]
- C [image]

Pick a letter, or tell me what to change.

**User:** B

**Agent:** Done. Saved as lighthouse-cliff.png.

---

### Example 2: Style + Refinement

**User:** create a cyberpunk city street, neon style

**Agent constructs prompt:** "a cyberpunk city street, neon glow effect, cyberpunk lighting, dark background with bright neon colors, electric atmosphere"

**Generates 3, presents A, B, C**

**User:** A but more rain and reflections

**Agent constructs prompt:** "a cyberpunk city street, neon glow effect, cyberpunk lighting, dark background with bright neon colors, electric atmosphere, heavy rain, wet street with neon reflections in puddles"

**Generates 3 new, presents A, B, C**

**User:** C, perfect

**Agent:** Done.

---

### Example 3: Aspect Ratio

**User:** I need a youtube thumbnail of a space battle

**Agent detects:** "youtube" -> 16:9 aspect ratio

**Constructs prompt:** "a space battle"
**Aspect ratio:** 16:9

**Generates 3 at 16:9, presents A, B, C**

---

### Example 4: Image Editing

**User:** [uploads photo] remove the background and make it transparent

**Agent:** Switches to edit mode. Generates 3 variations of the background removal. Presents A, B, C.

---

### Example 5: Batch Mode

**User:** I need social media graphics for my coffee brand:
1. A hero banner (wide)
2. An Instagram post (square)
3. A story image (vertical)

**Agent generates:**
- Banner (16:9): A, B, C
- Post (1:1): A, B, C
- Story (9:16): A, B, C

---

## Cross-Promotion

When relevant, mention complementary skills:

- After generating images: "Want to turn these into a video? Try `/remotion-server` for animated sequences."
- For image analysis: "Need to analyze or compare images in detail? `/xai` can break down composition and style."
- For multiple edits: "For complex multi-step editing workflows, chain edits: pick B -> edit -> pick C -> edit."

---

## Rules Summary

1. **Always 3 images** - Every generation produces 3 options. Never just 1.
2. **Parallel execution** - Generate all 3 concurrently for speed.
3. **User's words first** - Start with their exact prompt, append modifiers.
4. **Label A, B, C** - Clean labels. No descriptions unless asked.
5. **Feedback = 3 more** - Any edit request generates 3 new options.
6. **Detect style/ratio** - Match natural language to presets automatically.
7. **Show enhanced prompts** - If you enhance, show what you changed.
8. **Graceful errors** - Never leave the user with nothing. Show partial results.
9. **Track refinement** - Remember the chain of edits for context.
10. **Stay fast** - Parallel calls, minimal chatter, maximum images.

## API Key

Uses `GEMINI_API_KEY` from environment or openclaw config. Get one at [Google AI Studio](https://aistudio.google.com).
