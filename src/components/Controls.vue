<template>
    <div class="control-panel">
        <h2 class="control-panel__title">
            <svg class="control-panel__icon" viewBox="0 0 24 24">
                <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M12 6V4m0 2a2 2 0 100 4m0-4a2 2 0 110 4m-6 8a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4m6 6v10m6-2a2 2 0 100-4m0 4a2 2 0 110-4m0 4v2m0-6V4"
                />
            </svg>
            Управление
        </h2>

        <div class="control-panel__buttons">
            <button class="control-btn control-btn--generate" @click="$emit('generate')">
                <span class="control-btn__content">Сгенерировать</span>
            </button>

            <button
                class="control-btn control-btn--minimize"
                @click="$emit('minimize')"
                :disabled="!canMinimize || isAnimating"
            >
                <span class="control-btn__content">Минимизировать</span>
            </button>

            <button
                class="control-btn control-btn--animate"
                @click="$emit('minimize-animated')"
                :disabled="!canMinimize || isAnimating"
            >
                <span class="control-btn__content control-btn__content--pulse">
                    Минимизировать с анимацией
                </span>
            </button>

            <button class="control-btn control-btn--clear" @click="$emit('clear')">
                <span class="control-btn__content">Очистить</span>
            </button>
        </div>
    </div>
</template>

<script setup>
defineProps({
    canMinimize: {
        type: Boolean,
        default: false,
    },
    isAnimating: {
        type: Boolean,
        default: false,
    },
});

defineEmits(["generate", "minimize", "minimize-animated", "clear"]);
</script>

<style lang="scss" scoped>
.control-panel {
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

    &__title {
        display: flex;
        align-items: center;
        gap: 0.75rem;
        font-size: 1.25rem;
        font-weight: 600;
        color: #1e293b;
        margin-bottom: 1.5rem;
    }

    &__icon {
        width: 1.25em;
        height: 1.25em;
        stroke: currentColor;
        stroke-width: 2;
        color: #60a5fa;
    }

    &__buttons {
        display: flex;
        flex-direction: column;
        gap: 0.75rem;
    }
}

.control-btn {
    position: relative;
    display: block;
    width: 100%;
    padding: 0.9rem;
    border-radius: 0.75rem;
    border: none;
    font-weight: 600;
    font-size: 1rem;
    color: white;
    cursor: pointer;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
    overflow: hidden;
    z-index: 1;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1), 0 4px 12px rgba(0, 0, 0, 0.08);

    &::before {
        content: "";
        position: absolute;
        inset: 0;
        background: inherit;
        z-index: -1;
        transition: opacity 0.3s ease;
    }

    &::after {
        content: "";
        position: absolute;
        inset: 0;
        background: linear-gradient(to bottom, rgba(255, 255, 255, 0.2), rgba(255, 255, 255, 0));
        z-index: -1;
        opacity: 0;
        transition: opacity 0.3s ease;
    }

    &:hover::after {
        opacity: 1;
    }

    &:active {
        transform: scale(0.98);
    }

    &:disabled {
        opacity: 0.5;
        cursor: not-allowed;
        transform: none !important;
        box-shadow: none;
    }

    &:not(:disabled):hover {
        transform: translateY(-2px);
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.12), 0 8px 16px rgba(0, 0, 0, 0.1);
    }

    &__content {
        position: relative;
        display: flex;
        align-items: center;
        justify-content: center;
        gap: 0.5rem;
        transition: transform 0.2s ease;

        &--pulse {
            animation: pulse 2s infinite;

            @keyframes pulse {
                0%,
                100% {
                    opacity: 1;
                }
                50% {
                    opacity: 0.8;
                }
            }
        }
    }

    &--generate {
        background: linear-gradient(135deg, #3b82f6, #60a5fa);

        &:hover {
            background: linear-gradient(135deg, #2563eb, #3b82f6);
        }
    }

    &--minimize {
        background: linear-gradient(135deg, #10b981, #34d399);

        &:hover {
            background: linear-gradient(135deg, #059669, #10b981);
        }
    }

    &--animate {
        background: linear-gradient(135deg, #8b5cf6, #a78bfa);

        &:hover {
            background: linear-gradient(135deg, #7c3aed, #8b5cf6);
        }
    }

    &--clear {
        background: linear-gradient(135deg, #ef4444, #f87171);

        &:hover {
            background: linear-gradient(135deg, #dc2626, #ef4444);
        }
    }
}

@media (max-width: 640px) {
    .control-panel {
        padding: 1.25rem;

        &__title {
            font-size: 1.1rem;
            margin-bottom: 1.25rem;
        }
    }

    .control-btn {
        padding: 0.85rem;
        font-size: 0.95rem;
    }
}
</style>
