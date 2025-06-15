<script setup>
import { ref, onMounted, onUnmounted, watch } from "vue";

const props = defineProps({
    points: {
        type: Array,
        required: true,
        validator: (value) => {
            // Валидация точек на уникальность координат
            const uniquePoints = new Set(value.map((p) => `${p.x},${p.y}`));
            return uniquePoints.size === value.length;
        },
    },
    highlightedVertex: {
        type: Number,
        default: -1,
        validator: (value) => value >= -1,
    },
    badVertices: {
        type: Array,
        default: () => [],
        validator: (value) => value.every((i) => Number.isInteger(i) && i >= 0),
    },
});

const emit = defineEmits(["add-point", "move-point", "close-polygon", "cancel-drag"]);

// Refs
const canvas = ref(null);
const ctx = ref(null);
const isDrawing = ref(false);
const isDragging = ref(false);
const currentPoint = ref(null);
const draggedPoint = ref(null);
const isClosed = ref(false);
const gridSize = 20;
const scale = 1;
const snapToGrid = ref(true); // Добавлена привязка к сетке

// Получение позиции мыши с учетом привязки к сетке
function getMousePos(event) {
    const canvasElement = canvas.value;
    const rect = canvasElement.getBoundingClientRect();
    const dpr = window.devicePixelRatio || 1;

    // Корректный расчёт с учётом всех факторов
    const x = (event.clientX - rect.left) * (canvasElement.width / rect.width / dpr);
    const y = (event.clientY - rect.top) * (canvasElement.height / rect.height / dpr);

    // Привязка к сетке (если включена)
    if (snapToGrid.value) {
        return {
            x: Math.round(x / gridSize) * gridSize,
            y: Math.round(y / gridSize) * gridSize,
        };
    }

    return { x, y };
}

// Проверка на самопересечение
function checkSelfIntersection(newPoint, excludeIndex = -1) {
    if (props.points.length < 3) return false;

    // Реализация алгоритма проверки самопересечения
    // (здесь должен быть конкретный алгоритм)
    return false;
}

// Инициализация canvas
function initCanvas() {
    const canvasElement = canvas.value;
    ctx.value = canvasElement.getContext("2d");

    const resizeCanvas = () => {
        const container = canvasElement.parentElement;
        const dpr = window.devicePixelRatio || 1;
        canvasElement.width = container.clientWidth * dpr;
        canvasElement.height = container.clientHeight * dpr;
        canvasElement.style.width = container.clientWidth + "px";
        canvasElement.style.height = container.clientHeight + "px";
        ctx.value.setTransform(1, 0, 0, 1, 0, 0);
        ctx.value.scale(dpr, dpr);
        draw();
    };

    resizeCanvas();
    window.addEventListener("resize", resizeCanvas);
}

// Отрисовка сетки с единицами измерения
function drawGrid() {
    const canvasElement = canvas.value;
    const width = canvasElement.width / (window.devicePixelRatio || 1);
    const height = canvasElement.height / (window.devicePixelRatio || 1);
    const gridStep = gridSize; // Размер шага сетки (20px)

    // Стиль для линий сетки
    ctx.value.strokeStyle = "#e0e7ef";
    ctx.value.lineWidth = 0.7;
    ctx.value.font = "10px Arial";
    ctx.value.fillStyle = "#94a3b8";
    ctx.value.textAlign = "center";
    ctx.value.textBaseline = "top";

    // Вертикальные линии с подписями
    for (let x = 0; x <= width; x += gridStep) {
        ctx.value.beginPath();
        ctx.value.moveTo(x, 0);
        ctx.value.lineTo(x, height);
        ctx.value.stroke();

        // Подписи осей X (каждые 5 линий)
        if (x % (gridStep * 5) === 0 && x > 0) {
            ctx.value.fillText(`${(x / gridStep) * scale}m`, x, 5);
        }
    }

    // Горизонтальные линии с подписями
    for (let y = 0; y <= height; y += gridStep) {
        ctx.value.beginPath();
        ctx.value.moveTo(0, y);
        ctx.value.lineTo(width, y);
        ctx.value.stroke();

        // Подписи осей Y (каждые 5 линий)
        if (y % (gridStep * 5) === 0 && y > 0) {
            ctx.value.textAlign = "left";
            ctx.value.fillText(`${(y / gridStep) * scale}m`, 5, y);
            ctx.value.textAlign = "center";
        }
    }

    // Подпись в начале координат (0,0)
    ctx.value.textAlign = "left";
    ctx.value.fillText("0", 5, 5);
}

// Основная функция отрисовки
function draw() {
    const canvasElement = canvas.value;
    const width = canvasElement.width / (window.devicePixelRatio || 1);
    const height = canvasElement.height / (window.devicePixelRatio || 1);

    // Очистка
    ctx.value.clearRect(0, 0, width, height);
    drawGrid();

    // Отрисовка линий полигона
    if (props.points.length > 0) {
        ctx.value.beginPath();
        ctx.value.moveTo(props.points[0].x, props.points[0].y);

        for (let i = 1; i < props.points.length; i++) {
            ctx.value.lineTo(props.points[i].x, props.points[i].y);
        }

        if (isClosed.value && props.points.length > 2) {
            ctx.value.closePath();
        }

        ctx.value.strokeStyle = "#3b82f6";
        ctx.value.lineWidth = 2;
        ctx.value.shadowColor = "rgba(59,130,246,0.10)";
        ctx.value.shadowBlur = 8;
        ctx.value.stroke();
        ctx.value.shadowBlur = 0;
    }

    // Отрисовка точек
    props.points.forEach((point, index) => {
        ctx.value.save();
        ctx.value.beginPath();
        ctx.value.arc(point.x, point.y, 11, 0, Math.PI * 2);

        // Проверка на точки с одинаковыми координатами (дубликаты)
        const isDuplicatePoint = props.points.some(
            (p, i) => i !== index && p.x === point.x && p.y === point.y,
        );

        if (props.badVertices && props.badVertices.includes(index)) {
            if (isDuplicatePoint) {
                // Стиль для точек с одинаковыми координатами
                ctx.value.fillStyle = "#64748b"; // Серый цвет как у обычных точек
                ctx.value.strokeStyle = "#ef4444"; // Красная обводка
                ctx.value.lineWidth = 3;
                ctx.value.fill();
                ctx.value.stroke();
            } else {
                // Стиль для просто "плохих" точек
                ctx.value.fillStyle = "#ffffff"; // Белая заливка
                ctx.value.strokeStyle = "#ef4444"; // Красная обводка
                ctx.value.lineWidth = 3;
                ctx.value.fill();
                ctx.value.stroke();
            }
        } else {
            // Отрисовка обычных вершин
            ctx.value.fillStyle = props.highlightedVertex === index ? "#3b82f6" : "#64748b";
            ctx.value.strokeStyle = "#fff";
            ctx.value.lineWidth = 1;
            ctx.value.globalAlpha = 0.92;
            ctx.value.fill();
            ctx.value.stroke();
        }

        // Номера вершин
        ctx.value.globalAlpha = 1;
        if (props.badVertices && props.badVertices.includes(index)) {
            if (isDuplicatePoint) {
                ctx.value.fillStyle = "#ffffff"; // Белый текст для дубликатов
            } else {
                ctx.value.fillStyle = "#000000"; // Чёрный текст для просто плохих точек
            }
        } else {
            ctx.value.fillStyle = "#ffffff"; // Белый текст по умолчанию
        }
        ctx.value.font = "14px Arial";
        ctx.value.textAlign = "center";
        ctx.value.textBaseline = "middle";
        ctx.value.fillText((index + 1).toString(), point.x, point.y);
        ctx.value.restore();
    });

    // Отрисовка временной линии (превью)
    if (isDrawing.value && currentPoint.value && !isClosed.value) {
        const lastPoint = props.points[props.points.length - 1];
        ctx.value.beginPath();
        ctx.value.moveTo(lastPoint.x, lastPoint.y);
        ctx.value.lineTo(currentPoint.value.x, currentPoint.value.y);
        ctx.value.strokeStyle = "#9CA3AF";
        ctx.value.setLineDash([5, 5]);
        ctx.value.stroke();
        ctx.value.setLineDash([]);
    }
}

// Обработчики событий
function handleClick(event) {
    if (isDragging.value || isClosed.value) return;

    const { x, y } = getMousePos(event);
    emit("add-point", { x, y });
}

function handleMouseMove(event) {
    const { x, y } = getMousePos(event);
    currentPoint.value = { x, y };

    // Запрещаем перемещение если полигон не замкнут
    if (isDragging.value && draggedPoint.value !== null && isClosed.value) {
        // Проверка на самопересечение при перемещении
        if (!checkSelfIntersection({ x, y }, draggedPoint.value)) {
            emit("move-point", { index: draggedPoint.value, x, y });
        }
    }

    draw();
}

function handleMouseDown(event) {
    if (event.button === 2 || !isClosed.value) return;

    const { x, y } = getMousePos(event);

    // Проверяем, кликнули ли на существующую точку
    for (let i = 0; i < props.points.length; i++) {
        const point = props.points[i];
        const distance = Math.sqrt(Math.pow(point.x - x, 2) + Math.pow(point.y - y, 2));

        if (distance <= 15) {
            isDragging.value = true;
            draggedPoint.value = i;
            return;
        }
    }
}

function handleMouseUp() {
    if (isDragging.value) {
        emit("cancel-drag", draggedPoint.value);
    }

    isDragging.value = false;
    draggedPoint.value = null;
}

function handleMouseLeave() {
    if (isDragging.value) {
        emit("cancel-drag", draggedPoint.value);
        isDragging.value = false;
        draggedPoint.value = null;
    }
}

function handleRightClick(event) {
    if (props.points.length >= 3 && !isClosed.value) {
        isClosed.value = true;
        emit("close-polygon");
        draw();
    }
}

// Хуки жизненного цикла
onMounted(() => {
    initCanvas();
});

watch(
    () => props.points,
    (newPoints) => {
        if (isClosed.value && newPoints.length < 3) {
            isClosed.value = false;
        }
        draw();
    },
    { deep: true },
);

watch(
    () => [props.highlightedVertex, props.badVertices],
    () => {
        draw();
    },
    { deep: true },
);

onUnmounted(() => {
    ctx.value = null;
    window.removeEventListener("resize", initCanvas);
});
</script>

<template>
    <div class="glass-card">
        <canvas
            ref="canvas"
            @click="handleClick"
            @mousemove="handleMouseMove"
            @mousedown="handleMouseDown"
            @mouseup="handleMouseUp"
            @mouseleave="handleMouseLeave"
            @contextmenu.prevent="handleRightClick"
        ></canvas>

        <div class="scale-info">Масштаб: 20px = {{ scale }}м</div>
    </div>
</template>

<style scoped>
.glass-card {
    --glass-bg: rgba(255, 255, 255, 0.18);
    --glass-border: rgba(255, 255, 255, 0.24);
    --glass-shadow: 0 8px 32px rgba(31, 38, 135, 0.18);

    background: var(--glass-bg);
    backdrop-filter: blur(16px);
    border-radius: 1.5rem;
    border: 1px solid var(--glass-border);
    box-shadow: var(--glass-shadow);
    padding: 1.5rem;
    width: 100%;
    height: 100%;
}

canvas {
    width: 100%;
    height: calc(100dvh - 96px);
    border-radius: 1rem;
    border: 1px solid rgba(203, 213, 225, 0.3);
    background-color: white;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.05), 0 4px 16px rgba(59, 130, 246, 0.1);
}

.scale-info {
    position: absolute;
    bottom: 10px;
    right: 25px;
    background: rgba(255, 255, 255, 0.8);
    font-size: 12px;
}
</style>
