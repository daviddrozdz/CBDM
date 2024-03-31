<script setup>

import { ref } from "vue";

// Define props coming from parent component
const props = defineProps(["title", "options"]);
const title = ref(props.title)

// Define events that will be emitted to parent
const emits = defineEmits(["update"]);

// Value that is binded with selection option
var selectedOption = ref(0);

// Function called on v-on:input
function emitValueUpdate() {
  emits("update", selectedOption.value, title.value);
}

</script>

<template>

  <form class="definition-input">
    <!-- <label class="input-title">{{ title }} index: {{ selectedOption }} </label> -->
    <label class="input-title">{{ title }} </label>

    <select
      v-model="selectedOption"
      class="dropdown"
      v-on:change="emitValueUpdate"
    >
      <option
        v-for="option in options"
        v-bind:key="option.label"
        v-bind:value="option.value"
      >
        {{ option.label }}
      </option>
    </select>
  </form>

</template>



<style scoped>

.input-title {
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
}

.dropdown {
  height: 40px;
  margin: 10px 0 0;
  padding: 0 8px;
  border: 1px solid black;
  border-radius: 8px;
  font-family: 'Roboto', sans-serif;
  font-size: 15px;
  width: 100%;
  cursor: pointer;
  border: 1px solid #ccc;
  color: #1b314f;
}

</style>