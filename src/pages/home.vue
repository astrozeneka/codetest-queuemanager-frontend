<script setup lang="ts">
const exportStats = () =>{
  // @app.get('/export/pdf')
  fetch('http://localhost:8000/export/pdf', {
    method: 'GET',
  })
    .then(response => response.blob())
    .then(blob => {
      const url = window.URL.createObjectURL(new Blob([blob]));
      const link = document.createElement('a');
      link.href = url;
      link.setAttribute('download', 'stats.pdf');
      document.body.appendChild(link);
      link.click();
    })
    .catch((error) => {
      console.error('Error:', error);
    });
}
</script>

<template>
  <v-container class="fill-height">
    <v-responsive>
      <h1>Home</h1>
      <v-btn
        color="primary"
        @click="exportStats"
      >
        <v-icon left>
          mdi-file-pdf
        </v-icon>
        Export Stats as PDF
      </v-btn>
    </v-responsive>
  </v-container>
</template>

<style scoped>

</style>