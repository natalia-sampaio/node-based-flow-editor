<script setup lang="ts">
defineProps<{
    selected: boolean;
    isNew: boolean;
    position: { x0: number; y0: number; x1: number; y1: number };
}>();

const emit = defineEmits<{
    (e: 'onMouseDownConector'): void;
    (e: 'onClickDelete'): void;
}>();
</script>
<template>
    <svg class="wrapper">
        <path
            :class="isNew ? 'conectorNew' : selected ? 'conectorSelected' : 'conector'"
            :d="`M ${position.x0} ${position.y0} C ${position.x0 + Math.abs(position.x1 - position.x0)} ${position.y0}, ${position.x1 - Math.abs(position.x1 - position.x0)} ${position.y1}, ${position.x1} ${position.y1}`"
            @mousedown.stop="emit('onMouseDownConector')"
        />
    </svg>
</template>
<style scoped>
.wrapper {
    position: absolute;
    top: 0px;
    left: 0px;
    width: 100%;
    height: 100%;
    pointer-events: none;
}

.conector {
    pointer-events: all;
    stroke: rgba(168, 168, 168, 0.8);
    stroke-width: 2;
    fill: transparent;
    cursor: pointer;
}

.conectorSelected {
    pointer-events: all;
    stroke: rgb(216, 141, 62);
    stroke-width: 3;
    fill: transparent;
    z-index: 100;
}

.conectorNew {
    stroke: rgba(168, 168, 168, 0.4);
    stroke-width: 2;
    fill: transparent;
}
</style>
