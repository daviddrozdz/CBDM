<script setup>
  
import { ref } from "vue";

// Define properties that you will be able to access from parent component.
// Those properties will be binded from parent to child.
// Available JavaScript types: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures
const props = defineProps(['title', 'min', 'max', 'step', 'val'])

const title = ref(props.title)
var inputfieldValue = ref(props.val)

// Define events that will be accessible from parent component
const emits = defineEmits(['update'])

function emitValueUpdate() {
  emits("update", inputfieldValue.value, title.value)
}

</script>



<template>

    <div>

      <form class="definition-input">

        <!-- <label class="input-title" for="range-slider">{{ title }}: {{ inputfieldValue }}</label> -->
        <label class="input-title" for="range-slider">{{ title }}</label>
        
        <input
          type="number"
          class="modern-input"
          :min="min"
          :max="max"
          :step="step"
          v-model.number="inputfieldValue"
          @input="emitValueUpdate"
        />

      </form>

    </div>

</template>



 <style scoped>

  .input-title {
    font-family: 'Roboto', sans-serif;
    font-size: 16px;
  }
  
  .modern-input {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 8px;
    margin-top: 5px;
    box-sizing: border-box;
    font-family: 'Roboto', sans-serif;
    font-size: 15px;
    color:#1b314f;
  }

</style>
  