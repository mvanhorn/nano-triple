---
name: nano-triple
version: "3.0.0"
description: "3 images, one prompt, instant A/B/C comparison. 4 models: Nano Banana 2 (fast/cheap), Nano Banana Pro (studio), Imagen 4 (text-to-image). Style presets, iterative refinement, mask-free inpainting, character consistency, 14 aspect ratios, 4K output."
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
  - upscale this image
  - inpaint this
  - extend this image
metadata:
  openclaw:
    emoji: "🎨"
    tags:
      - image-generation
      - nano-banana-2
      - nano-banana-pro
      - imagen-4
      - creative
      - parallel
      - ai-art
      - style-presets
      - iterative-refinement
      - image-editing
      - inpainting
      - outpainting
      - aspect-ratios
      - batch-generation
      - prompt-enhancement
      - visual-design
      - content-creation
      - variations
      - character-consistency
      - upscaling
      - 4k
---

# Nano Triple: 3 Images, Same Prompt, You Pick

Generate 3 image variations in parallel for every request. The user picks a winner or refines any version. Always 3 at a time - never just 1.

---

## Models

### Model Selection Guide

| Model | ID | Best for | Cost | Max Resolution | Editing |
|-------|----|----------|------|---------------|---------|
| **Nano Banana 2** (default) | `gemini-3.1-flash-image-preview` | Fast iteration, previews, most tasks | $0.067/img | 4K | Yes |
| **Nano Banana Pro** | `gemini-2.0-flash-exp` | Studio-quality final output | $0.134/img | 4K | Yes |
| **Imagen 4 Fast** | `imagen` (fast mode) | Cheapest text-to-image | $0.02/img | 2K | No |
| **Imagen 4 Ultra** | `imagen` (ultra mode) | Highest quality text-to-image | $0.06/img | 2K | No |

**Default model: Nano Banana 2** - fastest, cheapest, supports all editing features, 4K output.

Use `--quality pro` for Nano Banana Pro studio work.
Use `--quality imagen-fast` or `--quality imagen-ultra` for dedicated text-to-image (no editing support).

### Model Detection

Match user language to model:
- "fast", "quick", "cheap", "preview", "draft" -> Nano Banana 2 (default)
- "pro", "studio", "high quality", "production" -> Nano Banana Pro
- "imagen", "best quality", "ultra" -> Imagen 4 Ultra
- "cheapest", "budget" -> Imagen 4 Fast

---

## Core Flow

### Step 1: Parse the Request

Extract these from the user's message:
- **Prompt** - What they want to see
- **Style** - Any style preference (default: none/natural)
- **Aspect ratio** - Any size preference (default: 1:1 square)
- **Resolution** - Size preference (default: 1K)
- **Mode** - New generation, editing, inpainting, or outpainting
- **Model** - Which model to use (default: Nano Banana 2)
- **Refinement** - Are they giving feedback on a previous set?
- **Reference images** - Any character/object consistency references

### Step 2: Build the Generation Prompt

Start with the user's words. Apply style and aspect ratio modifiers if specified.

**Prompt construction rules:**
1. User's core description comes first
2. Style preset appended if requested
3. Never rewrite the user's intent - only append style/quality modifiers
4. For refinements, incorporate feedback into the base prompt

### Step 3: Generate 3 Images in Parallel

Run all 3 simultaneously. Same prompt, 3 calls. The model's inherent randomness produces 3 different results.

#### Nano Banana 2 (default)

```bash
# Generate all 3 in parallel (run these concurrently, not sequentially)
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "[CONSTRUCTED PROMPT]"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageSizeOptions": {
        "aspectRatio": "[ASPECT_RATIO]",
        "outputImageSize": "[RESOLUTION]"
      }
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

#### Nano Banana Pro

```bash
# Same as above but with gemini-2.0-flash-exp model ID
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash-exp:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [{"text": "[CONSTRUCTED PROMPT]"}]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageSizeOptions": {
        "aspectRatio": "[ASPECT_RATIO]",
        "outputImageSize": "[RESOLUTION]"
      }
    }
  }'
```

#### Imagen 4

```bash
# Imagen 4 uses a different endpoint structure
# Supports 1-4 images per call (use numberOfImages: 3 for triple mode)
curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/imagen:generateImages?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "[CONSTRUCTED PROMPT]",
    "config": {
      "numberOfImages": 3,
      "aspectRatio": "[ASPECT_RATIO]",
      "mode": "[fast|standard|ultra]"
    }
  }'
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
| "upscale B" | Run B through Imagen 4 upscaling endpoint. |
| "B in 4K" | Regenerate B at 4K resolution. |

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

## Aspect Ratios (14 supported)

Default is 1:1 (square). User can request any of these:

| Name | Ratio | Use case | Model support |
|------|-------|----------|---------------|
| **Square** | 1:1 | Social posts, profile pics, icons | All |
| **Portrait** | 2:3 | Standard portrait orientation | All |
| **Landscape** | 3:2 | Photography standard, prints | All |
| **Photo** | 4:3 | Blog posts, article headers | All |
| **Photo Wide** | 3:4 | Portrait photos, book covers | All |
| **Social** | 4:5 | Instagram feed posts | All |
| **Social Wide** | 5:4 | Landscape social posts | All |
| **Story** | 9:16 | Instagram/TikTok stories, phone wallpapers | All |
| **Banner** | 16:9 | YouTube thumbnails, hero images | All |
| **Cinematic** | 21:9 | Cinematic, desktop wallpapers | All |
| **Tall** | 1:4 | Tall banners, bookmarks | Nano Banana 2 only |
| **Ultra-tall** | 1:8 | Extreme vertical, scrolling content | Nano Banana 2 only |
| **Wide Strip** | 4:1 | Wide banners, headers | Nano Banana 2 only |
| **Ultra-wide Strip** | 8:1 | Extreme horizontal, panoramic banners | Nano Banana 2 only |

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
- "story", "vertical", "phone" -> 9:16
- "square", "post", "instagram" -> 1:1
- "thumbnail", "thumb" -> 4:3
- "wallpaper", "cinematic", "ultrawide" -> 21:9
- "portrait", "headshot" -> 2:3
- "tall", "bookmark" -> 1:4
- "panoramic", "strip" -> 4:1 or 8:1
- "social", "feed" -> 4:5

---

## Resolution Options

| Resolution | Size | Models | Use case |
|------------|------|--------|----------|
| **0.5K** (512px) | 512x512 | Nano Banana 2 only | Quick previews, drafts |
| **1K** (1024px) | 1024x1024 | All | Standard quality |
| **2K** (2048px) | 2048x2048 | All | High quality (Imagen 4 max) |
| **4K** (4096px) | 4096x4096 | Nano Banana 2, Pro | Ultra-high quality |

Default resolution: 1K. Pass resolution in `outputImageSize`:

```json
{
  "generationConfig": {
    "imageSizeOptions": {
      "aspectRatio": "1:1",
      "outputImageSize": "1K"
    }
  }
}
```

### Resolution Detection

Match user language:
- "preview", "draft", "quick", "low res" -> 0.5K (Nano Banana 2 only)
- "standard", default -> 1K
- "high res", "detailed" -> 2K
- "4K", "ultra", "print quality", "maximum" -> 4K

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

When the user provides an existing image and asks to modify it, switch to editing mode. Editing is supported by Nano Banana 2 and Pro only (not Imagen 4).

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
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=$GEMINI_API_KEY" \
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

## Mask-free Inpainting

Nano Banana 2 supports inpainting without masks. Describe what to change in text and the model identifies the region automatically.

### How It Works

Instead of providing a mask image, describe the change in natural language. The model figures out which part of the image to modify.

**User:** "Replace the sky with a sunset" (with uploaded image)
**Result:** Model identifies the sky region and replaces it - no mask needed.

### Inpainting Rules

1. Use the same edit API call (image + text instruction)
2. Be specific about WHAT to change and HOW
3. The model preserves everything not mentioned
4. Still generate 3 variations

### Inpainting Examples

- "Change the wall color to blue" - model finds the wall, repaints it
- "Replace the person's shirt with a striped one" - model identifies the shirt region
- "Remove the car from the background" - model fills in what was behind the car
- "Add glasses to the person" - model identifies the face and adds glasses

---

## Layout-aware Outpainting

Extend images beyond their original boundaries with intelligent scene continuation.

### How It Works

Provide the original image and describe how to extend it. The model understands the scene layout and generates coherent extensions.

```bash
SOURCE_B64=$(base64 -i source-image.png)

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [
      {"inlineData": {"mimeType": "image/png", "data": "'$SOURCE_B64'"}},
      {"text": "Extend this image to the right, continuing the scene naturally. Output at 16:9 aspect ratio."}
    ]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"],
      "imageSizeOptions": {"aspectRatio": "16:9"}
    }
  }'
```

### Outpainting Rules

1. Specify direction: "extend left", "extend up", "widen to 16:9"
2. The model maintains visual consistency with the original
3. Still generate 3 variations for user choice
4. Works best when target aspect ratio is wider/taller than source

---

## Character Consistency

Nano Banana 2 supports up to 14 reference images (10 object references + 4 character references) for maintaining consistent characters and objects across generations.

### How It Works

Provide reference images of a character or object, then generate new scenes with that same character/object.

```bash
REF1_B64=$(base64 -i character-ref-1.png)
REF2_B64=$(base64 -i character-ref-2.png)

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/gemini-3.1-flash-image-preview:generateContent?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "contents": [{"parts": [
      {"inlineData": {"mimeType": "image/png", "data": "'$REF1_B64'"}},
      {"inlineData": {"mimeType": "image/png", "data": "'$REF2_B64'"}},
      {"text": "Generate an image of this character riding a bicycle in a park. Keep the character looking exactly like the reference images."}
    ]}],
    "generationConfig": {
      "responseModalities": ["TEXT", "IMAGE"]
    }
  }'
```

### Character Consistency Rules

1. More reference images = better consistency (2-4 recommended)
2. Reference images should show the character from different angles
3. Describe the character explicitly in the prompt as well
4. Still generate 3 variations
5. Max 14 reference images per call (10 object + 4 character)

---

## Thought Signatures

Nano Banana 2 returns encrypted thought signatures in multi-turn editing sessions. These maintain context across edits without re-processing the full conversation.

### How It Works

1. The API response may include a `thoughtSignature` field
2. Pass it back in the next request to maintain editing context
3. SDKs handle this automatically - manual curl users should capture and replay it

```json
{
  "contents": [{"parts": [
    {"text": "[NEXT EDIT INSTRUCTION]"}
  ]}],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "thoughtSignature": "[SIGNATURE FROM PREVIOUS RESPONSE]"
  }
}
```

### Rules

1. Capture `thoughtSignature` from each response
2. Include it in the next request in the same editing session
3. If missing or expired, the edit still works but may lose context
4. Start a new session (no signature) for unrelated edits

---

## Google Image Search Grounding

Nano Banana 2 can ground image generation in real Google Image Search results, producing images informed by actual visual references.

### How to Enable

Add `groundingConfig` to the request:

```json
{
  "contents": [{"parts": [{"text": "[PROMPT]"}]}],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"]
  },
  "groundingConfig": {
    "googleSearchRetrieval": {}
  }
}
```

### When to Use

- User asks for something specific and real: "the Eiffel Tower at night"
- Product mockups: "a MacBook Pro on a wooden desk"
- Architectural styles: "a house in the style of Frank Lloyd Wright"
- Fashion: "a dress inspired by the 2025 Met Gala"

### Rules

1. Enable grounding when the user references real things, places, or styles
2. Don't enable for pure creative/abstract prompts
3. Grounding may increase latency slightly
4. Still generate 3 variations

---

## Configurable Thinking

Nano Banana 2 supports configurable thinking levels that control how much reasoning the model does before generating.

### Thinking Levels

| Level | Use case | Latency |
|-------|----------|---------|
| **none** | Simple prompts, fastest output | Lowest |
| **low** | Standard generation | Low |
| **medium** | Complex scenes, multiple elements | Medium |
| **high** | Detailed compositions, artistic direction | Higher |

### How to Set

```json
{
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"],
    "thinkingConfig": {
      "thinkingLevel": "medium"
    }
  }
}
```

### Default Behavior

- Default: `low` for speed
- Auto-upgrade to `medium` for complex prompts (multiple subjects, specific compositions)
- User can request: "think harder about this" -> `high`

---

## Imagen 4 Upscaling

Dedicated upscaling endpoint for Imagen 4. Takes an existing image and increases resolution at $0.003/image.

### API Call

```bash
SOURCE_B64=$(base64 -i input-image.png)

curl -s -X POST \
  "https://generativelanguage.googleapis.com/v1beta/models/imagen:upscaleImage?key=$GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "image": {"imageBytes": "'$SOURCE_B64'"},
    "config": {
      "upscaleFactor": 2
    }
  }'
```

### Rules

1. Upscaling is a separate step, not part of generation
2. User says "upscale B" or "make B higher res" -> upscale the selected image
3. Cost: $0.003/image (very cheap)
4. Can upscale any image, not just Imagen 4 output

---

## Batch API

For large jobs, use the Batch API for 50% cost reduction with 24-hour turnaround.

### When to Suggest

- User needs more than 10 images
- User says "I need a lot of images" or "batch job"
- Non-urgent work: "I need these by tomorrow"

### How It Works

1. Submit batch request with all prompts
2. Get a batch ID back
3. Check status periodically
4. Results available within 24 hours
5. 50% cost reduction on all models

### Rules

1. Only suggest batch mode when appropriate (large volume, not time-sensitive)
2. Always offer real-time as the default
3. Mention the 50% savings when suggesting batch

---

## Batch Mode (Real-time)

When the user gives multiple prompts at once, generate 3 for each in real-time.

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

### Example 1: Basic Generation (Nano Banana 2)

**User:** make me an image of a lighthouse on a cliff

**Agent generates 3 images with Nano Banana 2, prompt "a lighthouse on a cliff", presents:**
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

**Generates 3 with Nano Banana 2, presents A, B, C**

**User:** A but more rain and reflections

**Agent constructs prompt:** "a cyberpunk city street, neon glow effect, cyberpunk lighting, dark background with bright neon colors, electric atmosphere, heavy rain, wet street with neon reflections in puddles"

**Generates 3 new, presents A, B, C**

**User:** C, perfect

**Agent:** Done.

---

### Example 3: 4K Studio Output

**User:** I need a hero image for my website, pro quality, 4K

**Agent detects:** "pro" -> Nano Banana Pro, "4K" -> 4K resolution

**Generates 3 at 4K with Nano Banana Pro, presents A, B, C**

---

### Example 4: Mask-free Inpainting

**User:** [uploads photo] change the sky to a dramatic sunset

**Agent:** Uses Nano Banana 2 inpainting. No mask needed - the model identifies the sky region automatically. Generates 3 variations. Presents A, B, C.

---

### Example 5: Character Consistency

**User:** [uploads 3 reference images of a cartoon mascot] Put this character in a spaceship cockpit

**Agent:** Uses character consistency mode with 3 reference images. Generates 3 variations maintaining the mascot's appearance. Presents A, B, C.

---

### Example 6: Imagen 4 Text-to-Image

**User:** generate the highest quality image of a crystal cave, budget doesn't matter

**Agent detects:** "highest quality" -> Imagen 4 Ultra

**Generates 3 with Imagen 4 Ultra, presents A, B, C**

**User:** B, and upscale it

**Agent:** Upscales B using Imagen 4 upscaling endpoint ($0.003). Delivers high-res version.

---

### Example 7: Ultra-wide Ratios

**User:** make a panoramic banner of a mountain range, 8:1

**Agent detects:** "8:1" -> ultra-wide strip (Nano Banana 2 only)

**Generates 3 at 8:1 with Nano Banana 2, presents A, B, C**

---

### Example 8: Preview Mode

**User:** quick draft of a logo concept, keep it fast

**Agent detects:** "quick", "draft", "fast" -> 0.5K preview resolution

**Generates 3 at 512px with Nano Banana 2 (fastest, cheapest), presents A, B, C**

---

### Example 9: Outpainting

**User:** [uploads a portrait photo] extend this to landscape 16:9, fill in the background

**Agent:** Uses outpainting mode. Extends the image to 16:9 with intelligent scene continuation. Generates 3 variations. Presents A, B, C.

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
3. **Default to Nano Banana 2** - Fastest, cheapest, most features. Offer Pro for studio work.
4. **User's words first** - Start with their exact prompt, append modifiers.
5. **Label A, B, C** - Clean labels. No descriptions unless asked.
6. **Feedback = 3 more** - Any edit request generates 3 new options.
7. **Detect style/ratio/model** - Match natural language to presets automatically.
8. **Show enhanced prompts** - If you enhance, show what you changed.
9. **Graceful errors** - Never leave the user with nothing. Show partial results.
10. **Track refinement** - Remember the chain of edits for context.
11. **Stay fast** - Parallel calls, minimal chatter, maximum images.
12. **Thought signatures** - Capture and replay for multi-turn editing sessions.
13. **Suggest batch** - For large jobs, mention 50% savings with batch API.

## API Key

Uses `GEMINI_API_KEY` from environment or openclaw config. Get one at [Google AI Studio](https://aistudio.google.com).
