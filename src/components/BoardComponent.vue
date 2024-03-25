<script setup lang="ts">
import { onMounted, ref } from 'vue';
import type { Ref } from 'vue';
import ControlButtons from './ControlButtons.vue';
import NodeComponent from './NodeComponent.vue';
import ConnectorComponent from './ConnectorComponent.vue';

const grabbingBoard = ref(false);
const scale = ref(1);
const clickedPosition = ref({ x: -1, y: -1 });
const selectedNode = ref<String | null>(null);

function handleMouseDown(event: any) {
    //Deselect node
    selectedNode.value = null;

    //Deselect connector
    selectedConnector.value = null;

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

    //If a new connector is being set and cursor is not inside of an input, stop creating connector
    if (newConnector.value !== null && insideInput.value === null) {
        newConnector.value = null;
    }

    //If a new connector is being set and cursor is inside an input, create connector and add it to the global array of connectors
    if (newConnector.value !== null && insideInput.value !== null) {
        const nodeStartId = newConnector.value.nodeStartId;
        const nodeEndId = insideInput.value!.nodeId;

        const nodeStart = nodes.value.find((node) => node.id === nodeStartId);
        const nodeEnd = nodes.value.find((node) => node.id === nodeEndId);

        const boardWrapperElement = document.getElementById('boardWrapper');

        if (nodeStart && nodeEnd && boardWrapperElement) {
            const connectorId = `connector_${nodeStart.id}_${newConnector?.value.outputIndex}_${nodeEnd.id}_${insideInput?.value.inputIndex}`;

            //Prevent creation of two identical connectors
            if (
                nodeStart.outputConnectorIds.includes(connectorId) &&
                nodeEnd.inputConnectorIds.includes(connectorId)
            ) {
                newConnector.value = null;
                return;
            }

            nodeStart.outputConnectorIds = [...nodeStart.outputConnectorIds, connectorId];
            nodeEnd.inputConnectorIds = [...nodeEnd.inputConnectorIds, connectorId];

            newConnector!.value.previousStartPosition = {
                x:
                    (newConnector!.value.currentStartPosition.x + boardWrapperElement.scrollLeft) /
                    scale.value,
                y:
                    (newConnector!.value.currentStartPosition.y + boardWrapperElement.scrollTop) /
                    scale.value
            };

            newConnector!.value.previousEndPosition = {
                x: (insideInput!.value.positionX + boardWrapperElement.scrollLeft) / scale.value,
                y: (insideInput!.value.positionY + boardWrapperElement.scrollTop) / scale.value
            };

            newConnector!.value.currentEndPosition = {
                x: (insideInput!.value.positionX + boardWrapperElement.scrollLeft) / scale.value,
                y: (insideInput!.value.positionY + boardWrapperElement.scrollTop) / scale.value
            };

            //Add new connector
            connectors.value = [
                ...connectors.value,
                {
                    ...newConnector.value!,
                    id: connectorId,
                    nodeEndId: nodeEnd.id,
                    inputIndex: insideInput.value!.inputIndex
                }
            ];
            newConnector.value = null;
        }
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

                //Update input connectors positions
                for (let i = 0; i < node.inputConnectorIds.length; i++) {
                    const connectorId = node.inputConnectorIds[i];
                    const connector = connectors.value.find(
                        (connector) => connector.id === connectorId
                    );

                    if (connector) {
                        connector.currentEndPosition = {
                            x: (connector.previousEndPosition.x + deltaX) / scale.value,
                            y: (connector.previousEndPosition.y + deltaY) / scale.value
                        };
                    }
                }

                //Update output connectors positions
                for (let i = 0; i < node.outputConnectorIds.length; i++) {
                    const connectorId = node.outputConnectorIds[i];
                    const connector = connectors.value.find(
                        (connector) => connector.id === connectorId
                    );
                    if (connector) {
                        connector.currentStartPosition = {
                            x: (connector.previousStartPosition.x + deltaX) / scale.value,
                            y: (connector.previousStartPosition.y + deltaY) / scale.value
                        };
                    }
                }
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

    //User setting new connector
    if (newConnector.value !== null) {
        const boardWrapperElement = document.getElementById('boardWrapper');

        if (boardWrapperElement) {
            newConnector.value.currentEndPosition = {
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
    id: string;
    inputs: number;
    outputs: number;
    previousPosition: Ref<{ x: number; y: number }>;
    currentPosition: Ref<{ x: number; y: number }>;
    inputConnectorIds: Ref<string[]>;
    outputConnectorIds: Ref<string[]>;
}

const nodes = ref<Node[]>([]);

function handleOnClickAdd(numberInputs: number, numberOutputs: number) {
    const randomX = Math.random() * window.innerWidth;
    const randomY = Math.random() * window.innerHeight;

    const nodePrevious = { x: randomX, y: randomY };
    const nodeCurrent = { x: randomX, y: randomY };

    const nodeId = `node_${Math.random().toString(36).substring(2, 8)}`;

    const inputsConnectorsIds = [''];
    const outputsConnectorsIds = [''];

    nodes.value.push({
        id: nodeId,
        inputs: numberInputs,
        outputs: numberOutputs,
        previousPosition: nodePrevious,
        currentPosition: nodeCurrent,
        inputConnectorIds: inputsConnectorsIds,
        outputConnectorIds: outputsConnectorsIds
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
    //Deselect connector
    selectedConnector.value = null;

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

        //Update input connectors positions
        for (let i = 0; i < node.inputConnectorIds.length; i++) {
            const connectorId = node.inputConnectorIds[i];
            const connector = connectors.value.find((connector) => connector.id === connectorId);

            if (connector) {
                connector.previousEndPosition = {
                    x: connector.currentEndPosition.x * scale.value,
                    y: connector.currentEndPosition.y * scale.value
                };
            }
        }

        //Update output connectors positions
        for (let i = 0; i < node.inputConnectorIds.length; i++) {
            const connectorId = node.outputConnectorIds[i];
            const connector = connectors.value.find((connector) => connector.id === connectorId);
            if (connector) {
                connector.previousStartPosition = {
                    x: connector.currentStartPosition.x * scale.value,
                    y: connector.currentStartPosition.y * scale.value
                };
            }
        }
    }
}

interface Connector {
    id: string;
    nodeStartId: Ref<string>;
    nodeEndId: Ref<string>;
    inputIndex: Ref<number>;
    outputIndex: Ref<number>;
    previousStartPosition: Ref<{ x: number; y: number }>;
    currentStartPosition: Ref<{ x: number; y: number }>;
    previousEndPosition: Ref<{ x: number; y: number }>;
    currentEndPosition: Ref<{ x: number; y: number }>;
}

const newConnector = ref<Connector | null>(null);
const connectors = ref<Connector[]>([]);

const selectedConnector = ref<string | null>(null);

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
        //Create connector positions with updated scale value
        const previousConnectorStart = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const currentConnectorStart = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const previousConnectorEnd = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        const currentConnectorEnd = {
            x: (outputPositionX + boardWrapperElement.scrollLeft) / scale.value,
            y: (outputPositionY + boardWrapperElement.scrollTop) / scale.value
        };

        newConnector.value = {
            id: '',
            nodeStartId: nodeId,
            outputIndex: outputIndex,
            nodeEndId: '',
            inputIndex: -1,
            previousStartPosition: previousConnectorStart,
            currentStartPosition: currentConnectorStart,
            previousEndPosition: previousConnectorEnd,
            currentEndPosition: currentConnectorEnd
        };
    }
}

const insideInput = ref<{
    nodeId: string;
    inputIndex: number;
    positionX: number;
    positionY: number;
} | null>(null);

function handleMouseOverInput(
    inputPositionX: number,
    inputPositionY: number,
    nodeId: string,
    inputIndex: number
) {
    insideInput.value = {
        nodeId: nodeId,
        inputIndex: inputIndex,
        positionX: inputPositionX,
        positionY: inputPositionY
    };
}
function handleMouseLeaveInput(nodeId: string, inputIndex: number) {
    if (insideInput.value?.nodeId === nodeId && insideInput.value.inputIndex === inputIndex)
        insideInput.value = null;
}

function handleMouseDownConnector(connectorId: string) {
    //Clear any previously selected connector
    selectedConnector.value = null;

    //Clear any previously selected node
    selectedNode.value = null;

    //Select currently clicked connector
    selectedConnector.value = connectorId;
}

function handleDeleteConnector(connectorId: string) {
    const connector = connectors.value.find((c) => c.id === connectorId);

    if (connector) {
        //Delete connector from start node
        const nodeStart = nodes.value.find((n) => n.id === connector.nodeStartId);
        if (nodeStart) {
            nodeStart.outputConnectorIds = [
                ...nodeStart.outputConnectorIds.filter(
                    (connectorId) => connectorId !== connector.id
                )
            ];
        }

        //Delete connector from end node
        const nodeEnd = nodes.value.find((n) => n.id === connector.nodeEndId);
        if (nodeEnd) {
            nodeEnd.inputConnectorIds = [
                ...nodeEnd.inputConnectorIds.filter((connectorId) => connectorId === connector.id)
            ];
        }

        //Delete connector from global connectors array
        connectors.value = [...connectors.value.filter((e) => e.id !== connector.id)];
    }
}
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
            <div v-if="newConnector !== null">
                <ConnectorComponent
                    :selected="false"
                    :isNew="true"
                    :position="{
                        x0: newConnector.currentStartPosition.x,
                        y0: newConnector.currentStartPosition.y,
                        x1: newConnector.currentEndPosition.x,
                        y1: newConnector.currentEndPosition.y
                    }"
                    @on-mouse-down-connector="() => {}"
                    @on-click-delete="() => {}"
                />
            </div>
            <div v-for="connector in connectors || []" :key="connector.id">
                <ConnectorComponent
                    :selected="selectedConnector === connector.id"
                    :isNew="false"
                    :position="{
                        x0: connector.currentStartPosition.x,
                        y0: connector.currentStartPosition.y,
                        x1: connector.currentEndPosition.x,
                        y1: connector.currentEndPosition.y
                    }"
                    @on-mouse-down-connector="handleMouseDownConnector(connector.id)"
                    @on-click-delete="handleDeleteConnector(connector.id)"
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
