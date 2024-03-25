<script setup lang="ts">
import IconTrash from './icons/IconTrash.vue';

defineProps<{
    selected: boolean;
    isNew: boolean;
    position: { x0: number; y0: number; x1: number; y1: number };
}>();

const emit = defineEmits<{
    (e: 'onMouseDownConnector'): void;
    (e: 'onClickDelete'): void;
}>();

function drawConnector(x0: number, y0: number, x1: number, y1: number): string {
    return `M ${x0} ${y0} C ${x0 + calculateOffset(Math.abs(x1 - x0))} ${y0}, ${x1 - calculateOffset(Math.abs(x1 - x0))} ${y1}, ${x1} ${y1}`;
}

function translateDeleteButton(
    x0: number,
    y0: number,
    x1: number,
    y1: number,
    selected: boolean
): string {
    const middlePoint = {
        x: x0 + (x1 - x0) / 2,
        y: y0 + (y1 - y0) / 2
    };

    return `translate(${middlePoint.x}, ${middlePoint.y - (selected ? 24 : 0)})`;
}

//Give the connector an offset to avoid connector overlap
function calculateOffset(value: number): number {
    return value / 2;
}
</script>

<template>
    <svg class="wrapper">
        <path
            :class="isNew ? 'connectorNew' : selected ? 'connectorSelected' : 'connector'"
            :d="drawConnector(position.x0, position.y0, position.x1, position.y1)"
            @mousedown.stop="emit('onMouseDownConnector')"
        />
        <g
            :class="selected ? 'delete' : 'deleteHidden'"
            :transform="
                translateDeleteButton(position.x0, position.y0, position.x1, position.y1, selected)
            "
            @mousedown.stop="emit('onClickDelete')"
        >
            <circle cx="0" cy="0" r="14" class="circle" />
            <IconTrash width="26px" height="20px" transform="translate(-13,-10)" />
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

.connector {
    pointer-events: all;
    stroke: rgba(168, 168, 168, 0.8);
    stroke-width: 2;
    fill: transparent;
    cursor: pointer;
}

.connectorSelected {
    pointer-events: all;
    stroke: rgb(216, 141, 62);
    stroke-width: 3;
    fill: transparent;
    z-index: 100;
}

.connectorNew {
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
