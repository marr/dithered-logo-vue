<script setup lang="ts">
import DitheredLogo from './components/DitheredLogo.vue';
</script>

<template>
  <main
    :style="{
      minHeight: '100vh',
      display: 'grid',
      placeItems: 'center',
      background: '#f5f5f5',
    }"
  >
    <DitheredLogo
      image-src="/frog.svg"
      :grid-size="200"
      :scale="0.5"
      :dot-scale="1"
      :invert="true"
      :corner-radius="0.2"
      :gamma="1.0"
      :blur="3.75"
      :diffusion-strength="1.0"
    />
  </main>
</template>
