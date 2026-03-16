# Nano Triple

3 images, one prompt, instant A/B/C comparison. Pick the winner or refine any version.

## Install

```
clawhub install nano-triple
```

Or copy `SKILL.md` to `~/.openclaw/skills/nano-triple/SKILL.md`

## What's New in v3.0.0

- **Nano Banana 2** - New default model. Faster, cheaper ($0.067/img), 4K output, ultra-wide ratios
- **Imagen 4** - Text-to-image from $0.02/img (Fast) to $0.06/img (Ultra), plus upscaling at $0.003/img
- **14 aspect ratios** - Including ultra-wide 1:8 and 8:1 (Nano Banana 2 exclusive)
- **Mask-free inpainting** - Describe changes in text, no mask needed
- **Layout-aware outpainting** - Extend images with intelligent scene continuation
- **Character consistency** - Up to 14 reference images for consistent characters/objects
- **Google Image Search grounding** - Generation informed by real search results
- **Configurable thinking** - Control reasoning depth before generating
- **0.5K preview mode** - 512px drafts for ultra-fast iteration
- **Batch API** - 50% cost reduction with 24hr turnaround for large jobs

## Models

| Model | Cost | Max Res | Editing | Best for |
|-------|------|---------|---------|----------|
| Nano Banana 2 (default) | $0.067/img | 4K | Yes | Fast iteration, most tasks |
| Nano Banana Pro | $0.134/img | 4K | Yes | Studio-quality output |
| Imagen 4 Fast | $0.02/img | 2K | No | Cheapest text-to-image |
| Imagen 4 Ultra | $0.06/img | 2K | No | Highest quality text-to-image |

## Features

- **3 parallel variations** - Same prompt, three distinct takes from the model's natural randomness
- **4 models** - Nano Banana 2, Nano Banana Pro, Imagen 4 Fast, Imagen 4 Ultra
- **Style presets** - Photorealistic, illustration, watercolor, minimalist, gradient, retro, neon, sketch, anime, pixel-art, oil-painting, 3d-render
- **14 aspect ratios** - 1:1, 1:4, 1:8, 2:3, 3:2, 3:4, 4:1, 4:3, 4:5, 5:4, 8:1, 9:16, 16:9, 21:9
- **4 resolutions** - 0.5K (preview), 1K, 2K, 4K
- **Iterative refinement** - "I like B but make it more colorful" generates 3 new variations
- **Image editing** - Edit existing images with natural language
- **Mask-free inpainting** - Change parts of an image by describing what to change
- **Outpainting** - Extend images beyond their borders
- **Character consistency** - Maintain consistent characters across generations with reference images
- **Upscaling** - Imagen 4 upscaling at $0.003/image
- **Batch API** - 50% cost reduction for large jobs

## Requirements

- `GEMINI_API_KEY` - Get one at [Google AI Studio](https://aistudio.google.com)

## License

MIT
