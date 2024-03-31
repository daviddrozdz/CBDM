<template>
  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container"></div>

</div>
</template>

<script setup>
// Imports;
import { onMounted, onUpdated, watch } from 'vue'
import * as THREE from "three"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls"
import { TransformControls } from 'three/addons/controls/TransformControls.js';
import { Rhino3dmLoader } from "three/addons/loaders/3DMLoader.js"
import { runCompute } from "@/scripts/compute.js"
import { loadRhino } from "@/scripts/compute.js";


const loader = new Rhino3dmLoader()
loader.setLibraryPath('https://cdn.jsdelivr.net/npm/rhino3dm@8.0.0-beta2/')


const props = defineProps(["data", "path", "runCompute"]);
const emits = defineEmits(["gumballMove"]);  


watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
})


// Three js objects
let renderer, camera, scene,  controls, container
let points = []


function init() {
  // https://threejs.org/docs/#api/en/renderers/WebGLRenderer
  // This object will render our scene
  renderer = new THREE.WebGLRenderer()

  container = document.getElementById("threejs-container")

  // Rendered needs to know what's the size of the scene. 
  renderer.setSize(container.offsetWidth, container.offsetHeight)

  // We are taking element defined in <template> and appending our render to it. 
    container.appendChild(renderer.domElement)

  // https://threejs.org/docs/#api/en/cameras/PerspectiveCamera
  camera = new THREE.PerspectiveCamera( 45, window.innerWidth/window.innerHeight, 1, 10000 )
  camera.position.x = 1000
  camera.position.y = 1000
  camera.position.z = 1000
  camera.up.set(0,0,1)

  // https://threejs.org/docs/?q=scene#api/en/scenes/Scene
  scene = new THREE.Scene()
  scene.background = new THREE.Color("#f5f6fa")

  //Set default three.js up
  THREE.Object3D.DEFAULT_UP = new THREE.Vector3(0, 0, 1);

  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement)

  // add some ambient light here
  const ambientlight = new THREE.AmbientLight(0xffffff, 1)
  ambientlight.position.set(0, 0, 0)
  scene.add(ambientlight)

  const directionalLight = new THREE.DirectionalLight(0xffffff, 3)
  directionalLight.position.set(0, 1, 0)
  scene.add(directionalLight)

  // add fun shape
  animate()
}

function visualizeGumballPoints() {
  let points = props.data.points
  for (let i = 0; i < points.length; i++) {
    //viz in three
    const icoGeo = new THREE.IcosahedronGeometry(25)
    const icoMat = new THREE.MeshNormalMaterial()
    const ico = new THREE.Mesh( icoGeo, icoMat )
    ico.name = 'ico'
    ico.position.set( JSON.parse(points[i]).X, JSON.parse(points[i]).Y, JSON.parse(points[i]).Z)
    scene.add( ico )
    
    //Add transform (gumball) controls
    let tcontrols = new TransformControls( camera, renderer.domElement )
    tcontrols.enabled = true
    tcontrols.attach( ico )
    tcontrols.showZ = false
    tcontrols.addEventListener('dragging-changed', onChange )
    scene.add(tcontrols)
  }
}

let dragging = false
//When transform controls are changing, record and store control position
function onChange() {
  dragging = ! dragging
  if ( !dragging ) {
    // update points position
    points = []
    scene.traverse(child => {
      if ( child.name === 'ico' ) {
        const pt = "{\"X\":" + child.position.x + ",\"Y\":" + child.position.y + ",\"Z\":" + child.position.z + "}"
        points.push( pt )
      }
    }, false)

    emits("gumballMove", points)
    props.data.points = points
    compute()
    controls.enabled = true

    return 
  }
  controls.enabled = false

}


// this function runs Compute
async function compute() {
  console.log("Runnning compute... \ndata sent: ", props.data)
  const doc = await runCompute(props.data, props.path)

  // clear objects from scene
  scene.traverse((child) => {
    if ( child.userData.hasOwnProperty( 'objectType' ) && child.userData.objectType === 'File3dm') {
      scene.remove( child )
    }
  })


  const buffer = new Uint8Array(doc.toByteArray()).buffer;
  loader.parse(buffer, function (object) {
    ///////////////////////////////////////////////////////////////////////
    // add object graph from rhino model to three.js scene
    object.traverse((child) => {
    
      if (child.isLine) {

        // console.log(child)
          
        if (child.userData.attributes.userStrings!= undefined && child.userData.attributes.userStrings.length > 0) {
            //get color from userStrings
            const colorData = child.userData.attributes.userStrings[0]
            const col = colorData[1]

            //convert color from userstring to THREE color and assign it
            const threeColor = new THREE.Color("rgb(" + col + ")")
            const mat = new THREE.LineBasicMaterial({ color: threeColor })
            child.material = mat

        }

      }
    })


    scene.add(object)

    // zoom to extents
    for (let child of scene.children) {
        if (child.type == "Object3D") {
          // zoom to extents
          //zoomCameraToSelection(camera, controls, child.children)

        }
      }
      

    console.log("Compute done")
  });
}



// for controls update
function animate() {
  // https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame
  // native Javascript function that tells your browser you are animating!
  requestAnimationFrame(animate);
  controls.update();
  renderer.render(scene, camera);
}

// This will be run whenever the window is resized
window.addEventListener("resize", onWindowResize);
function onWindowResize() {

  const height = container.offsetHeight
  const width = container.offsetWidth

  camera.aspect = width/ height
  camera.updateProjectionMatrix();


  renderer.setSize(width, height)

}

/**
 * Helper function that behaves like rhino's "zoom to selection", but for three.js!
 */
function zoomCameraToSelection(camera, controls, selection, fitOffset = 1.1) {

  const box = new THREE.Box3();

  for (const object of selection) {
    if (object.isLight) continue
    box.expandByObject(object);
  }

  const size = box.getSize(new THREE.Vector3());
  const center = box.getCenter(new THREE.Vector3());

  const maxSize = Math.max(size.x, size.y, size.z);
  const fitHeightDistance = maxSize / (2 * Math.atan(Math.PI * camera.fov / 360));
  const fitWidthDistance = fitHeightDistance / camera.aspect;
  const distance = fitOffset * Math.max(fitHeightDistance, fitWidthDistance);

  const direction = controls.target.clone()
    .sub(camera.position)
    .normalize()
    .multiplyScalar(distance);
  controls.maxDistance = distance * 10;
  controls.target.copy(center);

  camera.near = distance / 100;
  camera.far = distance * 100;
  camera.updateProjectionMatrix();
  camera.position.copy(controls.target).sub(direction);

  controls.update();

}



// This will be run whenever this component is instantiated
onMounted(async() => {
  init()
  await loadRhino()
  visualizeGumballPoints()
  compute();
})

// //onUpdated is called when an input prop is changed
// // this runs compute whenever the input data changes
// onUpdated(() => {
//   compute();
// })



</script>

<style scoped>


#viewport {

  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

#threejs-container {
  height: 100%;
  width: 100%;
  min-width: 200px;
  position:inherit;
}

</style>