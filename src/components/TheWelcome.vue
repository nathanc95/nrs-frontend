<template>
  <div>
    <h1>Data from API:</h1>
    <ul>
      <li v-for="item in apiData" :key="item.id">{{ item.title }}</li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      apiData: [],
    };
  },
  mounted() {
    axios.get('http://localhost:6000/state')
        .then(response => {
          // Update the data property with the received API data
          this.apiData = response.data;
        })
        .catch(error => {
          console.error('Error fetching data from the API:', error);
        });
  },
};
</script>
