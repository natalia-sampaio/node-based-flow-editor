<script setup lang="ts">
import { onMounted, ref } from 'vue';
import type { Ref } from 'vue';
import ControlButtons from './ControlButtons.vue';
import NodeComponent from './NodeComponent.vue';
import ConectorComponent from './ConectorComponent.vue';

const grabbingBoard = ref(false);
const scale = ref(1);
const clickedPosition = ref({ x: -1, y: -1 });
const selectedNode = ref<String | null>(null);

function handleMouseDown(event: any) {
    //Deselect node
    selectedNode.value = null;

    //Start grabbing board
    grabbingBoard.value = true;
    clickedPosition.value.x = event.x;
    clickedPosition.value.y = event.y;
}

function handleMouseUp() {
    //Reset clicked position
    clickedPosition.value.x = -1;
    clickedPosition.value.y = -1;

    //Stop grabbing board
    grabbingBoard.value = false;

    //Stop creating conector
    if (newConector.value !== null) {
        newConector.value = null;
    }
}

function handleMouseMove(event: any) {
    //User clicked somewhere
    if (clickedPosition.value.x >= 0 && clickedPosition.value.y >= 0) {
        const deltaX = event.x - clickedPosition.value.x;
        const deltaY = event.y - clickedPosition.value.y;

        if (selectedNode.value !== null) {
            const deltaX = event.x - clickedPosition.value.x;
            const deltaY = event.y - clickedPosition.value.y;

            const node = nodes.value.find((node) => node.id === selectedNode.value);

            if (node) {
                //Update node position
                node.currentPosition = {
                    x: (node.previousPosition.x + deltaX) / scale.value,
                    y: (node.previousPosition.y + deltaY) / scale.value
                };
            }
        }
        //User clicked on board, move board
        else {
            const boardWrapperElement = document.getElementById('boardWrapper');

            if (boardWrapperElement) {
                boardWrapperElement.scrollBy(-deltaX, -deltaY);
                clickedPosition.value.x = event.x;
                clickedPosition.value.y = event.y;
            }
        }
    }

    //User setting new conector
    if (newConector.value !== null) {
        const boardWrapperElement = document.getElementById('boardWrapper');

        if (boardWrapperElement) {
            newConector.value.currentEndPosition = {
                x: (event.x + boardWrapperElement.scrollLeft) / scale.value,
                y: (event.y + boardWrapperElement.scrollTop) / scale.value
            };
        }
    }
}

onMounted(() => {
    const boardElement = document.getElementById('board');

    if (boardElement) {
        boardElement.addEventListener(
            'wheel',
            (event) => {
                //Update scale
                scale.value = scale.value + event.deltaY * -0.005;

                //Restrict scale
                scale.value = Math.min(Math.max(1, scale.value), 2);

                //Apply scale transform
                boardElement.style.transform = `scale(${scale.value})`;
                boardElement.style.marginTop = `${(scale.value - 1) * 50}vh`;
                boardElement.style.marginLeft = `${(scale.value - 1) * 50}vw`;
            },
            { passive: false }
        );
    }
});

interface Node {
    id: Ref<string>;
    inputs: Ref<number>;
    outputs: Ref<number>;
    previousPosition: Ref<{ x: number; y: number }>;
    currentPosition: Ref<{ x: number; y: number }>;
}

const nodes = ref<Node[]>([]);

function handleOnClickAdd(numberInputs: number, numberOutputs: number) {
    const randomX = Math.random() * window.innerWidth;
    const randomY = Math.random() * window.innerHeight;

    const nodePrevious = { x: randomX, y: randomY };
    const nodeCurrent = { x: randomX, y: randomY };

    const nodeId = `node_${Math.random().toString(36).substring(2, 8)}`;

    nodes.value.push({
        id: nodeId,
        inputs: numberInputs,
        outputs: numberOutputs,
        previousPosition: nodePrevious,
        currentPosition: nodeCurrent
    });
}

function handleOnClickDelete() {
    //Find node in global nodes array
    const node = nodes.value.find((node) => node.id === selectedNode.value);

    //Check if node exists
    if (!node) {
        selectedNode.value = null;
        return;
    }

    //Delete node
    nodes.value = nodes.value.filter((node) => node.id !== selectedNode.value);

    //Hide delete button
    selectedNode.value = null;
}

function handleMouseDownNode(id: string, event: any) {
    //Select node
    selectedNode.value = id;

    //Update first click position
    clickedPosition.value = { x: event.x, y: event.y };

    const node = nodes.value.find((node) => node.id === selectedNode.value);
    if (node) {
        //Update node position
        node.previousPosition = {
            x: node.currentPosition.x * scale.value,
            y: node.currentPosition.y * scale.value
        };
    }
}

interface Edge {
    id: string;
    nodeStartId: string;
    nodeEndId: string;
    inputIndex: number;
    outputIndex: number;
    previousStartPosition: Ref<{ x: number; y: number }>;
    currentStartPosition: Ref<{ x: number; y: number }>;
    previousEndPosition: Ref<{ x: number; y: number }>;
    currentEndPosition: Ref<{ x: number; y: number }>;
}

const newConector = ref<Edge | null>(null);

function handleMouseDownOutput(
    outputPositionX: number,
    outputPositionY: number,
    nodeId: string,
    outputIndex: number
) {
    //Deselect node
    selectedNode.value = null;

    const boardWrapperElement = document.getElementById('boardWrapper');

    if (boardWrapperElement) {
        //Create conector positions with updated scale value
        const previousConectorStart = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const currentConectorStart = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const previousConectorEnd = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const currentConectorEnd = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        newConector.value = {
            id: '',
            nodeStartId: nodeId,
            outputIndex: outputIndex,
            nodeEndId: '',
            inputIndex: -1,
            previousStartPosition: previousConectorStart,
            currentStartPosition: currentConectorStart,
            previousEndPosition: previousConectorEnd,
            currentEndPosition: currentConectorEnd
        };
    }
}

function handleMouseOverInput(
    inputPositionX: number,
    inputPositionY: number,
    nodeId: string,
    inputIndex: number
) {}
function handleMouseLeaveInput(nodeId: string, inputIndex: number) {}
</script>

<template>
    <div id="boardWrapper" class="wrapper">
        <ControlButtons
            :showDelete="selectedNode !== null"
            @on-click-add="handleOnClickAdd"
            @on-click-delete="handleOnClickDelete"
        />

        <div
            id="board"
            @mousedown="handleMouseDown"
            @mouseup="handleMouseUp"
            @mousemove="handleMouseMove"
            :class="grabbingBoard ? 'boardDragging' : 'board'"
        >
            <div v-for="node in nodes" :key="node.id">
                <NodeComponent
                    :id="node.id"
                    :x="node.currentPosition.x"
                    :y="node.currentPosition.y"
                    :numberInputs="node.inputs"
                    :numberOutputs="node.outputs"
                    :selected="selectedNode === node.id"
                    @on-mouse-down-node="handleMouseDownNode"
                    @on-mouse-down-output="handleMouseDownOutput"
                    @on-mouse-over-input="handleMouseOverInput"
                    @on-mouse-leave-input="handleMouseLeaveInput"
                />
            </div>
            <div v-if="newConector !== null">
                <ConectorComponent
                    :selected="false"
                    :isNew="true"
                    :position="{
                        x0: newConector.currentStartPosition.x,
                        y0: newConector.currentStartPosition.y,
                        x1: newConector.currentEndPosition.x,
                        y1: newConector.currentEndPosition.y
                    }"
                    @on-mouse-down-conector="() => {}"
                    @on-click-delete="() => {}"
                />
            </div>
        </div>
    </div>
</template>

<style scoped>
.wrapper {
    position: fixed;
    width: 100vw;
    height: 100vh;
    top: 0px;
    left: 0px;
    overflow: scroll;
}

.board {
    position: relative;
    width: 100vw;
    height: 100vh;
    background-size: 30px 30px;
    background-image: radial-gradient(circle, #b8b8b8bf 1px, rgba(0, 0, 0, 0) 1px);
    cursor: grab;
}

.boardDragging {
    position: relative;
    width: 100vw;
    height: 100vh;
    background-size: 30px 30px;
    background-image: radial-gradient(circle, #b8b8b8bf 1px, rgba(0, 0, 0, 0) 1px);
    cursor: grabbing;
}
</style>
