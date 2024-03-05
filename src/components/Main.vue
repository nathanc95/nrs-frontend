<template>
  <div id="nrsApp">
      <div class="list">
        <p>Original List</p>
        <ul>
          <li v-for="(item, index) in apiData" :key="index"
              @dblclick="highlightItem(index)"
              :class="{ 'highlighted': index === selectedIndex }"
              @click="displayStateDetails(item)">
            {{ item.state }}
          </li>
        </ul>
      </div>
      <div class="list" v-if="displaySecondSelection">
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
      </div>

      <div class="list" v-if="displaySecondSelection">
        <p>State Counties Details: {{ state }}</p>
        <p>State Population: {{ statePopulation }}</p>
        <p>Number of counties: {{ countiesLength }}</p>
        <ul v-if="displaySecondSelection">
          <li v-for="(item, index) in countiesDetails" :key="index">
            {{ item.county }} population: {{ item.population }}
          </li>
        </ul>
        <p>county population {{ countiePopulation }}</p>
        <p>state population: {{ statePopulation }} countie population: {{ countiePopulation }}</p>
        <p>record state population match sum of counties populations: {{ record }}</p>
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
      countiesDetails: [],
      countiePopulation: 0,
      record: false
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
#nrsApp {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}

/* Add styles for the highlighted option */
ul li.highlighted {
  background-color: yellow;
  font-weight: bold;
}

.list {
  border: 1px solid #ccc;
  padding: 10px;
  overflow-y: scroll;
  height: 25em;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

li {
  padding: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
}

li:hover {
  background-color: #f0f0f0;
}
</style>
