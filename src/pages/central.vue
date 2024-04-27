<script setup lang="ts">
import {onMounted} from "vue";
let voices = speechSynthesis.getVoices();
let thaiVoice = voices.find(voice => voice.lang === 'th-TH');

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

  let utterance = new SpeechSynthesisUtterance("พาเชินหมายเลข B005 ตรงโกดังที่ 3");
  utterance.voice = thaiVoice;
  speechSynthesis.speak(utterance);


})
</script>

<template>
  <v-container class="">
    <v-responsive>
      <!-- 3 columns -->
      <v-row class="queue-list">
        <v-col cols="4">
          <h2>รถ 10 ล้อ</h2>
          <v-card class="ma-2">
            <v-card-title>
              <h3>A001</h3>
            </v-card-title>
            <v-card-text>
              <p>Warehouse 5</p>
            </v-card-text>
          </v-card>
          <v-card class="ma-2">
            <v-card-title>
              <h3>
                A002
              </h3>
            </v-card-title>
            <v-card-text>
              <p>Warehouse 5</p>
            </v-card-text>
          </v-card>
        </v-col>
        <v-col cols="4">
          <h2>รถ 6 ล้อ</h2>

        </v-col>
        <v-col cols="4">
          <h2>รถกระบะ</h2>

        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<style scoped>
.queue-list{
  & h2{
    text-align: center;
  }
  & h3{
    text-align: center;
    font-size: 2rem;
  }
  & p{
    text-align: center;
  }

}
</style>