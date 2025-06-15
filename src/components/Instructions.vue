<template>
    <div class="instruction-card">
        <h2 class="instruction-card__title">
            <svg class="instruction-card__icon" viewBox="0 0 24 24">
                <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                />
            </svg>
            Инструкция
        </h2>

        <div class="instruction-card__content">
            <div class="instruction-step">
                <h3 class="instruction-step__title">Как использовать:</h3>
                <ol class="instruction-step__list">
                    <li v-for="(step, index) in steps" :key="index" class="instruction-step__item">
                        <span class="instruction-step__number">{{ index + 1 }}</span>
                        <span class="instruction-step__text">{{ step }}</span>
                    </li>
                </ol>
            </div>
        </div>
    </div>
</template>

<script setup>
const steps = [
    "Кликните левой кнопкой мыши для добавления вершин",
    "Перетащите вершину для изменения её положения",
    "Кликните правой кнопкой мыши для замыкания многоугольника",
];
</script>

<style lang="scss" scoped>
.instruction-card {
    --glass-bg: rgba(255, 255, 255, 0.92);
    --glass-border: rgba(203, 213, 225, 0.3);
    --glass-shadow: 0 4px 30px rgba(0, 0, 0, 0.05);

    background: var(--glass-bg);
    backdrop-filter: blur(12px);
    border-radius: 1rem;
    border: 1px solid var(--glass-border);
    box-shadow: var(--glass-shadow);
    padding: 1.5rem;
    transition: all 0.3s ease;

    &:hover {
        box-shadow: 0 6px 24px rgba(0, 0, 0, 0.1);
    }

    &__title {
        display: flex;
        align-items: center;
        gap: 0.75rem;
        font-size: 1.25rem;
        font-weight: 600;
        color: #1e293b;
        margin-bottom: 1.25rem;
    }

    &__icon {
        width: 1.25em;
        height: 1.25em;
        stroke: currentColor;
        stroke-width: 2;
        color: #60a5fa;
    }

    &__content {
        display: flex;
        flex-direction: column;
        gap: 0.75rem;
    }
}

.instruction-step {
    padding: 1.25rem;
    border-radius: 0.75rem;
    border: 1px solid rgba(147, 197, 253, 0.4);
    background: linear-gradient(135deg, rgba(219, 234, 254, 0.7), rgba(191, 219, 254, 0.4));
    box-shadow: 0 2px 8px rgba(59, 130, 246, 0.08);

    &__title {
        font-weight: 600;
        font-size: 1rem;
        color: #1e40af;
        margin-bottom: 1rem;
    }

    &__list {
        display: flex;
        flex-direction: column;
        gap: 0.75rem;
    }

    &__item {
        display: flex;
        align-items: flex-start;
        gap: 0.75rem;
        line-height: 1.5;
    }

    &__number {
        flex-shrink: 0;
        width: 1.5rem;
        height: 1.5rem;
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: rgba(59, 130, 246, 0.15);
        color: #1e40af;
        border-radius: 50%;
        font-weight: 600;
        font-size: 0.75rem;
        transition: all 0.2s ease;
    }

    &__text {
        font-size: 0.9rem;
        color: #1e40af;
        padding-top: 0.125rem;
    }

    &__item:hover &__number {
        background-color: rgba(59, 130, 246, 0.25);
        transform: scale(1.1);
    }
}

@media (max-width: 1024px) {
    .instruction-card {
        padding: 1.25rem;

        &__title {
            font-size: 1.1rem;
            margin-bottom: 1rem;
        }
    }

    .instruction-step {
        padding: 1rem;

        &__title {
            font-size: 0.95rem;
            margin-bottom: 0.75rem;
        }

        &__text {
            font-size: 0.85rem;
        }
    }
}
</style>
