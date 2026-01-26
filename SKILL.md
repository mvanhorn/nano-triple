---
name: image-picker
description: Generate 3 Nano Banana Pro prompt variations and let user pick their favorite before generating.
triggers:
  - image picker
  - pick image
  - image options
  - 3 options
  - three options
metadata:
  clawdbot:
    emoji: "🎨"
---

# Image Picker: 3 Options for Nano Banana Pro

When user wants to generate an image, create THREE distinct prompt variations and let them choose.

## Flow

### Step 1: User Describes What They Want

User says something like: "generate an image of a sunset over mountains"

### Step 2: Generate 3 Prompt Variations

Create THREE distinctly different prompts based on their request. Each should take a different creative direction:

1. **Option A** - Realistic/photographic approach
2. **Option B** - Artistic/stylized approach  
3. **Option C** - Creative/unexpected interpretation

### Step 3: Present Options with Inline Buttons

Use Telegram inline buttons so user can tap to choose:

```
🎨 Here are 3 prompt options for your image:

**A) Photorealistic**
[Full prompt text here - 2-3 sentences max]

**B) Artistic** 
[Full prompt text here - 2-3 sentences max]

**C) Creative**
[Full prompt text here - 2-3 sentences max]

Tap to generate:
```

Send with buttons:
```json
{
  "buttons": [[
    {"text": "🅰️ Photorealistic", "callback_data": "imgpick_a"},
    {"text": "🅱️ Artistic", "callback_data": "imgpick_b"},
    {"text": "🅲️ Creative", "callback_data": "imgpick_c"}
  ]]
}
```

### Step 4: When User Picks (callback received)

When you receive `imgpick_a`, `imgpick_b`, or `imgpick_c`:

1. Recall which prompt that was (from your previous message)
2. Generate the image using nano-banana-pro:

```bash
uv run ~/.npm-global/lib/node_modules/clawdbot/skills/nano-banana-pro/scripts/generate_image.py \
  --prompt "[THE SELECTED PROMPT]" \
  --filename "$(date +%Y-%m-%d-%H-%M-%S)-picked.png" \
  --resolution 1K
```

3. Send the MEDIA: line to deliver the image

## Example Interaction

**User:** make me an image of a cat wearing a top hat

**You (with buttons):**
```
🎨 Here are 3 prompt options:

**A) Photorealistic**
A distinguished gray tabby cat wearing a black silk top hat, studio portrait lighting, shallow depth of field, whiskers in sharp focus

**B) Artistic**
Whimsical watercolor illustration of a fluffy orange cat in an oversized vintage top hat, soft pastel background, storybook charm

**C) Creative**
A cat made entirely of clockwork gears and brass, wearing a steampunk top hat with goggles, Victorian laboratory setting
```

**User taps:** 🅱️ Artistic

**You:** Generate that prompt, send image.

## Rules

1. **Always 3 options** - No more, no less
2. **Distinct directions** - Each should feel meaningfully different
3. **Keep prompts concise** - 2-3 sentences max per option
4. **Use buttons** - Don't make user type "A" or "B"
5. **Remember context** - When callback arrives, recall the prompts from your message

## API Key

Uses `GEMINI_API_KEY` from environment or clawdbot config.
