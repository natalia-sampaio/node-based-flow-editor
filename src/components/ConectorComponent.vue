<script setup lang="ts">
import IconTrash from './icons/IconTrash.vue';

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
        <g
            :class="selected ? 'delete' : 'deleteHidden'"
            :transform="`translate(${position.x0 + (position.x1 - position.x0) / 2}, ${position.y0 + (position.y1 - position.y0) / 2 - (selected ? 24 : 0)})`"
            @mousedown.stop="emit('onClickDelete')"
        >
            <circle cx="0" cy="0" r="14" class="circle" />
            <IconTrash class="icon" />
        </g>
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

.delete {
    cursor: pointer;
    pointer-events: all;
    transition: all ease 0.1s;
}

.deleteHidden {
    cursor: pointer;
    pointer-events: none;
    opacity: 0;
    transition: all ease 0.1s;
}

.icon {
    width: 26px;
    height: 20px;
    background-color: white;
    fill: white;
}

.circle {
    fill: rgb(175, 59, 59);
}
</style>
