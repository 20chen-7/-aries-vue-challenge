<template>
  <div>
    <h1>
      Options Profit Calculator</h1>
    <div class="row">
      <div class="col-mb-12 col-lg-12 strike-list">
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
              <th>Select</th>
              <th>#Contract</th>
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
              <td>
                <input type="number" v-model="option.number" :id="option.strike_price"/>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col-mb-6 col-lg-6 strike-visualization">
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
          <div class="slider-container">
            <input
              type="range"
              v-model="stepVal"
              min="0.1"
              max="10"
              @input="updateStepVal($event.target.value)"
            />
            <span>Step Change: {{stepVal}}</span>
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
      <div v-if="loaded" class="summary">
        <div class="card-container mb-4">
          <div class="card-header">
            <slot name="header">Max Profit:</slot>
          </div>
          <div class="card-body">
            <div v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{ summary.maxProfit }}</div>
          </div>
          <div class="card-footer">
            <slot name="footer"></slot>
          </div>
        </div>
        <div class="card-container mb-4">
          <div class="card-header">
            
            <slot name="header">Max Loss</slot>
          </div>
          <div class="card-body">
            <div v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{ summary.maxLoss }}</div>
          </div>
          <div class="card-footer">
            <slot name="footer"></slot>
          </div>
        </div>
        <div class="card-container mb-4">
          <div class="card-header">
            <slot name="header">Break Even Points</slot>
          </div>
          <div class="card-body">
            <div v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{summary.breakEvenPoint}}</div>
          </div>
          <div class="card-footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
     </div> 
    
    </div>
</template>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
import { Line } from 'vue-chartjs'
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale)
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
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
      this.optionsData.forEach((option,index)=>{
        option.number = 1;
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
      minPriceVal:  75,
      maxPriceVal: 125,
      stepVal: 0.5,
      summaryArray: [],
      breakEvenPoints: [],
  }}, 
  
  methods: {
    fetchedBreakEvenData(){
      const breakEvenPoints = [];
      if(this.optionsData.length){
        for(let index = 0; index < this.optionsData.length; index++){
          let option = this.optionsData[index];
          let point;
          let premium = (option.ask + option.bid)/2;
          if(option.long_short === 'long'){
            point = option.type === 'call' ? option.strike_price + premium : option.strike_price - premium;
          }else{
            point = option.type === 'call' ? option.strike_price + premium : option.strike_price - premium;
          }
          breakEvenPoints.push(point);
        }
      }
      this.breakEvenPoints = breakEvenPoints;
    },
    updateMinVal(val){
      this.minPriceVal = Math.max(this.min, val);
    },
    updateMaxVal(val){
      this.maxPriceVal = Math.min(this.max, val);
    },
    updateStepVal(val){
      this.stepVal = val;
    },

    toggleCheckbox(index){
      this.optionsData[index].checked = !this.optionsData[index].checked;
    },
    generateChartData() {
      const profits = []
      const colors = ['#f87979', "yellow", "red", "green"]
      
      this.optionsData.forEach((option,index)=>{
        profits.push({id: index, data:[],color:colors[index], breakEvenPoint:this.breakEvenPoints[index]})
      });
      const totalProfits = [];
      const prices = [];
      const totalBreakEvenPoint = [];
      this.optionsData.filter(option=>option.checked).forEach((option,index)=>{
        totalBreakEvenPoint.push(this.breakEvenPoints[index]);
      })

      for (let price = this.minPriceVal; price <= this.maxPriceVal; price+=this.stepVal){
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
          if(option.number !== 1){
            optionProfit *= option.number;
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
        breakEvenPoint: totalBreakEvenPoint,
        fill: false,
        tension: 0.2,
        pointRadius: 2,

      }
      const individalDataset = profits.filter((profit)=> profit.data.length > 0).map(item => ({
        label: `No. ${item.id+1}`,
        backgroundColor: item.color,
        data: item.data,
        breakEvenPoint: item.breakEvenPoint,
        fill: false,
        tension: 0.2,
        pointRadius: 2,
      }));

      this.chartData = {
        labels: prices,
        datasets: [...individalDataset, totalDataset],
      };
      
    },
    customAlert(options){
      return new Promise((resolve) => {
          alert(`${options.title}\n\n${options.message}`);
          resolve();
        });
    },
    summary(){
      if (!this.chartData.datasets.length) {
        return [];
      }
      this.summaryArray = this.chartData.datasets.map((dataset, index) => {
        if (Array.isArray(dataset.data) && dataset.data.length) {
          return {
            label:dataset.label,
            maxProfit: Math.max(...dataset.data),
            maxLoss: Math.min(...dataset.data),
            breakEvenPoint: dataset.breakEvenPoint,
          };
        }
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
        this.fetchedBreakEvenData();
        this.generateChartData();
        this.summary();
      }
    },
    
  },
}
</script>

<style scoped>
div{
  text-align: center;
}
h1 {
  color: #19222b;
  font-size: 2rem;
  font-family: fantasy;
  font-weight: bolder;
}
.row {
  display: flex;
  flex-wrap: wrap;
  margin-right: -15px;
  margin-left: -15px;
}

.col-lg-12, .col-md-12 {
  position: relative;
  width: 100%;
  padding-right: 15px;
  padding-left: 15px;
}
@media (min-width: 768px) { /* Adjusts for medium screens and up */
  .col-md-6 {
    flex: 0 0 50%;
    max-width: 50%;
  }
}

@media (min-width: 992px) { /* Adjusts for large screens */
  .col-lg-6 {
    flex: 0 0 50%;
    max-width: 50%;
  }
}
.strike-list{
  align-items: center;
}
table{
  font-family: sans-serif;;
  font-size: 1rem;
  border-collapse: collapse;
  width: 80%;
  color: white;
  margin: 10% auto;
  background: -webkit-linear-gradient(left, rgb(127, 26, 26) 0%, rgb(137, 18, 18) 20%, rgb(137, 18, 18) 60%, rgb(43, 22, 136) 100%);
}

thead tr, thead th { background:transparent; }
thead{
  border-bottom: 2px solid #ddd;
}

tr:hover {
  background: -webkit-linear-gradient(left, rgb(144, 178, 124) 0%, rgb(144, 178, 124) 20%, rgb(144, 178, 124) 80%, rgb(183, 143, 63) 100%);
}
input[type=number]{
  font-size: larger;
  text-align: center;
  color: white;
  background-color: transparent;
  border: none;
  padding: 0;
  margin: 0;
  width: 100%
}

input[type=number]:focus {
  background-color: lightblue;
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
  justify-content: center;
}
.slider-container {
  margin: 30px;
}
span {
  display: inline-block;
  text-align: left;
  width: 10%;
  margin-left: 5%;
}

.table-header{
  font: 1em sans-serif;
}
.row{
  text-align: center;
}
.summary{
  display: flex; 
  justify-content: center;
  gap: 20px;
  padding: 20px;
  flex-direction: row;
}
.card-container {
  border: 1px solid #e0e0e0;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  background-color: white;
  width: 40ch;
}
.card-header{
  color: white;
  font-weight: bold;
  background-color: rgb(67, 53, 131);
};

.card-footer {
  padding: 16px;
  background-color: #f5f5f5;
}

.card-body {
  padding: 16px;
}
</style>
