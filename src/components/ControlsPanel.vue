<template>
  <div class="controls-panel">
    <h2 class="panel-title">Управление</h2>
    <div class="controls-grid">
      <button @click="addRandomPoint" class="btn btn-green">+ Случайная точка</button>
      <button @click="clearPoints" class="btn btn-red">Очистить</button>
    </div>
    <button @click="minimizePolygon" :disabled="points?.length < 3" class="btn btn-blue w-full">Минимизировать</button>
    <button @click="minimizePolygonAnimated" :disabled="animationActive || points?.length < 3" class="btn btn-purple w-full">{{ animationActive ? "Идёт анимация..." : "Минимизировать (анимация)" }}</button>
  </div>
</template>

<script setup>
const props = defineProps({
  points: {
    type: Array,
    required: true,
  },
  animationActive: {
    type: Boolean,
    required: true,
  },
});

const emit = defineEmits([
  "add-random-point",
  "minimize-polygon",
  "minimize-animated",
  "clear-points",
]);

const addRandomPoint = () => {
  emit("add-random-point", {
    x: Math.random() * 500 + 50,
    y: Math.random() * 300 + 50,
  });
};

const minimizePolygon = () => emit("minimize-polygon");
const clearPoints = () => emit("clear-points");
const minimizePolygonAnimated = () => emit("minimize-animated");
</script>

<style scoped>
.controls-panel {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.panel-title {
  font-size: 1.25rem;
  font-weight: 600;
}
.controls-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;
}
.btn {
  padding: 0.5rem 1rem;
  border-radius: 8px;
  color: #fff;
  font-weight: 500;
  border: none;
  cursor: pointer;
  transition: background 0.2s;
}
.btn-green { background: #16a34a; }
.btn-green:hover { background: #15803d; }
.btn-red { background: #dc2626; }
.btn-red:hover { background: #b91c1c; }
.btn-blue { background: #2563eb; }
.btn-blue:hover { background: #1d4ed8; }
.btn-purple { background: #9333ea; }
.btn-purple:hover { background: #7e22ce; }
.btn:disabled, .btn[disabled] {
  background: #9ca3af;
  cursor: not-allowed;
  opacity: 0.6;
}
.w-full { width: 100%; }
</style>
