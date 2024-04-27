<script setup lang="ts">

// One-binding variables to be printed on the virtual paper
import {ref} from "vue";

let call_number = ref('')
let entity = ref({})
let qr_url = ref('')

const select = (category: string) => {
  const data = {
    category: category
  }
  entity.value = {}
  call_number.value = ''
  qr_url.value = ''

  fetch('http://localhost:8000/request-queue', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(data)
  })
    .then(response => response.json())
    .then(data => {
      console.log('Success:', data);
      // Create dictionnary in js
      let call_initial = {
        "10-wheels": "A",
        "6-wheels": "B",
        "pickup": "C"
      }[category]
      call_number.value = call_initial + data.call_id
      entity.value = data

      // Generate qr using api.qrserver.com
      qr_url.value = `https://api.qrserver.com/v1/create-qr-code/?size=100x100&data=${call_number.value}`
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}
</script>

<template>
  <v-container class="fill-height">
    <v-responsive>

      <h1>Select type of vehicle</h1>
      <br/><br/>
      <div class="button-list">
        <v-btn
          color="primary"
          @click="select('10-wheels')">
          10-wheels / รถ 10 ล้อ
        </v-btn>
        <v-btn
          color="primary"
          @click="select('6-wheels')">
          6-wheels / รถ 6 ล้อ
        </v-btn>
        <v-btn
          color="primary"
          @click="select('pickup')">
          Pickup / กระบะ
        ></v-btn>
      </div>

      <br/><br/><br/>
      <div>
        <v-card
          color="white"
          class="ma-3"
          v-if="call_number"
        >
          <v-card-text>
            <div class="paper align-center">
              <!-- logo -->
              <v-img
                class="mb-4"
                height="80"
                src="@/assets/logo.png"
              />

              <h3>Please keep electronic queue</h3>
              <div class="call_number">
                {{ call_number }}
              </div>
              <div class="info">
                Doors 5/6/7 (Pickup)<br/>
                XX persons waiting (xxx minutes)
              </div>
              <div class="qr">
                <img :src="qr_url">
              </div>
            </div>
          </v-card-text>
        </v-card>
      </div>
    </v-responsive>
  </v-container>
</template>

<style scoped>
.button-list{
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.paper{
  text-align: center;

  & .call_number{
    font-size: 3rem;
    font-weight: bold;
  }

  & .info{
    font-size: 1.2em;
  }

  & .qr img{
    margin-top: 2em;
    width: 100px;
    height: 100px;
  }

}
</style>