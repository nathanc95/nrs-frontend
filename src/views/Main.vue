<template>
  <div id="nrsApp">
      <div class="list" v-if="apiData">
        <div class="originalList"></div>
        <p class="bold">Original State List</p>
        <ul>
          <List v-for="(item, index) in apiData" :key="index"
                @click="displayStateDetails(item)"
                @dblclick="highlightItem(index)"
                :isHighlighted="(index === selectedIndex) && displayHighlighted"
                :item="item"></List>
        </ul>
      </div>
      <div class="list" v-if="duplicateList">
        <search-duplicate v-model="filterText"></search-duplicate>
        <p class="bold">Duplicate State List</p>
        <ul>
          <List v-for="(item, index) in filteredItems" :key="index"
                @click="displayStateDetails(item)"
                @dblclick="highlightItem(index)"
                :isHighlighted="(index === selectedIndex) && displayHighlighted"
                :item="item"></List>
        </ul>
      </div>

      <div class="list" v-if="displaySecondSelection">
        <div class="stateInfo">
        <ul>
          <li><span>State Name </span><span class="bold"> {{ state }}</span></li>
          <li><span>State Population </span><span class="bold"> {{ statePopulation }}</span></li>
          <li><span>Sum of county population </span> <span class="bold"> {{ countiePopulation }}</span></li>
          <li v-if="countiePopulation === statePopulation"><span>Countie population match the state population</span></li>
          <li v-if="countiePopulation !== statePopulation"><span>Countie population doesn't match the State population</span></li>
          <li><span>State Number of counties </span><span class="bold"> {{ countiesLength }}</span></li>
        </ul>
        </div>
        <div class="stateInfo countyStateInfo">
          <ul v-if="displaySecondSelection">
            <li v-for="(item, index) in countiesDetails" :key="index">
              <span>{{ item.county }} population:</span><span class="bold">{{ item.population }}</span>
            </li>
          </ul>
        </div>
      </div>
      <div v-if="apiErrorResponse">
        <span>{{apiErrorResponse}}</span>
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
      displayHighlighted: false,
      apiErrorResponse:null,
      filterText: '',
      apiData: null,
      duplicateList: null,
      displaySecondSelection: null,
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
      try {
        const stateApiRes = await axios.get(this.NRS_BACKEND + '/state');
        this.apiData = stateApiRes.data ?? [];
        this.duplicateList = this.apiData;
      } catch (err) {
        this.apiErrorResponse = 'unable to retrieve state data';
      }
    },
    async displayStateDetails(item) {
      try {
        this.state = item.state;
        const stateId = item.id;
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
        this.displaySecondSelection = true;
      } catch (err) {
        this.apiErrorResponse = 'unable to retrieve state details';
      }
    },
    highlightItem(index) {
      if (index === this.selectedIndex && this.displayHighlighted) {
        this.displayHighlighted = false;
        return;
      }
      this.selectedIndex = index;
      this.displayHighlighted = true;
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
.countyStateInfo {
  border: 1px solid #00000012;
  margin-top: 1em;
  border-radius: 15px;
  padding: 0.3em;
}

input {
  border-radius: 5px;
  border: 1px solid #8080808f;
  padding: 0.5em;
}
.originalList {
  height: 2em;
}
.stateInfo li {
  display: flex;
  justify-content: space-between;
}
.bold {
 font-weight: bold;
}

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
  background-color: #ffff;
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

li:nth-child(even) {
  background-color: #f0f0f0; /* Grey background for even-numbered items */
}

li:nth-child(odd) {
  background-color: #ffffff; /* White background for odd-numbered items */
}
</style>
