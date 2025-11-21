<script setup>
import { computed } from 'vue'

const props = defineProps({
  sources: { type: Array, required: true },
  selected: { type: Array, default: () => [] },
})
const emit = defineEmits(['update:selected'])

const model = computed({
  get: () => props.selected,
  set: (v) => emit('update:selected', v),
})
</script>

<template>
  <div class="selector">
    <label class="label">Sources</label>
    <div class="options">
      <label v-for="s in sources" :key="s.id" class="option">
        <input type="checkbox" :value="s.id" v-model="model" />
        <span class="swatch" :style="{ backgroundColor: s.color }" />
        {{ s.name }}
      </label>
    </div>
  </div>
</template>

<style scoped>
.selector {
  display: flex;
  align-items: center;
  gap: 12px;
}
.label {
  font-weight: 600;
}
.options {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}
.option {
  display: inline-flex;
  align-items: center;
  gap: 6px;
}
.swatch {
  width: 12px;
  height: 12px;
  border-radius: 2px;
  display: inline-block;
  border: 1px solid #ccc;
}
</style>
