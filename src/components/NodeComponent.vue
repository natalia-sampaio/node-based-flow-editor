<script setup lang="ts">
defineProps<{
    id: string;
    x: number;
    y: number;
    numberInputs: number;
    numberOutputs: number;
    selected: boolean;
}>();

const emit = defineEmits<{
    (e: 'onMouseDownNode', id: string, event: any): void;
    (
        e: 'onMouseOverInput',
        inputPositionX: number,
        inputPositionY: number,
        nodeId: string,
        inputIndex: number
    ): void;
    (e: 'onMouseLeaveInput', nodeId: string, inputIndex: number): void;
    (
        e: 'onMouseDownOutput',
        outputPositionX: number,
        outputPositionY: number,
        nodeId: string,
        outputIndex: number
    ): void;
}>();

function handleMouseOverInput(ref: any, inputIndex: number, id: string) {
    const centerX =
        ref[inputIndex].getBoundingClientRect().left +
        Math.abs(
            ref[inputIndex].getBoundingClientRect().right -
                ref[inputIndex].getBoundingClientRect().left
        ) /
            2;
    const centerY =
        ref[inputIndex].getBoundingClientRect().top +
        Math.abs(
            ref[inputIndex].getBoundingClientRect().bottom -
                ref[inputIndex].getBoundingClientRect().top
        ) /
            2;
    emit('onMouseOverInput', centerX, centerY, id, inputIndex);
}

function handleMouseLeaveInput(inputIndex: number, id: string) {
    emit('onMouseLeaveInput', id, inputIndex);
}

function handleMouseDownOutput(ref: any, event: any, outputIndex: number, id: string) {
    const centerX =
        ref[outputIndex].getBoundingClientRect().left +
        Math.abs(
            ref[outputIndex].getBoundingClientRect().right -
                ref[outputIndex].getBoundingClientRect().left
        ) /
            2;
    const centerY =
        ref[outputIndex].getBoundingClientRect().top +
        Math.abs(
            ref[outputIndex].getBoundingClientRect().bottom -
                ref[outputIndex].getBoundingClientRect().top
        ) /
            2;
    emit('onMouseOverInput', centerX, centerY, id, outputIndex);
}

const inputRef = null;
const outputRef = null;
</script>
<template>
    <div
        :class="selected ? 'nodeSelected' : 'node'"
        :style="{ transform: `translate(${x}px, ${y}px)` }"
        @mousedown.stop="emit('onMouseDownNode', id, $event)"
    >
        <div class="inputsWrapper">
            <div
                v-for="input in [...Array(Number(numberInputs)).keys()]"
                :key="input"
                ref="inputRef"
                class="input"
                @mouseover="handleMouseOverInput(inputRef, input, id)"
                @mouseleave="handleMouseLeaveInput(input, id)"
            ></div>
        </div>
        <div class="outputsWrapper">
            <div
                v-for="output in [...Array(Number(numberOutputs)).keys()]"
                :key="output"
                ref="outputRef"
                class="output"
                @mousedown.stop="
                    (event: any) => handleMouseDownOutput(outputRef, event, output, id)
                "
            ></div>
        </div>
    </div>
</template>
<style scoped>
.node {
    display: flex;
    flex-direction: column;
    position: absolute;
    cursor: grab;
    background-color: white;
    border: 2px solid #c4c4c4;
    border-radius: 6px;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    user-select: none;
    z-index: 1;
    transition:
        border ease 0.2s,
        box-shadow ease 0.2s;
    width: 120px;
    height: 120px;
}

.node:hover {
    box-shadow: 2px 2px 12px -6px rgba(0, 0, 0, 0.75);
}

.nodeSelected {
    display: flex;
    flex-direction: column;
    position: absolute;
    cursor: grab;
    background-color: white;
    border: 2px solid #e38c29;
    border-radius: 6px;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    user-select: none;
    z-index: 100;
    transition:
        border ease 0.2s,
        box-shadow ease 0.2s;
    width: 120px;
    height: 120px;
}

.nodeSelected:hover {
    box-shadow: 2px 2px 12px -6px rgba(0, 0, 0, 0.75);
}

.inputsWrapper {
    position: absolute;
    top: 0px;
    left: -24px;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 12px;
    width: 12px;
    pointer-events: none;
}

.outputsWrapper {
    position: absolute;
    top: 0px;
    right: -24px;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 12px;
    width: 12px;
    pointer-events: none;
}

.input,
.output {
    width: 12px;
    height: 12px;
    border-radius: 100%;
    background-color: #e38c29;
    cursor: crosshair;
    pointer-events: all;
}
</style>
