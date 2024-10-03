<script setup>

import { onMounted } from 'vue'
import EventBus from '../EventBus';

onMounted(() => {

  document.querySelector('#controlsMore').addEventListener('click', (e) => {
    EventBus.$emit('event name',{'size': 'more'});
  });
  
  document.querySelector('#controlsLess').addEventListener('click', (e) => {
    EventBus.$emit('event name',{'size': 'less'});
  });
  
});

</script>

<template>
  <div id='controlsMore'><p>+</p></div>
  <div id='controlsLess'><p>-</p></div>
</template>
