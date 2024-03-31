<script setup>
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue"
import GeometryView from "commonComponents/GeometryView.vue"
import SliderInput from "commonComponents/SliderInput.vue"
import ToggleInput from "commonComponents/ToggleInput.vue"
import DropdownSelector from "commonComponents/DropdownSelector.vue"
import ComputeButton from "commonComponents/ComputeButton.vue"
import Upload3dm from "commonComponents/Upload3dm.vue"

import def from './assets/geometry_copies.gh' //import Grasshopper definition for assets

let sliderName = ref("Count") //must match the Input name in your GH definition!
let sliderValue = ref(10) //default slider value

let toggleName = ref("PlaceRandomly")
let toggleValue = ref(true)

let dropdownName = ref("Color")
let dropdownIndex = ref(0)

const dropdownOptions = [
  { label: "Blue", value: 0 },
  { label: "Pink", value: 1 },
  { label: "Orange", value: 2 }
];

let encodedFile = ref(null);
let isButtonDisabled = ref(true)

///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)


function updateValue(newValue, parameterName) {

  if (parameterName === sliderName.value) {
    sliderValue.value = newValue
  }

  else if (parameterName === toggleName.value) {
    toggleValue.value = newValue
  }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  console.log(parameterName + " : " + newValue)
}

function update3dmData(newData) {
  encodedFile.value = newData
  isButtonDisabled.value = false
  compute.value = true

}

function receiveMetadata(newValue) {
  console.log(newValue)
  metadata.value = newValue
}

function runCompute(newVal) {
  compute.value = newVal
}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    ["encodedFile"]: encodedFile.value,
    [sliderName.value]: Number(sliderValue.value),
    [toggleName.value]: Boolean(toggleValue.value),
    [dropdownName.value]: Number(dropdownIndex.value)
  };

  return data
})

</script>


<template>
  <div id="sidebar" class="container">
    <h3>Project title</h3>
    <p>Project description. Explain the purpose of the project and how
      to use your definition.
    </p>
    <Upload3dm @encoded3dm="update3dmData" />

    <SliderInput :title="sliderName" :min="1" :max="10" :step="1" :val="5" @update="updateValue"></SliderInput>
    <ToggleInput :title="toggleName" :val="true" @update="updateValue"></ToggleInput>

    <DropdownSelector :title="dropdownName" :options="dropdownOptions" :val="dropdownIndex" @update="updateValue"
    />


    <ComputeButton title="Compute" @click="runCompute" :isDisabled="isButtonDisabled" />
  </div>

  <div id="viewer" class="container">
    <GeometryView :data="computeData" :path="path" :runCompute="compute" @updateMetadata="receiveMetadata" />
  </div>
</template>
  
 
  
<style scoped>

.container{
background-color: rgb(59, 59, 59);
border-style: dotted;
border-width: 1px;
}

#content {

  display: flex;
  height: calc(100vh - 68px);
}

#sidebar {

  width: 310px;
  padding: 10px;
}

#viewer {
  width: calc(100% - 310px);
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  background: linear-gradient(90deg, #ffffff, #ff0080);
  height: 17px;
  border-radius: 15px;
  margin: 10px 0px;
}

.modern-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 15px;
  background-color: black;
  cursor: pointer;
}
</style>