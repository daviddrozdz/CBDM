<script setup>
import { ref } from "vue"

// Define props
const props = defineProps(['title', 'val'])

const title = ref(props.title)
// Define emits
const emits = defineEmits(['update'])

// Define variables with refs
var toggleValue = ref(props.val)

function emitValueUpdate()
{
  emits("update", toggleValue.value, title.value)
}
</script>


<template>
    <form class = "definition-input">
        <label class="input-title" for="toggle"> {{ title }}: {{  toggleValue }}</label>

        <div class="checkbox-wrapper-22">
        <label class="switch" for="checkbox">
            <input type="checkbox" id="checkbox" 
            v-model="toggleValue" 
            v-on:change="emitValueUpdate"/>

            <div class="slider round"></div>
        </label>
        </div>
    </form>
</template>

<style scoped>
label {
    display: inline-block;
    margin-bottom: 8px;
    color: 'White';
    font-family: 'Arial', sans-serif; 
    font-size: 16px; 
    font-weight: bold; 
}

.checkbox-wrapper-22 .switch {
  display: inline-block;
  height: 28px;
  position: relative;
  width: 80px;
  margin: 9px 0px;
  box-shadow: 0 2px 4px 0 rgba(0,0,0,0.2); /* Add shadow for depth */
}

.checkbox-wrapper-22 .switch input {
  display: none;
}

.checkbox-wrapper-22 .slider {
  background-color: #ccc;
  bottom: 0;
  cursor: pointer;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
  transition: .4s;
  box-shadow: inset 0 2px 4px rgba(0,0,0,0.2); /* Inner shadow for a subtle inset effect */
}

.checkbox-wrapper-22 .slider:before {
  background-color: #fff;
  bottom: 4px;
  content: "";
  height: 20px;
  left: 4px;
  position: absolute;
  transition: .4s;
  width: 20px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.3); /* Add shadow to the slider button for depth */
}

.checkbox-wrapper-22 input:checked + .slider {
  background-color: #f099b5; /* Fun pink shade */
  /* Add a gradient for a more vibrant look */
  background-image: linear-gradient(45deg, #f099b5 0%, #f9d423 100%);
}

.checkbox-wrapper-22 input:checked + .slider:before {
  transform: translateX(51px); /* Adjust if necessary based on the size */
}

.checkbox-wrapper-22 .slider.round {
  border-radius: 34px;
}

.checkbox-wrapper-22 .slider.round:before {
  border-radius: 50%;
}

/* Add a hover effect for interactive feedback */
.checkbox-wrapper-22 .slider:hover {
  background-color: #bfbfbf; /* Lighter shade when hovering over the slider */
}

/* Checked state hover effect */
.checkbox-wrapper-22 input:checked + .slider:hover {
  background-image: linear-gradient(45deg, #e089a8 0%, #f7e287 100%);
}
  
  </style>