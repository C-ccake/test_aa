<template>
  <v-stage>
    <v-layer ref="layer_ref"> </v-layer>
  </v-stage>
</template>
<script setup>
import { ref, onMounted } from "vue";

let layer_ref = ref(null);
let layer;
onMounted(() => {
  layer = layer_ref.value.getNode();
  console.log(layer);
});
</script>
