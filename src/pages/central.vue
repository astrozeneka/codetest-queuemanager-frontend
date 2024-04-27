<script setup lang="ts">
import {onMounted} from "vue";

onMounted(() => {

  // Listen to websocket http://localhost:8000/ws
  const ws = new WebSocket('ws://localhost:8000/ws');
  ws.onopen = () => {
    console.log('connected');
  }
  ws.onmessage = (event) => {
    console.log(event.data);
  }
  ws.onclose = () => {
    console.log('disconnected');
  }
  ws.onerror = (error) => {
    console.error(error);
  }

})
</script>

<template>
  <v-container class="fill-height">
    <v-responsive>

      <h1>Queue</h1>

    </v-responsive>
  </v-container>
</template>

<style scoped>

</style>