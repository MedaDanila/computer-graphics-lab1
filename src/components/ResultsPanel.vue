<template>
  <div class="space-y-4">
    <h2 class="text-xl font-semibold">Результаты</h2>

    <div class="space-y-2">
      <div class="flex justify-between">
        <span>Вершин:</span>
        <span class="font-bold">{{ points.length }}</span>
      </div>

      <div class="flex justify-between">
        <span>Минимальность:</span>
        <span :class="isMinimal ? 'text-green-500' : 'text-red-500'">
          {{ isMinimal ? "Да" : "Нет" }}
        </span>
      </div>
    </div>

    <div
      v-if="badVertices.length"
      class="mt-4 p-3 bg-red-100 dark:bg-red-900 rounded"
    >
      <h3 class="font-medium mb-1">Проблемные вершины:</h3>
      <div class="flex flex-wrap gap-1">
        <span
          v-for="v in badVertices"
          :key="v"
          class="px-2 py-1 bg-red-200 dark:bg-red-800 rounded text-sm"
        >
          #{{ v + 1 }}
        </span>
      </div>
    </div>

    <div v-if="angles.length" class="mt-4">
      <h3 class="font-medium mb-2">Углы между рёбрами:</h3>
      <div class="max-h-64 overflow-y-auto pr-1">
        <div class="grid grid-cols-3 gap-2">
          <div
            v-for="(angle, i) in angles"
            :key="i"
            class="p-2 rounded text-center"
            :class="
              angle < 10 || angle > 170
                ? 'bg-red-100 dark:bg-red-900'
                : 'bg-gray-100 dark:bg-gray-700'
            "
          >
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
