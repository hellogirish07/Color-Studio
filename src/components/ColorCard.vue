
<script setup>
import { Lock, Unlock, Copy } from "lucide-vue-next";

defineProps({
  color: String,
  locked: Boolean
});

const emit = defineEmits(["toggle-lock", "copy"]);
</script>

<template>
  <div class="color-card">
    <div
      class="color-preview"
      :style="{ backgroundColor: color }"
    ></div>

    <div class="color-info">
      <span>{{ color }}</span>

      <div class="color-actions">
        <button @click="emit('copy')">
          <Copy :size="16" />
        </button>

        <button @click="emit('toggle-lock')">
          <Lock v-if="locked" :size="16" />
          <Unlock v-else :size="16" />
        </button>
      </div>
    </div>
  </div>
</template>
