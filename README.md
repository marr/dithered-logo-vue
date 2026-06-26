# Dithered Logo (Vue)

Vue 3 port of the [Reverse UI Dithered Logo](https://reverseui.com/components/dithered-logo) component — an interactive canvas logo rendered with Floyd–Steinberg error diffusion dithering and particle ripples.

## Usage

```vue
<script setup lang="ts">
import { DitheredLogo } from './components';
</script>

<template>
  <DitheredLogo
    image-src="/your-logo.svg"
    :grid-size="200"
    :scale="0.5"
    :dot-scale="1"
    :invert="true"
    :corner-radius="0.2"
    :gamma="1.0"
    :blur="3.75"
    :diffusion-strength="1.0"
  />
</template>
```

## Demo

```bash
npm install
npm run dev
```

## Props

| Prop | Default | Description |
|------|---------|-------------|
| `imageSrc` | — | Logo image URL (SVG or PNG) |
| `gridSize` | `200` | Dither grid resolution |
| `scale` | `0.5` | Logo size relative to container |
| `dotScale` | `1` | Dot size multiplier |
| `invert` | `true` | Invert dither mask |
| `cornerRadius` | `0.2` | Rounded mask radius (0–1) |
| `threshold` | `180` | Dither threshold |
| `contrast` | `0` | Contrast adjustment |
| `gamma` | `1.0` | Gamma correction |
| `blur` | `3.75` | Pre-dither blur |
| `diffusionStrength` | `1.0` | Error diffusion strength |
| `serpentine` | `true` | Serpentine scan order |

Hover and click/tap the canvas for cursor repulsion and ripple effects.
