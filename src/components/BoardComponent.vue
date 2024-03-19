<script setup lang="ts">
import { onMounted, ref } from 'vue';

const grabbingBoard = ref(false);
const scale = ref(1);
const clickedPosition = ref({ x: -1, y: -1 });

function handleMouseDown(event: any) {
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
}

function handleMouseMove(event: any) {
    //User clicked somewhere
    if (clickedPosition.value.x >= 0 && clickedPosition.value.y >= 0) {
        const deltaX = event.x - clickedPosition.value.x;
        const deltaY = event.y - clickedPosition.value.y;

        const boardWrapperElement = document.getElementById('boardWrapper');

        if (boardWrapperElement) {
            boardWrapperElement.scrollBy(-deltaX, -deltaY);
            clickedPosition.value.x = event.x;
            clickedPosition.value.y = event.y;
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
</script>

<template>
    <div id="boardWrapper" class="wrapper">
        <div
            id="board"
            @mousedown="handleMouseDown"
            @mouseup="handleMouseUp"
            @mousemove="handleMouseMove"
            :class="grabbingBoard ? 'boardDragging' : 'board'"
        ></div>
    </div>
</template>

<style scoped>
.wrapper {
    position: fixed;
    width: 100vw;
    height: 100vw;
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
    height: 100vw;
    background-size: 30px 30px;
    background-image: radial-gradient(circle, #b8b8b8bf 1px, rgba(0, 0, 0, 0) 1px);
    cursor: grabbing;
}
</style>
