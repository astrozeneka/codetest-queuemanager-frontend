<script setup lang="ts">


let warehouseLabelList = ['10-wheels (1)', '10-wheels (2)', '6-wheels (1)', '6-wheels (2)', '6-wheels (3)', '6-wheels (4)', 'Pickup (1)', 'Pickup (2)']
let warehouseDict = {
  '10-wheels (1)': {category: '10-wheels', warehouse: '1'},
  '10-wheels (2)': {category: '10-wheels', warehouse: '2'},
  '6-wheels (1)': {category: '6-wheels', warehouse: '3'},
  '6-wheels (2)': {category: '6-wheels', warehouse: '4'},
  '6-wheels (3)': {category: '6-wheels', warehouse: '5'},
  '6-wheels (4)': {category: '6-wheels', warehouse: '6'},
  'Pickup (1)': {category: 'pickup', warehouse: '7'},
  'Pickup (2)': {category: 'pickup', warehouse: '8'},
}
let activeWarehouse = ref('')
// Listen change event

const refreshData = () => {
  console.log("Update selected warehouse")
  category.value = warehouseDict[activeWarehouse.value].category
  warehouse.value = warehouseDict[activeWarehouse.value].warehouse
  loadData()
  initSocket()
  // Reconnect the websocket

}

import {onMounted, ref} from "vue";

let actuallyServing = ref(false)

let category = ref('10-wheels')
let warehouse = ref('5')
let entityList = ref([])

const prepareCallNumbers = (queueList)=>{
  for (let entity of queueList) {
    let call_initial = {
      "10-wheels": "A",
      "6-wheels": "B",
      "pickup": "C"
    }[category.value]
    entity.call_number = call_initial + entity.call_id.toString().padStart(3, '0')
  }
  return queueList
}

const prepareOneCallNumber = (queue)=>{
  let call_initial = {
    "10-wheels": "A",
    "6-wheels": "B",
    "pickup": "C"
  }[category.value]
  queue.call_number = call_initial + queue.call_id.toString().padStart(3, '0')
  return queue
}

const loadData = () => {

  // @app.get('/category/{category}/queue')
  console.log(category.value)
  fetch('http://localhost:8000/category/' + category.value + '/queue')
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
  fetch('http://localhost:8000/warehouse/' + warehouse.value + '/queue/' + entityList.value[0].id)
    .then(response => response.json())
    .then(data => {
      console.log('Success:', data);
      actuallyServing.value = prepareOneCallNumber(data)
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}

const done = () => {
  // @app.post('/queue/{id}/finish')
  fetch('http://localhost:8000/queue/' + entityList.value[0].id + '/finish', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({})
  })
    .then(response => response.json())
    .then(data => {
      console.log('Success:', data);
      actuallyServing.value = false
      loadData()
    })
    .catch((error) => {
      console.error('Error:', error);
    });

}

const initSocket = () => {
  const ws = new WebSocket('ws://localhost:8000/ws/' + category.value);
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
}

onMounted(() => {
  loadData()
  initSocket()
})
</script>

<template>
  <v-container>
    <h3>Category: {{ category }}</h3>
    <v-row>
      <v-col cols="8">

        <div class="controls pa-4">
          <v-select
            v-model="activeWarehouse"
            :items="warehouseLabelList"
            label="Select warehouse"
          ></v-select>


          <v-btn color="primary" @click="refreshData">
            <v-icon left>
              mdi-check
            </v-icon>
            Confirm
          </v-btn>
        </div>
        <div class="controls pa-4">
          <v-btn color="primary" @click="callNext">
            <v-icon left>
              mdi-arrow-right
            </v-icon>
            Call next
          </v-btn>
          <v-btn color="success" @click="done">
            <v-icon left>
              mdi-check
            </v-icon>
            Done
          </v-btn>
        </div>

        <v-card class="current-serving ma-4" v-if="actuallyServing">
          <v-card-title>
            <h2>Actually serving</h2>
          </v-card-title>
          <v-card-text>
            <h3>{{ actuallyServing.call_number }}</h3>
            <p>Warehouse {{ actuallyServing.warehouse }}</p>
            <p>{{ actuallyServing.category }}</p>
          </v-card-text>
        </v-card>

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
  display: flex;
  flex-direction: row;
  gap: 1em;
  justify-content: center;
  align-content: center;
}

.next-services{
  & h3, & p{
    text-align: center;
  }
}
</style>