<script setup>

import { ref } from "vue";
// import imageSrc from '@/assets/datamgmt-commits.png'

var submissions = [
  { name: "Student A", visible: ref(false) },
  { name: "Student B", visible: ref(false) },
  { name: "Student C", visible: ref(false) },
  { name: "Student D", visible: ref(false) },
  { name: "Dawid Drozdz", visible: ref(false) },

];
</script>



<template>

   <div class="other-stuff">
    <h4 class="home-page-titles">Master in Advanced Computation for Architecture & Design </h4>
    <h2 class="home-page-titles main">Cloud-based Data Management</h2>
        <h2 class="home-page-titles">Faculty</h2>
        <p class="names">David Andres Leon</p>
        <p class="names">Justyna Szychowska</p>


        <h2 class="home-page-titles">Faculty assistant</h2>
        <p class="names">Sophie Moore</p>

        <h2 class="home-page-titles">BIMSC24 module</h2>
        <ul class="student-list">
            <li v-for="(item, index) in submissions" :key="index">
            {{ item.name }}
            </li>
        </ul>

   </div>
   <div class="image-container">
    <!-- <img class="scrollable-image" :src="imageSrc" alt="Gitflow" /> -->
  </div>

</template>



<style scoped>

.image-container {
  max-width: 50%;
  height: auto;
  overflow-y: scroll;
  background-color: #f0f0f0;
  display: block;
margin-left: auto;
}

.other-stuff  {
  margin: 5px 1% 20px 15%;
  display: block;
  max-width: 500px;
}

.scrollable-image {
  width: 100%;
  height: auto;
}

.home-page-titles{
    margin:0px;
    text-align: left;
    padding-left:0px;
    padding-bottom: 1px;
}

.names {
    margin-top: 2px;
    color: black;
}

.student-list {
   list-style-type: none; 
   color: black;
   margin-left:0px;
   padding-left:0px;
}

/* Customize the scrollbar */
.image-container::-webkit-scrollbar {
  width: 15px;
}

.image-container::-webkit-scrollbar-track {
  background-color: #1c2127;
}

.image-container::-webkit-scrollbar-thumb {
  background-color: #f2dd1cd0;
  border-radius: 10px;
}

.image-container::-webkit-scrollbar-thumb:hover {
  background-color: #f2dd1c;
}

</style>