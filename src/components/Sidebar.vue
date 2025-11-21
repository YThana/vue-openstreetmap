<script setup>
const props = defineProps({
  item: { type: Object, default: null },
})
const emit = defineEmits(['close'])
</script>

<template>
  <aside class="sidebar" :class="{ open: !!item }">
    <div class="header">
      <div class="title">Details</div>
      <button class="close" @click="emit('close')">×</button>
    </div>
    <div v-if="item" class="body">
      <div class="name">{{ item.title }}</div>
      <div class="desc">{{ item.description }}</div>
      <div class="meta">Source: <strong>{{ item.sourceId }}</strong></div>
      <div class="coords">{{ item.lat.toFixed(5) }}, {{ item.lng.toFixed(5) }}</div>
    </div>
    <div v-else class="empty">Select a marker to see details.</div>
  </aside>
</template>

<style scoped>
.sidebar {
  width: 0;
  transition: width 0.2s ease;
  overflow: hidden;
  border-left: 1px solid #e5e7eb;
  background: #fafafa;
}
.sidebar.open {
  width: 320px;
}
.header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 12px;
  border-bottom: 1px solid #eee;
}
.title {
  font-weight: 600;
}
.close {
  background: transparent;
  border: none;
  font-size: 20px;
  line-height: 1;
  cursor: pointer;
}
.body {
  padding: 12px;
  display: grid;
  gap: 8px;
}
.name { font-weight: 600; }
.desc { color: #444; }
.meta, .coords { color: #666; font-size: 12px; }
.empty { padding: 12px; color: #777; }
</style>
