<template>
  <div class="w-full max-w-full overflow-auto border rounded bg-white dark:bg-gray-800">
    <canvas
      ref="canvas"
      width="800"
      height="600"
      class="block bg-gray-50 dark:bg-gray-900"
      @click="handleCanvasClick"
    ></canvas>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from "vue";

const props = defineProps({
  points: Array,
  highlightedVertex: Number,
});

const emit = defineEmits(["add-point"]);
const canvas = ref(null);

const drawPolygon = () => {
  const ctx = canvas.value?.getContext("2d");
  if (!ctx) return;

  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height);

  if (!props.points.length) return;

  ctx.beginPath();
  ctx.moveTo(props.points[0].x, props.points[0].y);
  for (let i = 1; i < props.points.length; i++) {
    ctx.lineTo(props.points[i].x, props.points[i].y);
  }

  if (props.points.length > 2) ctx.closePath();

  ctx.strokeStyle = "#3498db";
  ctx.lineWidth = 2;
  ctx.stroke();

  props.points.forEach((point, index) => {
    ctx.beginPath();
    ctx.arc(point.x, point.y, 5, 0, Math.PI * 2);
    ctx.fillStyle = index === props.highlightedVertex ? "red" : "#2c3e50";
    ctx.fill();
  });
};

const handleCanvasClick = (event) => {
  const rect = canvas.value.getBoundingClientRect();
  emit("add-point", {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top,
  });
};

watch(() => [props.points, props.highlightedVertex], drawPolygon, { deep: true });
onMounted(drawPolygon);
</script>
