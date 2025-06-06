<template>
  <div class="canvas-panel">
    <canvas
      ref="canvas"
      width="800"
      height="600"
      class="canvas-block"
      @mousedown="onMouseDown"
      @mousemove="onMouseMove"
      @mouseup="onMouseUp"
      @mouseleave="onMouseUp"
      @contextmenu.prevent="onRightClick"
    ></canvas>
  </div>
</template>

<script setup>
import { ref, watch, onMounted, computed } from "vue";

const props = defineProps({
  points: Array,
  highlightedVertex: Number,
  badVertices: Array // Передавать из App.vue
});
const emit = defineEmits(["add-point", "close-polygon", "move-point"]);
const canvas = ref(null);
const dragging = ref(false);
const dragIndex = ref(-1);

const drawGrid = (ctx, width, height) => {
  ctx.save();
  ctx.strokeStyle = "#e5e7eb";
  ctx.lineWidth = 1;
  for (let x = 0; x <= width; x += 40) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
    ctx.stroke();
  }
  for (let y = 0; y <= height; y += 40) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
    ctx.stroke();
  }
  ctx.restore();
};

const drawPolygon = () => {
  const ctx = canvas.value?.getContext("2d");
  if (!ctx) return;
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);
  drawGrid(ctx, canvas.value.width, canvas.value.height);
  if (!props.points.length) return;
  // Draw polygon
  ctx.beginPath();
  ctx.moveTo(props.points[0].x, props.points[0].y);
  for (let i = 1; i < props.points.length; i++) {
    ctx.lineTo(props.points[i].x, props.points[i].y);
  }
  if (props.points.length > 2) ctx.closePath();
  ctx.strokeStyle = "#3498db";
  ctx.lineWidth = 2;
  ctx.stroke();
  // Draw vertices
  props.points.forEach((point, index) => {
    ctx.beginPath();
    ctx.arc(point.x, point.y, 7, 0, Math.PI * 2);
    if (props.badVertices && props.badVertices.includes(index)) {
      ctx.strokeStyle = "#e53935";
      ctx.lineWidth = 2;
      ctx.fillStyle = "#fff";
      ctx.stroke();
    } else {
      ctx.fillStyle = index === props.highlightedVertex ? "red" : "#2c3e50";
      ctx.fill();
    }
  });
};

const getVertexAt = (x, y) => {
  for (let i = 0; i < props.points.length; i++) {
    const p = props.points[i];
    if (Math.hypot(p.x - x, p.y - y) < 10) return i;
  }
  return -1;
};

const onMouseDown = (e) => {
  const rect = canvas.value.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  const idx = getVertexAt(x, y);
  if (e.button === 0) {
    if (idx !== -1) {
      dragging.value = true;
      dragIndex.value = idx;
    } else {
      emit("add-point", { x, y });
    }
  }
};
const onMouseMove = (e) => {
  if (!dragging.value) return;
  const rect = canvas.value.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  emit("move-point", { index: dragIndex.value, x, y });
};
const onMouseUp = () => {
  dragging.value = false;
  dragIndex.value = -1;
};
const onRightClick = (e) => {
  emit("close-polygon");
};

watch(() => [props.points, props.highlightedVertex, props.badVertices], drawPolygon, { deep: true });
onMounted(drawPolygon);
</script>

<style scoped>
.canvas-panel {
  width: 100%;
  max-width: 100%;
  overflow: auto;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  background: #fff;
}
body.dark .canvas-panel {
  background: #1f2937;
}
.canvas-block {
  display: block;
  background: #f9fafb;
  border-radius: 12px;
}
body.dark .canvas-block {
  background: #111827;
}
</style>
