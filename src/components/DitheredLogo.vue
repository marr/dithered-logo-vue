<script setup lang="ts">
import {
  computed,
  onMounted,
  onUnmounted,
  shallowRef,
  watch,
  type CSSProperties,
} from 'vue';
import {
  applyMaskInversion,
  DITHERED_LOGO_DEFAULTS,
  drawParticles,
  errorDiffusionDither,
  fetchImage,
  initParticles,
  stepParticles,
  toGrayscaleGrid,
  type ParticleSystem,
  type Ripple,
} from '../lib/ditheredLogoCore';

export interface DitheredLogoProps {
  imageSrc: string;
  gridSize?: number;
  scale?: number;
  dotScale?: number;
  invert?: boolean;
  cornerRadius?: number;
  threshold?: number;
  contrast?: number;
  gamma?: number;
  blur?: number;
  diffusionStrength?: number;
  serpentine?: boolean;
  style?: CSSProperties;
  class?: string;
}

const props = withDefaults(defineProps<DitheredLogoProps>(), {
  ...DITHERED_LOGO_DEFAULTS,
});

const canvasRef = shallowRef<HTMLCanvasElement | null>(null);
const systemRef = shallowRef<ParticleSystem | null>(null);
const isMobile = shallowRef(false);

const cursor = { x: 0, y: 0, active: false };
const ripples: Ripple[] = [];
let animFrame = 0;
let running = false;
let prevConfigKey = '';
let resizeTimer: ReturnType<typeof setTimeout> | null = null;
let lastW = 0;
let lastH = 0;
let teardownCanvas: (() => void) | null = null;
let teardownMobileQuery: (() => void) | null = null;

const containerStyle = computed<CSSProperties>(() => ({
  position: 'relative',
  width: '240px',
  height: '240px',
  ...(isMobile.value ? { transform: 'scale(0.8)' } : {}),
  ...props.style,
}));

const canvasStyle: CSSProperties = {
  display: 'block',
  width: '500px',
  height: '500px',
  position: 'absolute',
  top: '50%',
  left: '50%',
  transform: 'translate(-50%, -50%)',
  touchAction: 'none',
  cursor: 'default',
};

const configKey = computed(() =>
  JSON.stringify([
    props.imageSrc,
    props.gridSize,
    props.scale,
    props.dotScale,
    props.invert,
    props.cornerRadius,
    props.threshold,
    props.contrast,
    props.gamma,
    props.blur,
    props.diffusionStrength,
    props.serpentine,
    isMobile.value,
  ])
);

function startLoop() {
  if (running) return;
  running = true;

  const canvas = canvasRef.value;
  if (!canvas) return;

  const ctx = canvas.getContext('2d');
  if (!ctx) return;

  const dpr = window.devicePixelRatio || 1;

  const tick = () => {
    const sys = systemRef.value;
    if (!sys) {
      running = false;
      return;
    }

    const rect = canvas.getBoundingClientRect();
    const needsMore = stepParticles(
      sys,
      cursor.x,
      cursor.y,
      cursor.active,
      ripples,
      performance.now()
    );
    drawParticles(ctx, sys, props.invert, rect.width, rect.height, dpr);

    if (needsMore) {
      animFrame = requestAnimationFrame(tick);
    } else {
      running = false;
    }
  };

  animFrame = requestAnimationFrame(tick);
}

async function rebuild(src: string) {
  const canvas = canvasRef.value;
  if (!canvas || !src) return;

  try {
    const img = await fetchImage(src);
    const rect = canvas.getBoundingClientRect();

    const processed = toGrayscaleGrid(
      img,
      props.gridSize,
      props.contrast,
      props.gamma,
      props.blur
    );
    const { width: gw, height: gh } = processed;

    let positions = errorDiffusionDither(
      processed.grayscale,
      gw,
      gh,
      {
        threshold: props.threshold,
        serpentine: props.serpentine,
        diffusionStrength: props.diffusionStrength,
      },
      processed.alpha
    );

    if (props.invert) {
      positions = applyMaskInversion(
        positions,
        gw,
        gh,
        props.cornerRadius,
        processed.alpha
      );
    }

    const s = Math.max(
      0.5,
      (Math.min(rect.width, rect.height) * props.scale) / Math.max(gw, gh)
    );
    const ox = Math.round((rect.width - gw * s) / 2);
    const oy = Math.round((rect.height - gh * s) / 2);
    const ds = isMobile.value ? props.dotScale * 0.8 : props.dotScale;

    systemRef.value = initParticles(positions, s, ds, ox, oy);
    startLoop();
  } catch (err) {
    console.error('DitheredLogo: failed to process image', err);
  }
}

function setupCanvas() {
  const canvas = canvasRef.value;
  if (!canvas) return () => {};

  const ctx = canvas.getContext('2d');
  if (!ctx) return () => {};

  const dpr = window.devicePixelRatio || 1;

  const handleResize = () => {
    const rect = canvas.getBoundingClientRect();
    canvas.width = rect.width * dpr;
    canvas.height = rect.height * dpr;

    const sys = systemRef.value;
    if (sys) {
      drawParticles(ctx, sys, props.invert, rect.width, rect.height, dpr);
    }

    const w = Math.round(rect.width);
    const h = Math.round(rect.height);
    if (lastW !== 0 && (w !== lastW || h !== lastH)) {
      if (resizeTimer) clearTimeout(resizeTimer);
      resizeTimer = setTimeout(() => {
        void rebuild(props.imageSrc);
      }, 200);
    }
    lastW = w;
    lastH = h;
  };

  handleResize();

  const ro = new ResizeObserver(handleResize);
  ro.observe(canvas);

  const onPointerMove = (e: PointerEvent) => {
    const rect = canvas.getBoundingClientRect();
    cursor.x = e.clientX - rect.left;
    cursor.y = e.clientY - rect.top;
    cursor.active = true;
    startLoop();
  };

  const onPointerLeave = (e: PointerEvent) => {
    if (e.pointerType !== 'mouse') return;
    cursor.active = false;
    startLoop();
  };

  const onPointerCancel = () => {
    cursor.active = false;
    startLoop();
  };

  const onPointerUp = (e: PointerEvent) => {
    const rect = canvas.getBoundingClientRect();
    ripples.push({
      x: e.clientX - rect.left,
      y: e.clientY - rect.top,
      start: performance.now(),
    });
    if (e.pointerType !== 'mouse') cursor.active = false;
    startLoop();
  };

  canvas.addEventListener('pointermove', onPointerMove);
  canvas.addEventListener('pointerleave', onPointerLeave);
  canvas.addEventListener('pointercancel', onPointerCancel);
  canvas.addEventListener('pointerup', onPointerUp);

  return () => {
    cancelAnimationFrame(animFrame);
    running = false;
    if (resizeTimer) clearTimeout(resizeTimer);
    ro.disconnect();
    canvas.removeEventListener('pointermove', onPointerMove);
    canvas.removeEventListener('pointerleave', onPointerLeave);
    canvas.removeEventListener('pointercancel', onPointerCancel);
    canvas.removeEventListener('pointerup', onPointerUp);
  };
}

function mountCanvas() {
  teardownCanvas?.();
  teardownCanvas = setupCanvas();
}

watch(configKey, (key) => {
  if (key === prevConfigKey) return;
  prevConfigKey = key;
  void rebuild(props.imageSrc);
});

watch(
  () => [props.invert, props.imageSrc] as const,
  () => {
    mountCanvas();
  }
);

onMounted(() => {
  const mq = window.matchMedia('(max-width: 640px)');
  isMobile.value = mq.matches;

  const onMqChange = (e: MediaQueryListEvent) => {
    isMobile.value = e.matches;
  };

  mq.addEventListener('change', onMqChange);
  teardownMobileQuery = () => mq.removeEventListener('change', onMqChange);

  mountCanvas();

  const key = configKey.value;
  prevConfigKey = key;
  void rebuild(props.imageSrc);
});

onUnmounted(() => {
  teardownCanvas?.();
  teardownMobileQuery?.();
  cancelAnimationFrame(animFrame);
  running = false;
});
</script>

<script lang="ts">
export default {
  name: 'DitheredLogo',
};
</script>

<template>
  <div :class="props.class" :style="containerStyle">
    <canvas ref="canvasRef" :style="canvasStyle" />
  </div>
</template>
