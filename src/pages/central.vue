<script setup lang="ts">
import {onMounted, ref} from "vue";
let voices = speechSynthesis.getVoices();
let thaiVoice = voices.find(voice => voice.lang === 'th-TH');


const prepareOneCallNumber = (queue)=>{
  let call_initial = {
    "10-wheels": "A",
    "6-wheels": "B",
    "pickup": "C"
  }[queue.category]
  queue.call_number = call_initial +' '+ queue.call_id.toString().padStart(3, '0')
  return queue
}

const prepareCallNumbers = (queueList)=>{
  for (let entity of queueList) {
    let call_initial = {
      "10-wheels": "A",
      "6-wheels": "B",
      "pickup": "C"
    }[entity.category]
    entity.call_number = call_initial + entity.call_id.toString().padStart(3, '0')
  }
  return queueList
}

// Variables bound to the interface
let queueList10Wheels = ref([])
let queueList6Wheels = ref([])
let queueListPickup = ref([])

onMounted(() => {

  // Listen to websocket http://localhost:8000/ws
  const ws = new WebSocket('ws://localhost:8000/ws/central');
  ws.onopen = () => {
    console.log('connected');
  }
  ws.onmessage = (event) => {
    let data = JSON.parse(event.data);
    console.log(data)
    if (data['category'] === '10-wheels')
      queueList10Wheels.value = prepareCallNumbers(data['entityList'])
    else if (data['category'] === '6-wheels')
      queueList6Wheels.value = prepareCallNumbers(data['entityList'])
    else if (data['category'] === 'pickup')
      queueListPickup.value = prepareCallNumbers(data['entityList'])

    let lastQueue = prepareOneCallNumber(data.lastQueue)

    // Call the last queue
    console.trace("Speak: พาเชินหมายเลข " + lastQueue.call_number + " ตรงโกดังที่ " + lastQueue.warehouse)
    let utterance = new SpeechSynthesisUtterance(`พาเชินหมายเลข ${lastQueue.call_number} ตรงโกดังที่ ${lastQueue.warehouse}`);
    utterance.voice = thaiVoice;
    speechSynthesis.speak(utterance);
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
  <v-container class="">
    <v-responsive>
      <!-- 3 columns -->
      <v-row class="queue-list">
        <v-col cols="4">
          <h2>รถ 10 ล้อ</h2>
          <v-card class="ma-2" v-for="queue in queueList10Wheels" :key="queue.id">
            <v-card-title>
              <h3>
                {{ queue.call_number }}
              </h3>
            </v-card-title>
            <v-card-text>
              <p>Warehouse {{ queue.warehouse }}</p>
            </v-card-text>
          </v-card>
        </v-col>
        <v-col cols="4">
          <h2>รถ 6 ล้อ</h2>
          <v-card class="ma-2" v-for="queue in queueList6Wheels" :key="queue.id">
            <v-card-title>
              <h3>
                {{ queue.call_number }}
              </h3>
            </v-card-title>
            <v-card-text>
              <p>Warehouse {{ queue.warehouse }}</p>
            </v-card-text>
          </v-card>

        </v-col>
        <v-col cols="4">
          <h2>รถกระบะ</h2>
          <v-card class="ma-2" v-for="queue in queueListPickup" :key="queue.id">
            <v-card-title>
              <h3>
                {{ queue.call_number }}
              </h3>
            </v-card-title>
            <v-card-text>
              <p>Warehouse {{ queue.warehouse }}</p>
            </v-card-text>
          </v-card>

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