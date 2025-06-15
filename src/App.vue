<script setup>
import { ref, computed } from "vue";
import Controls from "./components/Controls.vue";
import Instructions from "./components/Instructions.vue";
import PolygonCanvas from "./components/PolygonCanvas.vue";
import VertexList from "./components/VertexList.vue";

// State
const vertices = ref([]);
const highlightedVertex = ref(-1);
const badVertices = ref([]);
const isAnimating = ref(false);

// Computed
const canMinimize = computed(() => vertices.value.length >= 3);

// Methods
const addVertex = (point) => {
    vertices.value.push(point);
    checkForBadVertices();
};

const moveVertex = ({ index, x, y }) => {
    if (index >= 0 && index < vertices.value.length) {
        vertices.value[index] = { x, y };
        checkForBadVertices();
    }
};

const removeVertex = (index) => {
    vertices.value.splice(index, 1);
    checkForBadVertices();
};

const closePolygon = () => {
    if (vertices.value.length > 2) {
        checkForBadVertices();
    }
};

const clearPolygon = () => {
    vertices.value = [];
    highlightedVertex.value = -1;
    badVertices.value = [];
};

const generatePolygon = () => {
    // Generate a random polygon with intentional problems
    const numPoints = Math.floor(Math.random() * 5) + 5; // 5-10 points
    const centerX = 400;
    const centerY = 300;
    const radius = 150;

    vertices.value = [];

    for (let i = 0; i < numPoints; i++) {
        const angle = (i * 2 * Math.PI) / numPoints;
        const x = centerX + radius * Math.cos(angle);
        const y = centerY + radius * Math.sin(angle);

        // Add some random offset
        const offsetX = (Math.random() - 0.5) * 50;
        const offsetY = (Math.random() - 0.5) * 50;

        vertices.value.push({ x: x + offsetX, y: y + offsetY });

        // Sometimes add a duplicate point
        if (Math.random() < 0.3) {
            vertices.value.push({ x: x + offsetX, y: y + offsetY });
        }
    }

    checkForBadVertices();
};

const checkForBadVertices = () => {
    const bad = [];
    const n = vertices.value.length;

    if (n < 3) {
        badVertices.value = [];
        return;
    }

    // Check for collinear points
    for (let i = 0; i < n; i++) {
        const a = vertices.value[i];
        const b = vertices.value[(i + 1) % n];
        const c = vertices.value[(i + 2) % n];

        // Calculate area using cross product
        const area = Math.abs((b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x));

        // Use a small threshold based on the scale of the points
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

    // Check for duplicate points
    for (let i = 0; i < n; i++) {
        for (let j = i + 1; j < n; j++) {
            const a = vertices.value[i];
            const b = vertices.value[j];
            const dx = a.x - b.x;
            const dy = a.y - b.y;
            const distance = Math.sqrt(dx * dx + dy * dy);

            if (distance < 1) {
                if (!bad.includes(i)) bad.push(i);
                if (!bad.includes(j)) bad.push(j);
            }
        }
    }

    badVertices.value = bad;
};

const minimizePolygon = () => {
    if (!canMinimize.value) return;

    let changed = true;
    while (changed) {
        changed = false;
        for (let i = 0; i < vertices.value.length; i++) {
            const a = vertices.value[i];
            const b = vertices.value[(i + 1) % vertices.value.length];
            const c = vertices.value[(i + 2) % vertices.value.length];

            const area = Math.abs((b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x));
            const scale =
                1e-6 *
                (Math.abs(a.x) +
                    Math.abs(a.y) +
                    Math.abs(b.x) +
                    Math.abs(b.y) +
                    Math.abs(c.x) +
                    Math.abs(c.y));

            if (area < scale) {
                vertices.value.splice((i + 1) % vertices.value.length, 1);
                changed = true;
                break;
            }
        }
    }

    checkForBadVertices();
};

const minimizePolygonAnimated = async () => {
    if (!canMinimize.value || isAnimating.value) return;

    isAnimating.value = true;
    let changed = true;

    while (changed) {
        changed = false;
        for (let i = 0; i < vertices.value.length; i++) {
            const a = vertices.value[i];
            const b = vertices.value[(i + 1) % vertices.value.length];
            const c = vertices.value[(i + 2) % vertices.value.length];

            const area = Math.abs((b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x));
            const scale =
                1e-6 *
                (Math.abs(a.x) +
                    Math.abs(a.y) +
                    Math.abs(b.x) +
                    Math.abs(b.y) +
                    Math.abs(c.x) +
                    Math.abs(c.y));

            if (area < scale) {
                highlightedVertex.value = (i + 1) % vertices.value.length;
                await new Promise((resolve) => setTimeout(resolve, 800));
                vertices.value.splice(highlightedVertex.value, 1);
                changed = true;
                break;
            }
        }
    }

    highlightedVertex.value = -1;
    isAnimating.value = false;
    checkForBadVertices();
};
</script>

<template>
    <div class="main">
        <div class="controls">
            <Instructions />
            <Controls
                :can-minimize="canMinimize"
                :is-animating="isAnimating"
                @generate="generatePolygon"
                @minimize="minimizePolygon"
                @minimize-animated="minimizePolygonAnimated"
                @clear="clearPolygon"
            />
            <VertexList
                :vertices="vertices"
                :highlighted-vertex="highlightedVertex"
                :bad-vertices="badVertices"
                @remove-vertex="removeVertex"
            />
        </div>
        <div class="canvas">
            <PolygonCanvas
                :points="vertices"
                :highlighted-vertex="highlightedVertex"
                :bad-vertices="badVertices"
                @add-point="addVertex"
                @move-point="moveVertex"
                @close-polygon="closePolygon"
            />
        </div>
    </div>
</template>

<style lang="scss" scoped>
.main {
    width: 100%;
    height: auto;

    padding: 24px;

    display: grid;
    grid-template-columns: 1fr 3fr;
    gap: 14px;
}

.controls {
    width: 100%;
    height: auto;

    display: flex;
    flex-direction: column;
    gap: 14px;
}
</style>
