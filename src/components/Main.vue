<template>
  <div>
    <h1>Data from API:</h1>
    <div>
      <section class="flex">
        <p>Original List</p>
        <ul>
          <li v-for="(item, index) in apiData" :key="index"
              @dblclick="highlightItem(index)"
              :class="{ 'highlighted': index === selectedIndex }"
              @click="displayStateDetails(index)">
            {{ item.state }}
          </li>
        </ul>
      </section>
      <section class="flex">
        <p>Duplicate List</p>
        <ul v-if="displaySecondSelection">
          <li v-for="(item, index) in duplicateList" :key="index"
              :class="{ 'highlighted': index === selectedIndex }">
            {{ item.state }}
          </li>
        </ul>
      </section>
      <section class="flex">
        <p>State Counties Details</p>
        <p>{{ state }}</p>
        <p>State Population</p>
        <p>{{ statePopulation }}</p>
        <p>Number of counties</p>
        <p>{{ countiesLength }}</p>
        <ul v-if="displaySecondSelection">
          <li v-for="(item, index) in countiesDetails" :key="index">
            {{ item.county }}
          </li>
        </ul>
      </section>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      apiData: [],
      duplicateList: [],
      displaySecondSelection: false,
      selectedIndex: null,
      state: null,
      statePopulation: 0,
      countiesLength: 0,
      countiesDetails:[]
    };
  },
  methods: {
    async getStates() {
      const stateApiRes = await axios.get('http://localhost:8080/state');
      this.apiData = stateApiRes.data ?? [];
    },
    async displayStateDetails(index) {
      const stateName = this.apiData[index];
      this.state = stateName.state;
      const stateId = stateName.id;
      this.duplicateList = this.apiData;
      this.displaySecondSelection = true;
      const stateApiRes = await axios.get(`http://localhost:8080/county/${stateId}`);
      const stateApiResData = stateApiRes.data;
      console.log({ stateApiResData });
      this.statePopulation = parseInt(stateApiResData.sumDetails[0].statepopulation, 10);
      this.countiesLength = stateApiResData.countiesDetails.length;
      this.countiesDetails = stateApiResData.countiesDetails;
    },
    highlightItem(index) {
      this.selectedIndex = index;
    },
  },
  beforeMount() {
    this.getStates()
  }
};
</script>
<style>
.flex {
  border: 1px solid black;
  margin: 1em;
  display: inline-flex;
  justify-content: center;
  flex-direction: row;
}
/* Add styles for the highlighted option */
ul li.highlighted {
  background-color: yellow;
  font-weight: bold;
}
</style>
