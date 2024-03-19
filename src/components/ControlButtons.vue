<script setup lang="ts">
import { ref } from 'vue';
import IconAdd from './icons/IconAdd.vue';
import IconTrash from './icons/IconTrash.vue';

defineProps<{
    showDelete: boolean;
}>();

const emit = defineEmits<{
    (e: 'onClickAdd', numberInputs: number, numberOutputs: number): void;
    (e: 'onClickDelete'): void;
}>();

const isOpen = ref(false);
const numberInputs = ref(0);
const numberOutputs = ref(0);

function handleOnClickAdd() {
    isOpen.value = true;
}

function handleOnClickAddNode() {
    if (
        numberInputs.value > 4 ||
        numberInputs.value < 0 ||
        numberOutputs.value > 4 ||
        numberOutputs.value < 0
    )
        return;

    isOpen.value = false;
    emit('onClickAdd', numberInputs.value, numberOutputs.value);
}

function handleChangeNumberInputs(event: any) {
    numberInputs.value = event.target.value;
}

function handleChangeNumberOutputs(event: any) {
    numberOutputs.value = event.target.value;
}
</script>
<template>
    <div>
        <div class="wrapper">
            <button
                :class="showDelete ? 'buttonDelete' : 'buttonDeleteHidden'"
                @click="$emit('onClickDelete')"
            >
                <IconTrash class="buttonIcon" />
            </button>
            <button class="buttonAdd" @click.stop="handleOnClickAdd">
                <IconAdd class="buttonIcon" />
            </button>
            <div :class="isOpen ? 'dropdown' : 'dropdownHidden'">
                <label class="label">Number of inputs</label>
                <input
                    type="number"
                    class="input"
                    :value="numberInputs"
                    @input="handleChangeNumberInputs"
                />
                <label class="label">Number of outputs</label>
                <input
                    type="number"
                    class="input"
                    :value="numberOutputs"
                    @input="handleChangeNumberOutputs"
                />
                <button class="buttonRect" @click.stop="handleOnClickAddNode">Add node</button>
            </div>
        </div>
    </div>
</template>
<style scoped>
.wrapper {
    position: fixed;
    top: 0px;
    left: 0px;
    width: 100vw;
    padding: 24px 38px;
    display: flex;
    align-items: center;
    justify-content: flex-end;
    gap: 24px;
    z-index: 300;
    pointer-events: none;
    box-sizing: border-box;
}

.buttonIcon {
    width: 24px;
}

.buttonAdd {
    transition: all ease 0.2s;
    background-color: #4f46e5;
    padding: 12px;
    border-radius: 100%;
    color: white;
    font-size: 24px;
    cursor: pointer;
    pointer-events: all;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    outline: none;
    border: none;
    display: flex;
    justify-content: center;
    align-items: center;
}

.buttonAdd:hover,
.buttonRect:hover {
    background-color: #3730a3;
    box-shadow: 2px 2px 12px -6px rgba(0, 0, 0, 0.75);
    transform: scale(1.05);
}

.buttonAdd:active,
.buttonRect:active {
    transform: scale(0.95);
}

.buttonDelete {
    transition: all ease 0.2s;
    background-color: #dc2626;
    padding: 12px;
    border-radius: 100%;
    color: white;
    font-size: 24px;
    cursor: pointer;
    pointer-events: all;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    outline: none;
    border: none;
    display: flex;
    justify-content: center;
    align-items: center;
}

.buttonDelete:hover {
    background-color: #b91c1c;
    box-shadow: 2px 2px 12px -6px rgba(0, 0, 0, 0.75);
    transform: scale(1.05);
}

.buttonDelete:active {
    transform: scale(0.95);
}

.buttonDeleteHidden {
    transform: translateY(-100px);
    background-color: #dc2626;
    padding: 12px;
    border-radius: 100%;
    color: white;
    font-size: 24px;
    cursor: pointer;
    pointer-events: all;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    transition: all ease 0.2s;
}

.dropdown {
    pointer-events: all;
    position: absolute;
    top: 86px;
    display: flex;
    flex-direction: column;
    background-color: white;
    border: 2px solid #c4c4c4;
    border-radius: 6px;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    padding: 12px;
    transition: all ease 0.2s;
}

.dropdownHidden {
    transform: translateX(24px);
    opacity: 0;
    pointer-events: none;
    position: absolute;
    top: 86px;
    display: flex;
    flex-direction: column;
    background-color: white;
    border: 2px solid #c4c4c4;
    border-radius: 6px;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    padding: 12px;
    transition: all ease 0.2s;
}

.label {
    line-height: 1.5rem;
    font-weight: 500;
    font-size: 0.875rem;
    color: #27272a;
}

.input {
    line-height: 1.5rem;
    font-weight: 500;
    font-size: 0.875rem;
    color: #27272a;
    border-radius: 0.375rem;
    width: 100%;
    padding: 0.5rem 0.75rem;
    background-color: #ffffff0d;
    border: 1px solid #b2b3b5;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    margin-bottom: 12px;
    box-sizing: border-box;
}

.buttonRect {
    transition: all ease 0.2s;
    background-color: #4f46e5;
    padding: 12px;
    border-radius: 6px;
    color: white;
    font-size: 16px;
    cursor: pointer;
    pointer-events: all;
    box-shadow: 1px 1px 11px -6px rgba(0, 0, 0, 0.75);
    outline: none;
    border: none;
}
</style>
