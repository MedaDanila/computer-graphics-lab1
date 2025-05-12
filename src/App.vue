<template>
  <div
    class="min-h-screen bg-gray-100 dark:bg-gray-900 text-gray-900 dark:text-white transition-colors"
  >
    <div class="container mx-auto p-4">
      <h1 class="text-3xl font-bold mb-6 text-center">Минимизация полигона</h1>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        <!-- Левая панель: Управление -->
        <ControlsPanel
          @add-random-point="addPoint"
          @minimize-polygon="minimizePolygon"
          @minimize-animated="minimizePolygonAnimated"
          @clear-points="clearPoints"
          :animation-active="animationActive"
          class="bg-white dark:bg-gray-800 p-4 rounded-lg shadow"
        />

        <!-- Центр: Холст -->
        <div
          class="relative bg-white dark:bg-gray-800 rounded-lg shadow overflow-hidden"
        >
          <PolygonCanvas
            :points="polygonPoints"
            :highlighted-vertex="currentStep"
            class="border border-gray-200 dark:border-gray-700"
            @add-point="addPoint"
          />

          <!-- Заглушка -->
          <div
            v-if="polygonPoints.length < 2"
            class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-20"
          >
            <div
              class="text-center p-6 bg-white dark:bg-gray-700 rounded-lg shadow-lg"
            >
              <svg
                class="w-12 h-12 mx-auto text-gray-400"
                fill="none"
                stroke="currentColor"
                viewBox="0 0 24 24"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M12 6v6m0 0v6m0-6h6m-6 0H6"
                ></path>
              </svg>
              <p class="mt-2 font-medium">Добавьте минимум 2 точки</p>
              <button
                @click="addRandomPoints(3)"
                class="mt-4 px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 transition"
              >
                Сгенерировать случайный полигон
              </button>
            </div>
          </div>
        </div>

        <!-- Правая панель: Результаты -->
        <ResultsPanel
          :points="polygonPoints"
          :is-minimal="isMinimal"
          :bad-vertices="badVertices"
          class="bg-white dark:bg-gray-800 p-4 rounded-lg shadow"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import ControlsPanel from "./components/ControlsPanel.vue";
import PolygonCanvas from "./components/PolygonCanvas.vue";
import ResultsPanel from "./components/ResultsPanel.vue";

const polygonPoints = ref([]);
const animationActive = ref(false);
const currentStep = ref(-1);

const addPoint = (point) => {
  polygonPoints.value.push(point);
};

const addRandomPoints = (count = 3) => {
  for (let i = 0; i < count; i++) {
    addPoint({ x: Math.random() * 500 + 50, y: Math.random() * 300 + 50 });
  }
};

const clearPoints = () => {
  polygonPoints.value = [];
  currentStep.value = -1;
};

const badVertices = computed(() => {
  const bad = [];
  const n = polygonPoints.value.length;
  if (n < 3) return bad;

  for (let i = 0; i < n; i++) {
    const a = polygonPoints.value[i];
    const b = polygonPoints.value[(i + 1) % n];
    const c = polygonPoints.value[(i + 2) % n];

    const area = Math.abs(
      (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)
    );
    const scale =
      1e-6 *
      (Math.abs(a.x) +
        Math.abs(a.y) +
        Math.abs(b.x) +
        Math.abs(b.y) +
        Math.abs(c.x) +
        Math.abs(c.y));

    if (area < scale) {
      bad.push((i + 1) % n);
    }
  }

  return bad;
});

const isMinimal = computed(() => badVertices.value.length === 0);

const minimizePolygon = () => {
  if (polygonPoints.value.length < 3) return;
  let changed = true;
  while (changed) {
    changed = false;
    for (let i = 0; i < polygonPoints.value.length; i++) {
      const a = polygonPoints.value[i];
      const b = polygonPoints.value[(i + 1) % polygonPoints.value.length];
      const c = polygonPoints.value[(i + 2) % polygonPoints.value.length];

      const area = Math.abs(
        (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)
      );
      const scale =
        1e-6 *
        (Math.abs(a.x) +
          Math.abs(a.y) +
          Math.abs(b.x) +
          Math.abs(b.y) +
          Math.abs(c.x) +
          Math.abs(c.y));

      if (area < scale) {
        polygonPoints.value.splice((i + 1) % polygonPoints.value.length, 1);
        changed = true;
        break;
      }
    }
  }
};

const minimizePolygonAnimated = async () => {
  if (polygonPoints.value.length < 3) return;
  animationActive.value = true;
  let changed = true;
  while (changed) {
    changed = false;
    for (let i = 0; i < polygonPoints.value.length; i++) {
      const a = polygonPoints.value[i];
      const b = polygonPoints.value[(i + 1) % polygonPoints.value.length];
      const c = polygonPoints.value[(i + 2) % polygonPoints.value.length];

      const area = Math.abs(
        (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x)
      );
      const scale =
        1e-6 *
        (Math.abs(a.x) +
          Math.abs(a.y) +
          Math.abs(b.x) +
          Math.abs(b.y) +
          Math.abs(c.x) +
          Math.abs(c.y));

      if (area < scale) {
        currentStep.value = (i + 1) % polygonPoints.value.length;
        await new Promise((resolve) => setTimeout(resolve, 800));
        polygonPoints.value.splice(currentStep.value, 1);
        changed = true;
        break;
      }
    }
  }
  currentStep.value = -1;
  animationActive.value = false;
};
</script>

<style scoped>
canvas {
  touch-action: none;
}
</style>
