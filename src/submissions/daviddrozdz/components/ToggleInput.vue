<script setup>
import { ref } from "vue"

// Define props
const props = defineProps(['title', 'val'])

const title = ref(props.title)
// Define emits
const emits = defineEmits(['update'])

// Define variables with refs
var toggleValue = ref(props.val)

function emitValueUpdate() {
  emits("update", toggleValue.value, title.value)
}

</script>



<template>
  <form class="definition-input">
    <label class="input-title" for="toggle">{{ title }}: {{ toggleValue }}</label>

    <div class="checkbox-wrapper-22">
      <label class="switch" for="checkbox">
        <input
          type="checkbox"
          id="checkbox"
          v-model="toggleValue"
          v-on:change="emitValueUpdate"
        />
        <div class="slider round" :class="{ active: toggleValue }"></div>
      </label>
    </div>
  </form>
</template>



<style scoped>
.input-title {
  font-family: 'Roboto', sans-serif;
  font-size: 16px;
}

.slider.round {
  background-color: rgb(156, 156, 156);
}

.slider.round.active {
  background-color: #2391ff;
}

.slider.round:before {
  background-color: white;
}
</style>