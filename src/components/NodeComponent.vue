<script setup lang="ts">
import { ref } from 'vue';

defineProps<{
    id: string;
    x: number;
    y: number;
    numberInputs: number;
    numberOutputs: number;
    selected: boolean;
    onMouseDownOutput: (
        outputPositionX: number,
        outputPositionY: number,
        nodeId: string,
        outputIndex: number
    ) => void;
    onMouseOverInput: (
        inputPositionX: number,
        inputPositionY: number,
        nodeId: string,
        inputIndex: number
    ) => void;
    onMouseLeaveInput: (nodeId?: string, inputIndex: number) => void;
}>();

const emit = defineEmits<{
    (e: 'onMouseDownNode', id: string, event: any): void;
}>();

const inputs = Array(Number(numberInputs)).keys();
const outputs = Array(Number(numberOutputs)).keys();

function handleMouseOverInput(ref: any, inputIndex: number) {
    const centerX =
        ref.getBoundingClientRect().left +
        Math.abs(ref.getBoundingClientRect().right - ref.getBoundingClientRect().left) / 2;
    const centerY =
        ref.getBoundingClientRect().top +
        Math.abs(ref.getBoundingClientRect().bottom - ref.getBoundingClientRect().top) / 2;

    onMouseOverInput.inputPositionX = centerX;
    onMouseOverInput.inputPositionY = centerY;
    onMouseOverInput.nodeId = id;
    onMouseOverInput.inputIndex = inputIndex;
}

function handleMouseLeaveInput(inputIndex: number) {
    onMouseLeaveInput.nodeId = id;
    onMouseLeaveInput.inputIndex = inputIndex;
}

function handleMouseDownOutput(ref: any, event: any, outputIndex: number) {
    event.stopPropagation();

    const centerX =
        ref.getBoundingClientRect().left +
        Math.abs(ref.getBoundingClientRect().right - ref.getBoundingClientRect().left) / 2;
    const centerY =
        ref.getBoundingClientRect().top +
        Math.abs(ref.getBoundingClientRect().bottom - ref.getBoundingClientRect().top) / 2;

    onMouseDownOutput.outputPositionX = centerX;
    onMouseDownOutput.outputPositionX = centerY;
    onMouseDownOutput.nodeId = id;
    onMouseDownOutput.outputIndex = outputIndex;
}
</script>
<template>
    <div
        :class="selected ? 'nodeSelected' : 'node'"
        :style="{ transform: `translate(${x}px, ${y}px)` }"
        @mousedown.stop="emit('onMouseDownNode', id, event)"
    >
        <div class="inputsWrapper" v-for="input in inputs" :key="input.id">
            <div
                ref="inputRef"
                class="input"
                @mouseover="handleMouseOverInput(inputRef, input)"
                @mouseleave="handleMouseLeaveInput(input)"
            ></div>
        </div>
        <div class="outputsWrapper" v-for="output in outputs" :key="output.id">
            <div
                ref="outputRef"
                class="output"
                @mousedown="(event: any) => handleMouseDownOutput(outputRef, event, output)"
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
