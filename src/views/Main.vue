<template>
  <div id="nrsApp">
      <div class="list">
        <p>Original List</p>
        <ul>
          <List v-for="(item, index) in apiData" :key="index"
                @click="displayStateDetails(item)"
                @dblclick="highlightItem(index)"
                :isHighlighted="index === selectedIndex"
                :item="item"></List>
        </ul>
      </div>
      <div class="list" v-if="displaySecondSelection">
        <search-duplicate v-model="filterText"></search-duplicate>
        <p>Duplicate List</p>
        <ul>
          <List v-for="(item, index) in filteredItems" :key="index"
                @click="displayStateDetails(item)"
                @dblclick="highlightItem(index)"
                :isHighlighted="index === selectedIndex"
                :item="item"></List>
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
import SearchDuplicate from "@/components/SearchDuplicate.vue";
import List from "@/components/List.vue";

export default {
  components: {List, SearchDuplicate},
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
      record: false,
      NRS_BACKEND: import.meta.env.VITE_NRS_BACKEND_PATH
    };
  },
  methods: {
    async getStates() {
      console.log(import.meta.env)
      const stateApiRes = await axios.get(this.NRS_BACKEND + '/state');
      this.apiData = stateApiRes.data ?? [];
    },
    async displayStateDetails(item) {
      try {
        this.state = item.state;
        const stateId = item.id;
        this.duplicateList = this.apiData;
        this.displaySecondSelection = true;
        const stateApiRes = await axios.get(this.NRS_BACKEND + `/county/${stateId}`);
        const stateApiResData = stateApiRes.data;
        const statePopulation = stateApiResData?.sumDetails?.[0]?.statepopulation;
        const countiePopulation = stateApiResData?.sumDetails?.[0]?.sumcountypopulation;
        if (statePopulation !== undefined) {
          this.statePopulation = parseInt(stateApiResData?.sumDetails?.[0]?.statepopulation, 10);
        }

        if (countiePopulation !== undefined) {
          this.countiePopulation = parseInt(stateApiResData.sumDetails[0].sumcountypopulation, 10);
        }

        this.countiesLength = stateApiResData.countiesDetails.length;
        this.countiesDetails = stateApiResData.countiesDetails;
        if (this.countiePopulation === this.statePopulation) {
          this.record = true;
        }
      } catch (err) {}
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
