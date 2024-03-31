<script setup>
import { ref } from "vue";
// Define properties that you will be able to access from parent component. 
// Those properties will be binded from parent to child. 
// Available JavaScript types: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures
const props = defineProps(['title', 'min', 'max', 'step', 'val'])

const title = ref(props.title)

// Define events that will be accessible from parent component
const emits = defineEmits(['update'])



var sliderValue = ref(props.val)  

function emitValueUpdate()
{
  emits("update", sliderValue.value, title.value)
}



</script>

<template>
<div>
	<form class="definition-input">
			<label class="input-title" for="range-slider">{{ title }}: {{ sliderValue }}</label>

			<input type="range" class="modern-range"
			:min="min" 
      :max="max" 
      :step="step"
      :val = "val"
			v-model="sliderValue" 
      @mouseup="emitValueUpdate"
			/>
      
	</form>
</div>


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
.modern-range {
  -webkit-appearance: none;
  width: 100%;
  height: 10px;
  border-radius: 5px;
  background: linear-gradient(to right, #000 0%, #8a8a8a 50%, #6d6d6d 50%, #8d8d8d 100%);
  outline: none;
  margin: 8px 0px;
  background-image: -webkit-repeating-linear-gradient(left, #8a8a8a, #8a8a8a 1px, transparent 1px, transparent 4px);
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #000;
  cursor: pointer;
}

.modern-range::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #000;
  cursor: pointer;
}

.modern-range::-ms-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: #000;
  cursor: pointer;
}
</style>