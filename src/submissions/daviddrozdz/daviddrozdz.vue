<!--  -->
<!-- Script is in some way equivalent to script.js. This is place to define variables, methods and imports of other Vue compoennts. -->
<!--  -->

<script setup>

// Import js libraries and other Vue components
import { ref, onBeforeMount, computed } from "vue"
import { loadRhino } from "@/scripts/compute.js"
import {useComputeStore} from "@/stores/computeStore.js"
const computeStore = useComputeStore()

// Import other Vue components in order to add them to a template.
// import Header from "commonComponents/Header.vue"
import SliderInput from "./components/SliderInput.vue"
import DropdownSelector from "./components/DropdownSelector.vue"
import ComputeButton from "./components/ComputeButton.vue"
import ToggleInput from "./components/ToggleInput.vue"
import GeometryView from "./components/GeometryView.vue"
import InputField from "./components/InputField.vue"
import Loading from "./components/Loading.vue"

// Define what the component emits
const emits = defineEmits(['updateMetadata', 'gumballMove']) 

// Import Grasshopper definition for assets
import def from './assets/EvaluateYourCity.gh' 

let inputName = ref("Location (N/S)") //must match the Input name in your GH definition!
let inputValue = ref(52.23955) //default slider value

let inputName2 = ref("Location (W/E)") //must match the Input name in your GH definition!
let inputValue2 = ref(20.99155) //default slider value

let dropdownName = ref("Map size") //must match the Input name in your GH definition!
let dropdownIndex = ref(0) //default slider value

const dropdownOptions = [
  { label: "Small", value: 0 },
  { label: "Medium", value: 1 },
  { label: "Large", value: 2 },
];

let dropdownName2 = ref("Amenity") //must match the Input name in your GH definition!
let dropdownIndex2 = ref(0) //default slider value

const dropdownOptions2 = [
  { label: "Parks", value: 0 },
  { label: "Playgrounds", value: 1 },
  { label: "Groceries", value: 2 },
  { label: "Restaurants", value: 3 },
  { label: "Public transport", value: 4 },
  { label: "Education", value: 5 }
];

let toggleName = ref("3D inside") //must match the Input name in your GH definition!
let toggleValue = ref(true) //default slider value

let sliderName = ref("Data radius") //must match the Input name in your GH definition!
let sliderValue = ref(150) //default slider value

let inputPoint = ref("inputPoint")
let inputPointValue = ref("")
createInitialPoint()

//.............................................

const path = def //path to your GH definition
let data = ref({})
let metadata = ref([])
let compute = ref(false)

// function to create initial point
function createInitialPoint(){
  // generate points
  let inputPoint = []
  const cntPt = 1

  for (let i = 0; i < cntPt; i++) {
    const x = 0
    const y = 0
    const z = 25

    const pt = "{\"X\":" + x + ",\"Y\":" + y + ",\"Z\":" + z + "}"

    inputPoint.push(pt)
  }
  inputPointValue.value = inputPoint
}

// function to update value
function updateValue(newValue, parameterName) {

  if (parameterName === inputName.value) {
    inputValue.value = newValue
  }

  if (parameterName === inputName2.value) {
    inputValue2.value = newValue
  }

  else if (parameterName === dropdownName.value) {
    dropdownIndex.value = newValue
  }

  else if (parameterName === dropdownName2.value) {
    dropdownIndex2.value = newValue
  }

  else if (parameterName === toggleName.value) {
    toggleValue.value = newValue
  }

  else if (parameterName === sliderName.value) {
    sliderValue.value = newValue
  }

  // console.log(parameterName + " : " + newValue)

}

// Show error message when error happens
const findError = computed(() => {
  const errorItem = metadata.value.find(item => item.name === 'Error');
  return errorItem ? errorItem.value : null;
});

// function to receive metadata
// function receiveMetadata(newValue) { //this function is called when the compute is done
//   console.log(newValue)
//   metadata.value = newValue
// }

// function to receive metadata v2
function receiveMetadata(newValue) { //this function is called when the compute is done
  console.log(newValue)

  let AmenitiesMeta = []
  let locationsX = []
  let locationsY = []
  let locationsZ = []

  newValue.forEach(element => {
    //Check if element is metadata --> add to metadata array
    if (element.name =='AmenitiesM'){
      AmenitiesMeta.push(element)
    }
    //Check if element is location ==> add to location array
    else if (element.name == 'AmenitiesX'){
      locationsX.push(element)
    }
    //Check if element is location ==> add to location array
    else if (element.name == 'AmenitiesY'){
      locationsY.push(element)
    }
     //Check if element is location ==> add to location array
     else if (element.name == 'AmenitiesZ'){
      locationsZ.push(element)
    }
  });
  
  console.log(AmenitiesMeta)
  console.log(locationsX)
  console.log(locationsY)
  console.log(locationsZ)

  metadata.value = newValue

}


// function to run Compute
function runCompute(newVal) {
  compute.value = newVal
}

// a computed ref. Vue will keep track of this and update it
const computeData = computed(() => {
  data = {
    [inputName.value]: Number(inputValue.value),
    [inputName2.value]: Number(inputValue2.value),
    [dropdownName.value]: Number(dropdownIndex.value),
    [dropdownName2.value]: Number(dropdownIndex2.value),
    [toggleName.value]: Boolean(toggleValue.value),
    [sliderName.value]: Number(sliderValue.value),
    [inputPoint.value]: inputPointValue.value,
  };

  return data
})

// function to update point
function updatePoint(newPoint){
  inputPointValue.value = newPoint
}

onBeforeMount( () => {
})

</script>



<!--  -->
<!-- Template is a HTML-based syntax that allows you to bind the rendered DOM elements with data, objects, functions etc. -->
<!--  -->

<template>
  
  <!-- SIDEBAR RIGHT -->
  <div id="sidebar" class="container">

    <p>
      <img src="./assets/Project-logo.png" class=logo alt="Logo">
    </p>

    <h3>
      About the app
    </h3>

    <p>
      This app will help you with viewing and getting information on different types of amenities (like parks, playgrounds, restaurants etc.) near selected location, based on the available OSM data. 
      <br>
      <br>
      1. To start, first go to: <a href="https://www.openstreetmap.org" target="_blank" class="osm-link">www.openstreetmap.org</a>, and find the location you are interested in.
      <br>
      <br>
      2. Zoom to a desired location, right click the map in the desired spot and select "Show adress".
      <br>
      <br>
      3. From the opened side panel, paste the first line of the geographical location in the "Location (N/S)" input field, and the second line in the "Location (W/E)" input field.
      <br>
      <br>
      4. Set the map size that you want to analyze from the first dropdown list.
      <br>
      <br>
      5. Choose the type of amenity you are interested in from the second dropdown list (based on the <a href="https://wiki.openstreetmap.org/wiki/Map_features" target="_blank" class="osm-link">OSM keys</a>).
      <br>
      <br>
      6. For better display, choose the radius of the area you are interested in, and if you want to see 3D building geometry inside or outside it. Of course, you can move it however you want!
      <br>
      <br>
      7. Finally, click the "COMPUTE" button and view the results!

    </p>

  </div>

  <!-- SIDEBAR RIGHT -->
  <div id="sidebar2" class="container">

    <h3>
      Project variables
    </h3>

    <InputField
    :title="inputName"
    :min="-90"
    :max="90"
    :step="0.00001"
    :val="52.23955"
    @update="updateValue">
    </InputField>

    <InputField
    :title="inputName2"
    :min="-180"
    :max="180"
    :step="0.00001"
    :val="20.99155"
    @update="updateValue">
    </InputField>

    <DropdownSelector 
    :title="dropdownName" 
    :options="dropdownOptions" 
    :val="dropdownIndex" 
    @update="updateValue">
    </DropdownSelector>

    <DropdownSelector 
    :title="dropdownName2" 
    :options="dropdownOptions2" 
    :val="dropdownIndex2" 
    @update="updateValue">
    </DropdownSelector>

    <SliderInput 
    :title="sliderName" 
    :min="100" 
    :max="500" 
    :step="1" 
    :val="150" 
    @update="updateValue">
    </SliderInput>

    <ToggleInput 
    :title="toggleName" 
    :val="true" 
    @update="updateValue">
    </ToggleInput>

    <ComputeButton title="Compute" @click="runCompute"/>

    <br>
    <br>

    <h3>
      Results
    </h3>

    <p> 
      Amount of amenities in the area: <span v-if="metadata[0]">{{ metadata[0].value }}</span><span v-else>...</span>
    </p>

    <p> 
      Site area (km2): <span v-if="metadata[1]">{{ metadata[1].value }}</span><span v-else>...</span>
    </p>

    <p> 
      Amenities per km2: <span v-if="metadata[2]">{{ metadata[2].value }}</span><span v-else>...</span>
    </p>





    <p> 
      Amenities data: <span v-if="metadata[2]">{{ metadata[2].value }}</span><span v-else>...</span>
    </p>


    <div v-if="metadata && metadata.length > 0" class="results">
      <template v-if="findError">
        <p>ERROR:</p>
        <p>{{ findError }}</p>
      </template>
      <template v-else>
        <p>RESULT:</p>
        <p>In  meed by evaporation.</p>
      </template>
    </div>


  </div>

  <!-- OVERLAY / DISPLAY -->
  <div id="viewer" class="container">

    <GeometryView 
    :data="computeData" 
    :path="path" 
    :runCompute="compute" 
    @updateMetadata="receiveMetadata"
    @gumballMove="updatePoint" 
    />
    
    <!-- Loading -->
    <div id="viewer" class="container">
    <Loading class="loader-overlay" v-if="computeStore.computing"></Loading>
    </div>

  </div>

</template>
  

 
<!--  -->
<!-- Style is for CSS styling -->
<!--  -->

<style scoped>

.results 
{
  position: absolute; 
  bottom: 20px; 
  right: 20px; 
  margin-top: 0; 
  background-color: gainsboro;
  border-radius: 5px;
  padding: 10px;
  width: auto; 
  max-width: 300px; 
  z-index: 10; 
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
}

.logo {
  width: 100%;
  height: 100%;
  margin: 0 auto;
  display: block;
  padding: 0px;
}

.osm-link {
  color: #51b1ff;
  text-decoration: underline;
}

.container {
  background-color: rgb(66, 66, 66);
  border-style: solid;
  border-width: 1px;
}

#content {
  display: flex;
  height: calc(100vh);
  font-family: 'Roboto', sans-serif;
}

#sidebar {
  position: absolute;
  width: 320px;
  padding: 15px;
  top: 80px;
  left: 10px;
  border-radius: 15px;
  background-color: rgb(63, 63, 63);
  opacity: 1;
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
}

#sidebar2 {
  position: absolute;
  width: 320px;
  padding: 15px;
  top: 80px;
  right: 10px;
  border-radius: 15px;
  background-color: rgb(63, 63, 63);
  opacity: 1;
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
}

#viewer {
  width: 100%;
}

.modern-range {
  -webkit-appearance: none;
  width: 100%;
  background: linear-gradient(90deg, #ffffff, #1897ff);
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