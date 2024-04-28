<template>
  <v-app>
    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script lang="ts" setup>

import {onMounted} from "vue";

onMounted(()=>{
  // show environment variable BACKEND_SERVER
  console.log("Server : " +import.meta.env.VITE_BACKEND_SERVER)
})
</script>
