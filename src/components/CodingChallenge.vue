<template>
  <div>
    <h1>Options Profit Calculator</h1>
    <div class="row">
      <div class="col mb-6 strike-list">
        <table>
          <thead>
            <tr class="table-header">
              <th >No. </th>
              <th>Strike Price</th>
              <th>Type</th>
              <th>Bid</th>
              <th>Ask</th>
              <th>Position</th>
              <th>Expiration Date</th>
              <th>Chose</th>
            </tr>
          </thead>
          <tbody class="table-body">
            <tr v-for="(option, index) in optionsData" :key="index" @click="toggleCheckbox(index)">
              <td>{{ index+1 }}</td>
              <td>{{ option.strike_price }}</td>
              <td>{{ option.type }}</td>
              <td>{{ option.bid }}</td>
              <td>{{ option.ask }}</td>
              <td>{{ option.long_short }}</td>
              <td>{{ new Date(option.expiration_date).toLocaleDateString() }}</td>
              <td>
                <input type="checkbox" v-model="option.checked" :id="option.strike_price" @click.stop />
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col mb-6 strike-visualization">
        <div class="sliders-container">
          <div class="slider-container">
            <input
              type="range"
              v-model="minPriceVal"
              min="0"
              max="100"
              @input="updateMinVal($event.target.value)"
            />
            <span>Min Price: {{minPriceVal}}</span>
          </div>
          <div class="slider-container">
            <input
              type="range"
              v-model="maxPriceVal"
              min="100"
              max="200"
              @input="updateMaxVal($event.target.value)"
            />
            <span>Max Price: {{maxPriceVal}}</span>
          </div> 
        </div>
        <div>
          <button @click="calculate">Generate</button>
        </div>
        <div class="chart-container">
          <Label v-if="loaded">Here is the Risk vs Reward Graph</Label>
          <line-chart v-if="loaded" :data="chartData" :options="options"></line-chart>
        </div>
      </div>
      <div class="profit-summary mb-12">
        <div class="card-header">
          <slot name="header">Max Profit:</slot>
        </div>
        <div class="card-body">
          <slot name="body">{{ maxProfit }}</slot>
        </div>
        <div class="card-footer">
          <slot name="footer"></slot>
        </div>
       </div>
       <div class="card-header">
          <slot name="header">Max Loss</slot>
        </div>
        <div class="card-body">
          <slot name="body">{{ maxLoss }}</slot>
        </div>
        <div class="card-footer">
          <slot name="footer"></slot>
        </div>
       </div>
       <div class="card-header">
          <slot name="header">Break Even Points</slot>
        </div>
        <div class="card-body">
          <slot name="body">{{ breakEvenPoints.join(', ') }}</slot>
        </div>
        <div class="card-footer">
          <slot name="footer"></slot>
        </div>
    </div>
</template>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
import { Line } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale)

export default {
  name: 'CodingChallenge',
  props: {
    optionsData: Array
  },
  components: {
    LineChart: Line
   },
  data(){
    const datasets = [];
    this.optionsData.forEach((option,index)=>{
      datasets.push({});
      });
    return {
      chartData: {
        labels: [],
        datasets: datasets,
      },
      options: {},
      loaded: false,
      min: 0,
      max: 250,
      minPriceVal: 75,
      maxPriceVal: 125,
  }}, 
  
  methods: {
    updateMinVal(val){
      this.minPriceVal = Math.max(this.min, val);
    },
    updateMaxVal(val){
      this.maxPriceVal = Math.min(this.max, val);
    },

    toggleCheckbox(index){
      this.optionsData[index].checked = !this.optionsData[index].checked;
    },
    generateChartData() {
      const profits = []
      const colors = ['#f87979', "yellow", "red", "green"]
      
      this.optionsData.forEach((option,index)=>{
        profits.push({id: index, data:[],color:colors[index]})
      });
      const totalProfits = []
      const prices = [];
      for (let price = this.minPriceVal; price <= this.maxPriceVal; price+=0.50){
        prices.push(price);
        let totalProfit = 0;
        for(let index = 0; index < this.optionsData.length; index++){
          let option = this.optionsData[index];
          if(!option.checked){
            continue;
          }
          let intrinsicVal = 0;
          let optionProfit = 0;
          if(option.type === 'call'){
            intrinsicVal = Math.max(0, price - option.strike_price);
          }else{
            intrinsicVal = Math.max(0, option.strike_price-price);
          }
          if(option.long_short === 'long'){
            optionProfit = intrinsicVal - option.ask;
          }else{
            optionProfit = option.bid - intrinsicVal;
          }
          totalProfit += optionProfit;
          if (!profits[index].data) {
            profits[index].data = [];
          }
          profits[index].data.push(optionProfit);
          
        };
        totalProfits.push(totalProfit);
      }
      const totalDataset = {
        label: 'Total Profit/Loss',
        backgroundColor: 'blue', 
        data: totalProfits,
        fill: false,
        tension: 0.2,
        pointRadius: 2,

      }
      const individalDataset = profits.filter((profit)=> profit.data.length > 0).map(item => ({
        label: `No. ${item.id+1}`,
        backgroundColor: item.color,
        data: item.data,
        fill: false,
        tension: 0.2,
        pointRadius: 2,
      }));

      this.chartData = {
        labels: prices,
        datasets: [...individalDataset, totalDataset],
      };
      
    },
    customAlert(options) {
      return new Promise((resolve) => {
          alert(`${options.title}\n\n${options.message}`);
          resolve();
        });
    },
    calculate(){
      if (!this.optionsData.filter(option => option.checked).length) {
        new Promise((resolve) => {
          alert("If you want to see the graph, please choose AT LEAST ONE STRIKE!");
          resolve();
        });
      }else{
        this.loaded = true;
        this.generateChartData();
      }
    }
  },
  computed: {
    maxProfit() {
      if (!this.chartData || !this.chartData.datasets || !this.chartData.datasets[0].data) {
      return 0; 
    }
      return Math.max(...this.chartData.datasets[0].data);
    },
    maxLoss() {
      if (!this.chartData || !this.chartData.datasets || !this.chartData.datasets[0].data) {
      return 0; 
    }
      return Math.min(...this.chartData.datasets[0].data);
    },
    breakEvenPoints() {
      if (!this.chartData || !this.chartData.datasets || !this.chartData.datasets[0].data) {
      return [] 
    }
      return this.chartData.labels.filter((_, index) => this.chartData.datasets[0].data[index] === 0);
    }
  },
}
</script>

<style scoped>
h1 {
  text-align: center;
  color: #19222b;
  font-family: fantasy;
  font-weight: bolder;
  background-color: beige;
  border: 2mm none rgba(213, 152, 38, 0.6);
  padding: 10px;
  box-shadow: 10px 10px 10px 5px rgb(174, 174, 159);
  border-radius: 20px;
  margin: 20px auto;
}
table{
  font-family: sans-serif;;
  margin: 0px auto;
  border-collapse: collapse;
  width: 80%;
  color: white;
  background: -webkit-linear-gradient(left, rgb(127, 26, 26) 0%, rgb(137, 18, 18) 20%, rgb(137, 18, 18) 60%, rgb(43, 22, 136) 100%);
}

thead tr, thead th { background:transparent; }
thead{
  border-bottom: 2px solid #ddd;
}

tr:hover {
  background: -webkit-linear-gradient(left, rgb(144, 178, 124) 0%, rgb(144, 178, 124) 20%, rgb(144, 178, 124) 80%, rgb(183, 143, 63) 100%);
}
.strike-visualization{
  align-items: center;
}
button{
  text-align: center;
  font-size: larger;
  font-family: sans-serif;
  border-radius: 10px;
  background-color: rgba(225, 180, 171, 0.601);
  padding: 10px;
  margin:10px;
  border: 1px solid rgb(122, 132, 97);
  box-shadow: 2px 2px 4px 1px rgb(174, 174, 159);
}
.chart-container {
  width:80%;
  display: flex; 
  justify-content: space-between;
  flex-direction: column;
  margin-left: 10%;
}
.sliders-container {
  display: flex;
  flex: 0 0 100vw;
  flex-direction: column;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  align-content: center;
}
.slider-container {
  margin: 10px;
}
.slider {
  display: flex;
  justify-content: flex-start;
}

input[type="range"] {
  margin-right: 10px;
  flex: 1;
}
span {
  width: 100px;
  text-align: center;
}

.table-header{
  font: 1em sans-serif;
}
.row{
  text-align: center;
}
</style>
