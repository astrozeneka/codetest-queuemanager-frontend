<script setup lang="ts">
const exportStats = () =>{
  console.log("Export PDF")
  // @app.get('/export/pdf')
  fetch(import.meta.env.VITE_BACKEND_SERVER + '/export/pdf', {
    method: 'GET',
  })
      .then(response => response.blob())
      .then(blob => {
        console.log("Receive file")
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

      <v-img
          class="mb-4"
          height="150"
          src="@/assets/logo.png"
      />
      <div class="text-center">
        <h4 class="text-h4 font-weight-bold">Queue manager</h4>
        <br/><br/><br/>
      </div>


      <div class="menu">
        <v-btn
            to="/central"
            color="primary"
        >
          Central screen
        </v-btn>
        <v-btn
            to="/officer"
            color="primary"
        >
          Officer screen
        </v-btn>
        <v-btn
            to="/distributor"
            color="primary"
        >
          Queue Distributor
        </v-btn>
        <v-btn
            color="primary"
            @click="exportStats"
        >
          <v-icon left>
            mdi-file-pdf
          </v-icon>
          Export Stats as PDF
        </v-btn>
      </div>
    </v-responsive>
  </v-container>
</template>

<style scoped>
.menu{
  display: flex;
  flex-direction: column;
  gap: 1em;
}
</style>