<script setup>
// Imports from packages
import { ref, onMounted } from "vue";
import { loadRhino } from "@/scripts/compute.js";

import Loading from 'commonComponents/Loading.vue'
import WelcomePage from 'commonComponents/WelcomePage.vue'
import StudentA from "./submissions/studentA/StudentA.vue";
import StudentB from "./submissions/studentB/StudentB.vue";
import StudentC from "./submissions/studentC/StudentC.vue";
import daviddrozdz from "./submissions/daviddrozdz/daviddrozdz.vue";

// PINIA STORE
import { store } from './stores/storeSingletons'

var submissions = [
  { name: "Student A", visible: ref(false) },
  { name: "Student B", visible: ref(false) },
  { name: "Student C", visible: ref(false) },
  { name: "Dawid Drozdz", visible: ref(false) }
];

function toggleComponent(item) {
  aboutPageVisible.value=false
  submissions.forEach(
    (menuItem) => (menuItem.visible.value = item === menuItem)
  );
}

function openNav() {
  document.getElementById("mySidenav").style.width = "250px";
}

function closeNav() {
  document.getElementById("mySidenav").style.width = "0";
}

let aboutPageVisible = ref(true)
function aboutPage()
{
  submissions.forEach(submission => submission.visible.value = false)
  store.computing = false
  aboutPageVisible.value = true;
}

onMounted(() => {
  loadRhino();
});
</script>

<template>

<div id="top-bar">
    <div>
       <span class="my-nav" @mouseenter="openNav">  &#9776;</span>
     </div>

    <div id="title-container">

      <h2 class="course-title" style="font-family: 'Roboto Mono', monospace;">
        Digital Tools for Cloud-based Data Management    
      </h2>
      <img class="logo-image" alt="Iaac logo" src="./assets/iaac-white.png" />
    </div>
</div>

<div id="mySidenav" class="sidenav" @mouseleave="closeNav">
    <a href="javascript:void(0)" class="closebtn" 
    :onclick="closeNav" 
    >&times;</a>
    <div class="left-sidebar">
      <h3 class="home-button" @click="aboutPage()">About</h3>
      <hr />

      <ul class="menu">
        <li v-for="(item, index) in submissions" :key="index" :class="{ selected: item.visible.value }"
          @click="toggleComponent(item)">
          {{ item.name }}
        </li>
      </ul>
    </div>
</div>

<div>
  <div id="content">
    <WelcomePage v-if="aboutPageVisible"></WelcomePage>
    <StudentA v-if="submissions[0].visible.value"></StudentA>
    <StudentB v-if="submissions[1].visible.value"></StudentB>
    <StudentC v-if="submissions[2].visible.value"></StudentC>
    <daviddrozdz v-if="submissions[3].visible.value"></daviddrozdz>
  </div>
</div>

<!-- <div v-if="store.computing" class="loading-overlay">
  <div class="loading-container"> 
    <Loading />
  </div>
</div> -->

</template>

<style>
body {

  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}

#top-bar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #1c2127;
}

#title-container {
  display: flex;
  align-items: center;
  color: white;
  margin-right: 1.5rem;
}

.left-sidebar {
  margin: 0px 10px 2px 0px;
  padding: 5px;
}

.closebtn
{
  margin:0px;
  padding:0px;
}

#content{
    
    display: flex;
    height: calc(100vh - 68px);
}

.logo-image {
  height: 3.25rem;
  padding: 0.5rem;
}
.home-button {
  margin: 0px 3px;
}

.home-button:hover {
  color:white;
  cursor: pointer;
}

h2 {
  font-size: 1.125rem;
  font-weight: 600;
  letter-spacing: -0.05em;
  font-size: 1.125rem;
  font-weight: 600;
  letter-spacing: 0.01em;
}

.menu {
  display: flex;
  flex-direction: column;
  list-style: none;
  padding: 10px 0px;
  width: 200px;
  font-size:  1 em;
}

.menu li {
  padding: 7px 7px;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
  -webkit-transition: background-color 0.2s ease-out;
  transition: background-color 0.2s ease-out;
  border-radius: 30px;
  border: solid transparent;
  z-index: 2;
  color:#848d99;
}

.menu li:hover {
  background-color: #f2dd1c4e;
  color:white;
}

.selected {
  background-color: #f2dd1c;
  z-index: 10;
  font-weight: bolder;
  color:#2c3e50 !important;
}

.my-nav {
  font-size:30px;
  cursor:pointer;
  color: white;
}

.sidenav {
  height: 100%;
  width: 0;
  position: fixed;
  z-index: 10000;
  top: 0;
  left: 0;
  background-color: #1c2127;
  overflow-x: hidden;
  transition: 0.5s;
  padding-top: 45px;
}

.sidenav a {
  padding: 8px 8px 8px 32px;
  text-decoration: none;
  font-size: 25px;
  color: #818181;
  display: block;
  transition: 0.3s;
}

.sidenav a:hover {
  color: #f1f1f1;
}

.sidenav .closebtn {
  position: absolute;
  top: 0;
  right: 25px;
  font-size: 36px;
  margin-left: 50px;
}

@media screen and (max-height: 450px) {
  .sidenav {
    padding-top: 15px;
  }

  .sidenav a {
    font-size: 18px;
  }
}

.loading-overlay {
  position: absolute;
  top:82px;
  left: 0;
  backdrop-filter: blur(2px);
  -webkit-mask-image: linear-gradient(to bottom,rgba(168, 181, 192, 0.649) 90%,transparent);
  mask-image: linear-gradient(to bottom,rgba(168, 181, 192, 0.649) 90%,transparent);
  background-image: linear-gradient(rgba(168, 181, 192, 0.649) 80%, transparent); /* bg-stone-900 with opacity 90% */
  height: calc(100% - 81px);
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  flex-direction: column;
  user-select: all;
  z-index: 100;
}

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
</style>
