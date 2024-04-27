<script setup lang="ts">


import {onMounted, ref} from "vue";

let actuallyServing = ref(false)

let category = '10-wheels'
let warehouse = '5'
let entityList = ref([])

const prepareCallNumbers = (queueList)=>{
  for (let entity of queueList) {
    let call_initial = {
      "10-wheels": "A",
      "6-wheels": "B",
      "pickup": "C"
    }[category]
    entity.call_number = call_initial + entity.call_id.toString().padStart(3, '0')
  }
  return queueList
}
const loadData = () => {

  // @app.get('/category/{category}/queue')

  fetch('http://localhost:8000/category/' + category + '/queue')
    .then(response => response.json())
    .then(data => {
      console.trace('Success:', data);
      entityList.value = prepareCallNumbers(data)
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}

const callNext = () => {
  // @app.get('/warehouse/{warehouse}/queue/{id}')
  fetch('http://localhost:8000/warehouse/' + warehouse + '/queue/' + entityList.value[0].id)
    .then(response => response.json())
    .then(data => {
      console.log('Success:', data);
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}

onMounted(() => {
  loadData()
  // Listen to websocket http://localhost:8000/ws
  const ws = new WebSocket('ws://localhost:8000/ws/' + category);
  ws.onopen = () => {
    console.log('connected');
  }
  ws.onmessage = (event) => {
    entityList.value = prepareCallNumbers(JSON.parse(event.data))
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
  <v-container>
    <v-row>
      <v-col cols="8">
        <v-card class="current-serving" v-if="actuallyServing">
          <v-card-title>
            <h2>Actually serving</h2>
          </v-card-title>
          <v-card-text>
            <h3>A001</h3>
            <p>Warehouse 5</p>
            <p>Pickup</p>
          </v-card-text>
        </v-card>
        <div class="controls pa-4">

          <v-btn color="primary" @click="callNext">
            <v-icon left>
              mdi-arrow-right
            </v-icon>
            Call next
          </v-btn>
        </div>

      </v-col>
      <v-col cols="4" class="next-services">
        <h2>Next services</h2>
        <v-card class="ma-2" v-for="entity in entityList" :key="entity.id">
          <v-card-title>
            <h3>{{ entity.call_number }}</h3>
          </v-card-title>
          <v-card-text>
            <p>{{ entity.category }}</p>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
.current-serving{
  text-align: center;
  & h2{
    font-size: 1.2rem;
  }
  & h3{
    font-size: 5rem;
  }
}

.controls{
  text-align: center;
}

.next-services{
  & h3, & p{
    text-align: center;
  }
}
</style>