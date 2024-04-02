<template>

  <div id="viewport">
    <!-- To this element we will append our 3d scene. -->
    <div id="threejs-container">
  </div>

  <button @click="setTopDownView">Top view</button>

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
const emits = defineEmits(['updateMetadata', 'gumballMove'])  

watch(() => props.runCompute, (newValue) => {
  if (newValue) {
    compute();
  }
})

// Three js objects
let renderer, camera, scene, controls, container
let inputPoint= []

// function to initialize the scene
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
  camera = new THREE.PerspectiveCamera(75, container.offsetWidth / container.offsetHeight, 0.1, 1000)
  camera.position.set(0, 40, 40)
  camera.up.set(0, 0, 1)

  // https://threejs.org/docs/?q=scene#api/en/scenes/Scene
  scene = new THREE.Scene()
  const linearGradientUrl = createGradientBackground("#FDFFFE", "#1998FD");
  const texture = new THREE.TextureLoader().load(linearGradientUrl, function (texture) {
  scene.background = texture;
  // scene.background = new THREE.Color("#f5f6fa")
});

  //Set default three.js up
  THREE.Object3D.DEFAULT_UP = new THREE.Vector3(0, 0, 1);

  // orbit controls
  controls = new OrbitControls(camera, renderer.domElement)

  // add  ambient light
  const ambientlight = new THREE.AmbientLight(0xffffff, 4)
  ambientlight.position.set(0, 0, 0)
  scene.add(ambientlight)

  // add directional light
  const directionalLight = new THREE.DirectionalLight(0xffffff, 5)
  directionalLight.position.set(0, 0, 1)
  scene.add(directionalLight)

  // add fun shape
  animate()

  // add text sprite
	var spritey = makeTextSprite( "Map centerpoint", 
		{ fontsize: 30, 
      borderColor: {r:255, g:0, b:0, a:1.0}, 
      backgroundColor: {r:255, g:100, b:100, a:0.8} 
    } );
	spritey.position.set(0,0,35);
	scene.add( spritey );

}

// function to create gradient background
function createGradientBackground(colorTop, colorBottom) {
  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d");
  const gradient = context.createLinearGradient(0, 0, 0, window.innerHeight);

  gradient.addColorStop(0, colorTop);
  gradient.addColorStop(1, colorBottom);

  context.fillStyle = gradient;
  context.fillRect(0, 0, canvas.width, canvas.height);

  return canvas.toDataURL();
}

// function to make text sprite
function makeTextSprite( message, parameters )
{
	if ( parameters === undefined ) parameters = {};
	
	var fontface = parameters.hasOwnProperty("fontface") ? 
		parameters["fontface"] : "Arial";
	
	var fontsize = parameters.hasOwnProperty("fontsize") ? 
		parameters["fontsize"] : 18;
	
	var borderThickness = parameters.hasOwnProperty("borderThickness") ? 
		parameters["borderThickness"] : 4;
	
	var borderColor = parameters.hasOwnProperty("borderColor") ?
		parameters["borderColor"] : { r:0, g:0, b:0, a:1.0 };
	
	var backgroundColor = parameters.hasOwnProperty("backgroundColor") ?
		parameters["backgroundColor"] : { r:255, g:255, b:255, a:1.0 };

    var spriteAlignment =
    parameters.hasOwnProperty("alignment") &&
    parameters["alignment"] instanceof THREE.Vector2
      ? parameters["alignment"]
      : new THREE.Vector2(0, 0);
		
	var canvas = document.createElement('canvas');
	var context = canvas.getContext('2d');
	context.font = "Bold " + fontsize + "px " + fontface;
    
	// get size data (height depends only on font size)
	var metrics = context.measureText( message );
	var textWidth = metrics.width;
	
	// background color
	context.fillStyle   = "rgba(" + backgroundColor.r + "," + backgroundColor.g + "," + backgroundColor.b + "," + backgroundColor.a + ")";
	
  // border color
	context.strokeStyle = "rgba(" + borderColor.r + "," + borderColor.g + "," + borderColor.b + "," + borderColor.a + ")";
	
	// text color
	context.fillStyle = "rgba(0, 0, 0, 1.0)";

	context.fillText( message, borderThickness, fontsize + borderThickness);
	
	// canvas contents will be used for a texture
	var texture = new THREE.Texture(canvas) 
	texture.needsUpdate = true;

  // sprite
	var spriteMaterial = new THREE.SpriteMaterial( 
		{ map: texture } );
	var sprite = new THREE.Sprite( spriteMaterial );

	sprite.scale.set(100,50,1.0);
	return sprite;	

}

// function to visualize gumball points
function visualizeGumballPoint() {
  let inputPoint = props.data.inputPoint
  for (let i = 0; i < inputPoint.length; i++) {
    //viz in three
    const icoGeo = new THREE.IcosahedronGeometry(1)
    const icoMat = new THREE.MeshNormalMaterial()
    const ico = new THREE.Mesh( icoGeo, icoMat )
    ico.name = 'ico'
    ico.position.set( JSON.parse(inputPoint[i]).X, JSON.parse(inputPoint[i]).Y, JSON.parse(inputPoint[i]).Z)
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

// function to update the position of the gumball points
function onChange() {
  dragging = ! dragging
  if ( !dragging ) {
    // update points position
    inputPoint = []
    scene.traverse(child => {
      if ( child.name === 'ico' ) {
        const pt = "{\"X\":" + child.position.x + ",\"Y\":" + child.position.y + ",\"Z\":" + child.position.z + "}"
        inputPoint.push( pt )
      }
    }, false)

    emits("gumballMove", inputPoint)
    props.data.inputPoint = inputPoint

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

  if (doc.metadata) {
    emits("updateMetadata", doc.metadata);
  }

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

      // print all visible elements in console
      // console.log(child)

      // if child is line add color
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
          zoomCameraToSelection(camera, controls, child.children)

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

// function to update the window size
function onWindowResize() {

  const height = container.offsetHeight
  const width = container.offsetWidth

  camera.aspect = width/ height
  camera.updateProjectionMatrix();

  renderer.setSize(width, height)

}

// Helper function that behaves like rhino's "zoom to selection", but for three.js!
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
  visualizeGumballPoint()
  compute();
})

//onUpdated is called when an input prop is changed
// this runs compute whenever the input data changes
// onUpdated(() => {
//   compute();
// })

function setTopDownView() {
  camera.position.set(0, 0, 800);
  camera.lookAt(new THREE.Vector3(0, 0, 0));
  controls.target.set(0, 0, 0);
  controls.update();
}

</script>



<style scoped>

#viewport button {
  font-family: 'Roboto', sans-serif;
  background-color: #1897ff;
  color: white;
  border: none;
  font-weight: bold;
  padding: 10px 20px; 
  text-align: center;
  font-size: 16px;
  cursor: pointer;
  border-radius: 10px;
  transition: all 0.1s ease-in-out;
  position: absolute;
  top: 80px;
  left: 50%; 
  transform: translateX(-50%)
}

button:hover, .customUpload:hover {
    background-color: #007ffd;
    box-shadow: 3px 3px rgb(191, 191, 191);
  }

#viewport {
  height: 100%;
  width: 100%;
  min-width: 200px;
  min-height: 900px;
  position:inherit;
}

#threejs-container {
  height: 100%;
  width: 100%;
  min-width: 200px;
  min-height: 900px;
  position:inherit;
}

</style>