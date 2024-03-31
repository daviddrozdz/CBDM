<script setup>
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"

// Import other Vue components in order to add them to a template.
import Header from "commonComponents/Header.vue"
import GeometryView from "./components/GeometryView.vue"
import Loading from "commonComponents/Loading.vue"
import { download } from "@/scripts/compute.js"

//Import and define Store
import {useComputeStore} from "@/stores/computeStore.js"
const computeStore = useComputeStore()

import def from './assets/metaballTable.gh' //import Grasshopper definition for assets

let dimensionSlider = ref("dimension") //must match the Input name in your GH definition!
let dimensionSliderValue = ref(800) //default slider value

let heightSlider = ref("height") //must match the Input name in your GH definition!
let heightSliderValue = ref(300) //default slider value

let points = ref("points")
let pointsValue = ref("")
createInitalRandomPoints()

///.............................................
let path = def //path to the Grasshopper definition
let data = ref({})
let compute = ref(false)

function createInitalRandomPoints(){
  // generate random points
  let points = []
  const cntPts = 3
  const bndX = dimensionSliderValue.value/2
  const bndY = dimensionSliderValue.value/2

  for (let i = 0; i < cntPts; i++) {
    const x = Math.random() * (bndX - -bndX) + -bndX
    const y = Math.random() * (bndY - -bndY) + -bndY
    const z = 0

    const pt = "{\"X\":" + x + ",\"Y\":" + y + ",\"Z\":" + z + "}"

    points.push(pt)
  }
  pointsValue.value = points
}

function updateValue(newValue, parameterName) {

  if (parameterName === dimensionSlider.value) {
    dimensionSliderValue.value = newValue
  }
  else if (parameterName === heightSlider.value) {
    heightSliderValue.value = newValue
  }
}

function receiveMetadata(newValue) {
  metadata.value = newValue
}

function runCompute(newVal) {
  compute.value = newVal
}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    [dimensionSlider.value]: Number(dimensionSliderValue.value),
    [heightSlider.value]: Number(heightSliderValue.value),
    [points.value]: pointsValue.value
  };
  return data
})

function updatePoints(newPoints){
  pointsValue.value = newPoints
}

</script>


<template>
  <div id="sidebar" class="container">
    <h3>Metaball Table Example</h3>
    <p>Credits: Luis Fraguada (McNeel)</p>
    <a href="https://github.com/mcneel/compute.rhino3d.appserver/tree/main/src/examples/metaballTable" style="color:white">
      Original Code
    </a>
    <br>
    <br>
    <button @click="download('downloadfilename')">
      Download 3dm
    </button>
    <br><br>    
    <div class="metadata bold">Point 1 Position: </div>
    <div class="metadata"> X: {{JSON.parse(pointsValue[0]).X.toFixed(2)}}, Y:{{JSON.parse(pointsValue[0]).Y.toFixed(2)}}</div>
    <br>
    <div class="metadata bold">Point 2 Position: </div>
    <div class="metadata"> X: {{JSON.parse(pointsValue[1]).X.toFixed(2)}}, Y:{{JSON.parse(pointsValue[1]).Y.toFixed(2)}}</div>
    <br>
    <div class="metadata bold">Point 3 Position: </div>
    <div class="metadata"> X: {{JSON.parse(pointsValue[2]).X.toFixed(2)}}, Y:{{JSON.parse(pointsValue[2]).Y.toFixed(2)}}</div>
    <br>
  </div>

  <div id="viewer" class="container">
    <GeometryView :data="computeData" :path="path" :runCompute="compute" @gumballMove="updatePoints" />
    <Loading class="loader-overlay" v-if="computeStore.computing"></Loading>
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
.metadata{
  color:white
}
.bold{
  font-weight:bold
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
.loader-overlay {
  position: fixed;
  top: 0;
  left: 0; 
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 999;
}
</style>