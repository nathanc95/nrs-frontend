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
              @click="displayStateDetails(item)">
            {{ item.state }}
          </li>
        </ul>
      </section>
      <section class="flex" v-if="displaySecondSelection">
        <div>
          <input type="text" v-model="filterText" placeholder="filter by name">
        </div>
        <p>Duplicate List</p>
        <ul>
          <li v-for="(item, index) in filteredItems" :key="index"
              @dblclick="highlightItem(index)"
              @click="displayStateDetails(item)"
              :class="{ 'highlighted': index === selectedIndex }">
            {{ item.state }}
          </li>
        </ul>
      </section>

      <section class="flex" v-if="displaySecondSelection">
        <p>State Counties Details</p>
        <p>{{ state }}</p>
        <p>State Population</p>
        <p>{{ statePopulation }}</p>
        <p>Number of counties</p>
        <p>{{ countiesLength }}</p>
        <ul v-if="displaySecondSelection">
          <li v-for="(item, index) in countiesDetails" :key="index">
            {{ item.county }} population: {{item.population}}
          </li>
        </ul>
        <p>county population</p>
        <p>{{ countiePopulation }}</p>
        <p>record</p>
        <p>state population: {{statePopulation}} countie populaiton: {{ countiePopulation }}</p>
        <p>{{record}}</p>
      </section>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      filterText: '',
      apiData: [],
      duplicateList: [],
      displaySecondSelection: false,
      selectedIndex: null,
      state: null,
      statePopulation: 0,
      countiesLength: 0,
      countiesDetails:[],
      countiePopulation: 0,
      record:false
    };
  },
  methods: {
    async getStates() {
      const stateApiRes = await axios.get('http://localhost:8080/state');
      this.apiData = stateApiRes.data ?? [];
    },
    async displayStateDetails(item) {
      this.state = item.state;
      const stateId = item.id;
      this.duplicateList = this.apiData;
      this.displaySecondSelection = true;
      const stateApiRes = await axios.get(`http://localhost:8080/county/${stateId}`);
      const stateApiResData = stateApiRes.data;
      console.log({ stateApiResData });
      this.statePopulation = parseInt(stateApiResData.sumDetails[0].statepopulation, 10);
      this.countiesLength = stateApiResData.countiesDetails.length;
      this.countiesDetails = stateApiResData.countiesDetails;
      this.countiePopulation = parseInt(stateApiResData.sumDetails[0].sumcountypopulation, 10);
      if (this.countiePopulation === this.statePopulation) {
        this.record = true;
      }
    },
    highlightItem(index) {
      this.selectedIndex = index;
    },
  },
  beforeMount() {
    this.getStates()
  },
  computed: {
    filteredItems() {
      const filterText = this.filterText.toLowerCase();
      return this.duplicateList.filter(item => item.state.toLowerCase().includes(filterText));
    },
  },
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
