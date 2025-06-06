<template>
  <div class="results-panel">
    <h2 class="panel-title">Результаты</h2>
    <div class="results-block">
      <div class="results-row">
        <span>Вершин:</span>
        <span class="font-bold">{{ points.length }}</span>
      </div>
      <div class="results-row">
        <span>Минимальность:</span>
        <span :class="isMinimal ? 'text-green' : 'text-red'">{{ isMinimal ? "Да" : "Нет" }}</span>
      </div>
    </div>
    <div v-if="badVertices.length" class="bad-vertices">
      <h3 class="font-medium mb-1">Проблемные вершины:</h3>
      <div class="bad-vertices-list">
        <span v-for="v in badVertices" :key="v" class="bad-vertex">#{{ v + 1 }}</span>
      </div>
    </div>
    <div v-if="angles.length" class="angles-block">
      <h3 class="font-medium mb-2">Углы между рёбрами:</h3>
      <div class="angles-list">
        <div class="angles-grid">
          <div v-for="(angle, i) in angles" :key="i" :class="['angle-cell', angle < 10 || angle > 170 ? 'angle-bad' : 'angle-ok']">
            <div>Вершина #{{ i + 1 }}</div>
            <div class="font-bold">{{ angle.toFixed(1) }}°</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed } from "vue";

const props = defineProps({
  points: Array,
  isMinimal: Boolean,
  badVertices: Array,
});

const angles = computed(() => {
  if (props.points.length < 3) return [];

  return props.points.map((_, i) => {
    const a = props.points[(i - 1 + props.points.length) % props.points.length];
    const b = props.points[i];
    const c = props.points[(i + 1) % props.points.length];

    const ab = { x: b.x - a.x, y: b.y - a.y };
    const bc = { x: c.x - b.x, y: c.y - b.y };

    const dot = ab.x * bc.x + ab.y * bc.y;
    const magAB = Math.hypot(ab.x, ab.y);
    const magBC = Math.hypot(bc.x, bc.y);

    return magAB === 0 || magBC === 0
      ? 0
      : Math.acos(dot / (magAB * magBC)) * (180 / Math.PI);
  });
});
</script>

<style scoped>
.results-panel {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}
.panel-title {
  font-size: 1.25rem;
  font-weight: 600;
}
.results-block {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}
.results-row {
  display: flex;
  justify-content: space-between;
}
.text-green { color: #22c55e; }
.text-red { color: #ef4444; }
.bad-vertices {
  margin-top: 1rem;
  padding: 0.75rem;
  background: #fee2e2;
  border-radius: 8px;
}
.bad-vertices-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.25rem;
}
.bad-vertex {
  padding: 0.25rem 0.5rem;
  background: #fecaca;
  border-radius: 8px;
  font-size: 0.875rem;
}
.angles-block {
  margin-top: 1rem;
}
.angles-list {
  max-height: 16rem;
  overflow-y: auto;
  padding-right: 0.25rem;
}
.angles-grid {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 0.5rem;
}
.angle-cell {
  padding: 0.5rem;
  border-radius: 8px;
  text-align: center;
}
.angle-bad {
  background: #fee2e2;
}
.angle-ok {
  background: #f3f4f6;
}
.font-bold { font-weight: bold; }
.font-medium { font-weight: 500; }
.mb-1 { margin-bottom: 0.25rem; }
.mb-2 { margin-bottom: 0.5rem; }
</style>
