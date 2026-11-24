<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import { RefreshCw, Copy } from "lucide-vue-next";
import ColorCard from "./ColorCard.vue";

const colors = ref([]);
const lockedColors = ref([]);
const paletteType = ref("Random");

const paletteTypes = [
  "Random",
  "Monochrome",
  "Analogous",
  "Complementary",
  "Triadic",
];

// generate a random HEX color
function randomColor() {
  return (
    "#" +
    Math.floor(Math.random() * 16777215)
      .toString(16)
      .padStart(6, "0")
      .toUpperCase()
  );
}

// convert HSL to HEX
function hslToHex(h, s, l) {
  s /= 100;
  l /= 100;

  const k = (n) => (n + h / 30) % 12;
  const a = s * Math.min(l, 1 - l);

  const f = (n) => {
    const color = l - a * Math.max(-1, Math.min(k(n) - 3, 9 - k(n), 1));

    return Math.round(255 * color)
      .toString(16)
      .padStart(2, "0");
  };

  return `#${f(0)}${f(8)}${f(4)}`.toUpperCase();
}

// generate a color palette based on the selected harmony
function generateHarmony() {
  const baseHue = Math.floor(Math.random() * 360);

  const colors = [];

  for (let i = 0; i < 5; i++) {
    let hue = baseHue;
    let saturation = 70;
    let lightness = 50;

    switch (paletteType.value) {
      case "Monochrome":
        lightness = 20 + i * 15;
        saturation = 65;
        break;

      case "Analogous":
        hue = (baseHue + (i - 2) * 25 + 360) % 360;
        break;

      case "Complementary":
        hue = i < 3 ? baseHue : (baseHue + 180) % 360;
        lightness = 35 + (i % 3) * 15;
        break;

      case "Triadic":
        hue = (baseHue + (i % 3) * 120) % 360;
        lightness = 35 + Math.floor(i / 3) * 25;
        break;

      default:
        return Array.from({ length: 5 }, () => randomColor());
    }

    colors.push(hslToHex(hue, saturation, lightness));
  }

  return colors;
}

// generate a new palette, preserving locked colors
function generatePalette() {
  const newColors = generateHarmony();

  colors.value = newColors.map((color, index) => {
    return lockedColors.value[index] ? colors.value[index] : color;
  });
}

const toastMessage = ref("");
let toastTimer = null;
const toastBackground = ref("#FF6B4A");

// show a temporary toast notification
// function showToast(message) {
//   toastMessage.value = message;

//   clearTimeout(toastTimer);

//   toastTimer = setTimeout(() => {
//     toastMessage.value = "";
//   }, 1500);
// }

function showToast(message, background = colors.value[0]) {
  toastMessage.value = message;
  toastBackground.value = background;

  clearTimeout(toastTimer);

  toastTimer = setTimeout(() => {
    toastMessage.value = "";
  }, 1500);
}

function toggleLock(index) {
  lockedColors.value[index] = !lockedColors.value[index];
}

// copy color to clipboard
// async function copyColor(color) {
//   await navigator.clipboard.writeText(color);
//   showToast("Color copied!");
// }
async function copyColor(color) {
  await navigator.clipboard.writeText(color);
  showToast("Color copied!", color);
}

// copy CSS variables to clipboard
async function copyCSS() {
  const cssCode = `:root {
${colors.value
  .map((color, index) => `  --color-${index + 1}: ${color};`)
  .join("\n")}
}`;

  await navigator.clipboard.writeText(cssCode);
  showToast("CSS copied!");
}

// handle spacebar key press to generate a new palette
function handleKeyboard(event) {
  if (
    event.code === "Space" &&
    event.target.tagName !== "INPUT" &&
    event.target.tagName !== "TEXTAREA"
  ) {
    event.preventDefault();
    generatePalette();
  }
}

onMounted(() => {
  generatePalette();
  window.addEventListener("keydown", handleKeyboard);
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyboard);
});
</script>

<template>
  <section class="generator">
    <div class="generator-header">
      <div>
        <h1>Color Studio</h1>
        <p>Discover your next color combination.</p>
      </div>

      <button class="generate-btn" @click="generatePalette">
        <RefreshCw :size="18" />
        Generate
      </button>
    </div>

    <div class="palette-selector">
      <button
        v-for="type in paletteTypes"
        :key="type"
        :class="{ active: paletteType === type }"
        @click="
          paletteType = type;
          generatePalette();
        "
      >
        {{ type }}
      </button>
    </div>

    <div class="palette-grid">
      <ColorCard
        v-for="(color, index) in colors"
        :key="index"
        :color="color"
        :locked="lockedColors[index]"
        @toggle-lock="toggleLock(index)"
        @copy="copyColor(color)"
      />
    </div>

    <div class="css-export">
      <div class="css-export-header">
        <div>
          <h3>CSS Variables</h3>
          <p>Copy your palette as CSS custom properties.</p>
        </div>

        <button @click="copyCSS">
          <Copy :size="16" />
          Copy CSS
        </button>
      </div>

      <pre><code>:root {
<span
  v-for="(color, index) in colors"
  :key="index"
>  --color-{{ index + 1 }}: {{ color }};
</span>}</code></pre>
    </div>

    <p class="keyboard-hint">Press SPACE to generate a new palette</p>
  </section>

  <Transition name="toast">
    <div
      v-if="toastMessage"
      class="toast-notification"
      :style="{ backgroundColor: toastBackground }"
      role="status"
      aria-live="polite"
    >
      <span class="toast-icon">✓</span>
      {{ toastMessage }}
    </div>
  </Transition>
</template>
