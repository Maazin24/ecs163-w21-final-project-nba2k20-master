<!-- This file will be used to layout and connect your views -->
<template>
  <div id="app">
    <!-- This is only a basic setup to display your views. Use HTML and CSS to create your layout using these basic commands -->
    <ScatterPlot
      class="scatter-plot"
      v-if="dataset"
      chartId="poke1"
      title="Teams Overview"
      :dataset="dataset"
      attribX="rating"
      labelX="total rating of each team"
      attribY="salaries"
      labelY="total spending of each team"
      attribColor="name"
      @item-selected="teamSelected"
      :width="800"
      :height="500"
    />
    <BarChart 
      class="bar-chart" 
      v-if="datasetSelected" 
      chartId="poke2" 
      :title="`Details about ${datasetSelected.name}`" 
      :dataset="datasetSelected.entries" 
      attribX="name" 
      attribY="ratings"  
      labelY="ratings"
      :width="800"
      :height="500" />
    <h1 v-else class="bar-chart-info">Select a team</h1>
  </div>
</template>

<script>
import * as d3 from "d3";

// Try to use descriptive names like 'OverviewScatterPlot' or 'DetailLineChart'
import ScatterPlot from "./components/ScatterPlot.vue";
import BarChart from "./components/BarChart.vue";

export default {
  name: "App",
  components: {
    ScatterPlot,
    BarChart,
  },
  data() {
    return {
      data: null,
      dataset: null,
      datasetSelected: null,
    };
  },
  created() {
      d3.csv("/data/nba2k20-full.csv", d3.autoType).then((data) => {      
        let teams = []
        for(var i = 0; i < data.length; i++){
          teams[i] = data[i].team;
          }
        let unique = [];
          teams.forEach((t) => {
            if (!unique.includes(t)) {
              unique.push(t);
            }
          })
          teams = unique

        let j = 0
        let money = new Array(teams.length).fill(0)
        let salary = []
        while(j < teams.length) {
          for(var k = 0; k < data.length; k++){
            if(data[k].team == teams[j]){
              money[j] = parseInt(data[k].salary.replace('$', '')) + money[j]
            }
          }
              j = j + 1;
        }
        salary = money
        var i1 = 0
        let ratings = new Array(teams.length).fill(0)
        while(i1 < teams.length) {
          for(var k1 = 0; k1 < data.length; k1++){
            if(data[k1].team == teams[i1]){
              ratings[i1] = Number(data[k1].rating) + ratings[i1]
            }
          }
          i1 = i1 + 1;
        }

        let table = new Array(teams.length).fill(0)
        for(var i2 = 0; i2 < teams.length; i2++){
          table[i2] = { 
            name: teams[i2],
            salary : salary[i2],
            ratings: ratings[i2]
          }
        }
        this.data = data  
       this.dataset = d3.map(table,  (d) =>{
         return Object.assign(d, {
           rating: d.ratings, 
           salaries: d.salary
         })
       }) 
     
    });

  },
  methods: {

    teamSelected(p){
      if(p == null) {
        this.datasetSelected = null
        return
      }
      let team = []

      var thisteam = p.name
      if(p.name == null){
        thisteam = "Free Agents"
      }
      team = d3.filter(this.data, d => d.team == p.name)

      console.log(thisteam)

      let entries = []
      for (var i = 0; i < team.length; i++){
        entries[i] = { 
          name: team[i].full_name,
          ratings: team[i].rating
        }
      }
      
      this.datasetSelected = {
        name: thisteam,
        entries: entries
      }
    }
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: hsl(210, 29%, 24%);
  display: grid;
  grid-template-rows: 5vh 60vh auto;
  grid-template-columns: 1fr 10fr 10fr 1fr;
  grid-template-areas: 
    ". . . ." 
    ". overview detail ."
    ". . . .";
}

.scatter-plot {
  grid-area: overview;
}

.bar-chart {
  grid-area: detail;
}

.bar-chart-info {
  grid-area: detail;
  color: rgb(214, 214, 214);
  margin: auto;
}
</style>
