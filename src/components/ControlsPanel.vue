<template>
  <div class="space-y-4">
    <h2 class="text-xl font-semibold">Управление</h2>

    <div class="grid grid-cols-2 gap-2">
      <button
        @click="addRandomPoint"
        class="px-4 py-2 bg-green-600 text-white rounded hover:bg-green-700 transition"
      >
        + Случайная точка
      </button>

      <button
        @click="clearPoints"
        class="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition"
      >
        Очистить
      </button>
    </div>

    <button
      @click="minimizePolygon"
      :disabled="points?.length < 3"
      class="w-full px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:bg-gray-400 transition"
    >
      Минимизировать
    </button>

    <button
      @click="minimizePolygonAnimated"
      :disabled="animationActive || points?.length < 3"
      class="w-full px-4 py-2 bg-purple-600 text-white rounded hover:bg-purple-700 disabled:bg-gray-400 transition"
    >
      {{ animationActive ? "Идёт анимация..." : "Минимизировать (анимация)" }}
    </button>
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
