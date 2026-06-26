<script setup lang="ts">
import { ref } from 'vue';
import DitheredLogo from './components/DitheredLogo.vue';

const logoSrc = `${import.meta.env.BASE_URL}frog.svg`;

const gridSize = ref(200);
const scale = ref(0.5);
const dotScale = ref(1);
const invert = ref(true);
const cornerRadius = ref(0.2);
const gamma = ref(1);
const blur = ref(3.75);
const diffusionStrength = ref(1);
const contrast = ref(40);
</script>

<template>
  <a
    class="github-link"
    href="https://github.com/marr/dithered-logo-vue"
    aria-label="View source on GitHub"
  >
    <img src="/github.svg" alt="" width="24" height="24" />
  </a>

  <h1>Dithered Logo</h1>
  <p class="lede">
    Vue 3 port of the Reverse UI dithered logo — Floyd–Steinberg error diffusion
    with cursor repulsion and click ripples. Hover and tap the logo to interact.
  </p>

  <p class="hint">Move your pointer over the logo, or click to send a ripple.</p>

  <div class="demo-stage">
    <DitheredLogo
      :image-src="logoSrc"
      :grid-size="gridSize"
      :scale="scale"
      :dot-scale="dotScale"
      :invert="invert"
      :corner-radius="cornerRadius"
      :gamma="gamma"
      :blur="blur"
      :diffusion-strength="diffusionStrength"
      :contrast="contrast"
    />
  </div>

  <div class="controls">
    <label>
      <span>Grid size</span>
      <input v-model.number="gridSize" type="range" min="80" max="320" step="10" />
      <span class="value">{{ gridSize }}</span>
    </label>
    <label>
      <span>Scale</span>
      <input v-model.number="scale" type="range" min="0.2" max="1" step="0.05" />
      <span class="value">{{ scale.toFixed(2) }}</span>
    </label>
    <label>
      <span>Dot scale</span>
      <input v-model.number="dotScale" type="range" min="0.5" max="2" step="0.1" />
      <span class="value">{{ dotScale.toFixed(1) }}</span>
    </label>
    <label>
      <span>Corner radius</span>
      <input
        v-model.number="cornerRadius"
        type="range"
        min="0"
        max="0.5"
        step="0.05"
      />
      <span class="value">{{ cornerRadius.toFixed(2) }}</span>
    </label>
    <label>
      <span>Contrast</span>
      <input v-model.number="contrast" type="range" min="0" max="120" step="5" />
      <span class="value">{{ contrast }}</span>
    </label>
    <label>
      <span>Blur</span>
      <input v-model.number="blur" type="range" min="0" max="8" step="0.25" />
      <span class="value">{{ blur.toFixed(2) }}</span>
    </label>
    <label>
      <span>Diffusion</span>
      <input
        v-model.number="diffusionStrength"
        type="range"
        min="0"
        max="1.5"
        step="0.05"
      />
      <span class="value">{{ diffusionStrength.toFixed(2) }}</span>
    </label>
    <label>
      <span>Invert</span>
      <input v-model="invert" type="checkbox" />
      <span class="value">{{ invert ? 'on' : 'off' }}</span>
    </label>
  </div>

  <div class="footer-links">
    <a href="https://reverseui.com/components/dithered-logo">Reverse UI original</a>
    <a href="https://marr.github.io/">marr.github.io</a>
  </div>
</template>
