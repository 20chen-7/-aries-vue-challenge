<template>
  <div class="option-profit-calculator">
    <div class="title-box">
      <h1>Options Profit Calculator
      <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-info-circle intro" viewBox="0 0 16 16" @click="showInstruction"> <path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"/> <path d="m8.93 6.588-2.29.287-.082.38.45.083c.294.07.352.176.288.469l-.738 3.468c-.194.897.105 1.319.808 1.319.545 0 1.178-.252 1.465-.598l.088-.416c-.2.176-.492.246-.686.246-.275 0-.375-.193-.304-.533L8.93 6.588zM9 4.5a1 1 0 1 1-2 0 1 1 0 0 1 2 0z"/> </svg>
      </h1>
          <WarningDia v-bind:visible.sync="showInstructionDialog" title="Warning" type="warning">
            <template v-slot:header>
              <h3>Options Profit Calculator Instructions</h3>
            </template>
            <template>
                <p>1. Based on the details of four type of strike, you can check up tp four strike and choose the number of Contract for each option. </p>
                <p>2. The <strong>Min Price</strong> and <strong>Max Price</strong> are the lower boundary and upper boundary of current ASSET PRICE range, and <strong>Step Change</strong> is the increment 
                between the two range.</p>
                <p>3. After that, click the <strong>Generate</strong> button, you will get the chart based on your choices, and the <strong>Max Profit</strong>, <strong>Max Loss</strong> and <strong>Break Even Points</strong>
                        for each strike and total options profits will be shown in the bottom. </p>
            </template>
            <template v-slot:footer>
              <button @click="showInstructionDialog = false">Dismiss</button>
            </template>
          </WarningDia>
      <div class="strike-table-intro">
          <p class="strike-list-para">Here is the list of Strike and how to calculate the option profit.</p>      
        </div>
    </div>
    <div>
      <div class="col-mb-12 col-lg-12 strike-list">
        <table>
          <thead>
            <tr class="table-header">
              <th>No. </th>
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
                <div class="checkbox-container">
                  <label class="switch" :for="option.strike_price">
                    <input type="checkbox" :id="option.strike_price" v-model="option.checked" @click.stop/>
                    <div class="slider round"></div>
                  </label>
                </div>    
              </td>
              <td>
                <input type="number" v-model="option.number" :id="option.strike_price"/>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="col-mb-12 col-lg-12 strike-visualization">
        <div class="sliders-container">
          <div class="slider-container">
              <input
              type="range"
              v-model="minPriceVal"
              min="0"
              max="100"
              @input="updateMinVal($event.target.value)"
            />
            <span class="slider-span">Min Price: {{minPriceVal}}</span>
          </div>
          <div class="slider-container">
            <input
              type="range"
              v-model="maxPriceVal"
              min="100"
              max="200"
              @input="updateMaxVal($event.target.value)"
            />
            <span class="slider-span">Max Price: {{maxPriceVal}}</span>
          </div> 
          <div class="slider-container">
            <input
              type="range"
              v-model="stepVal"
              min="0.1"
              max="10"
              step="0.1"
              @input="updateStepVal($event.target.value)"
            />
            <span class="slider-span">Step Change: {{stepVal}}</span>
          </div> 
        </div>
        <div class="generate-btn-container">
          <div class="btn-container">
            <button class="btn" @click="calculate"><span class="btn-span">Generate</span></button>
          </div>
          <WarningDia v-bind:visible.sync="showWarningDialog" title="Warning" type="warning">
            <template v-slot:header>
              <h3>Warning</h3>
            </template>
            <template>
              <p>Please select At Leaset One Strike!</p>
            </template>
            <template v-slot:footer>
              <button @click="showWarningDialog = false">Dismiss</button>
            </template>
          </WarningDia>
        </div>
        <div class="chart-container">
          <label v-if="loaded">Here is the Risk vs. Reward Graph</label>
          <line-chart v-if="loaded" :data="chartData" :options="options"></line-chart>
        </div>
      </div>
      <div v-if="loaded" class="col-mb-12 col-lg-12 summary">
        <div class="card-container mb-4">
          <div class="card">
            <span></span>
            <div class="card-content">
              <h2>Max Profit</h2>
              <p v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{ summary.maxProfit }}</p>
            </div>
          </div>
          <div class="card">
            <span></span>
            <div class="card-content">
              <h2>Max Loss</h2>
              <p v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{ summary.maxLoss }}</p>
            </div>
          </div>
          <div class="card">
            <span></span>
            <div class="card-content">
              <h2>Break Even Points</h2>
              <p v-for="summary in summaryArray" :key="summary.label">{{ summary.label }}: {{summary.breakEvenPoint}}</p>
            </div>
          </div>
        </div>
      </div> 
    </div>
  </div>
</template>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script>
import { Line } from 'vue-chartjs';
import { Chart as ChartJS, Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale } from 'chart.js'
ChartJS.register(Title, Tooltip, Legend, LineElement, PointElement, LinearScale, CategoryScale)
import WarningDia  from './WarningDia.vue';   
export default {
  name: 'CodingChallenge',
  props: {
    optionsData: Array
  },
  components: {
    LineChart: Line,
    WarningDia,
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
      showWarningDialog: false,
      showInstructionDialog: false,
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
      const backgroundColors = [
        'rgba(255, 99, 132, 0.2)',
        'rgba(54, 162, 235, 0.2)',
        'rgba(255, 206, 86, 0.2)',
        'rgba(75, 192, 192, 0.2)',
        'rgba(153, 102, 255, 0.2)'
      ]
      const borderColors = [
        'rgba(255, 99, 132, 1)',
        'rgba(54, 162, 235, 1)',
        'rgba(255, 206, 86, 1)',
        'rgba(75, 192, 192, 1)',
        'rgba(153, 102, 255, 1)'
      ]
      
      this.optionsData.forEach((option,index)=>{
        profits.push({id: index, data:[],backgroundColor:backgroundColors[index],borderColor:borderColors[index], breakEvenPoint:this.breakEvenPoints[index]})
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
        backgroundColor: backgroundColors[backgroundColors.length-1], 
        borderColor: borderColors[borderColors.length-1],
        data: totalProfits,
        breakEvenPoint: totalBreakEvenPoint,
        fill: false,
        tension: 0.2,
        pointRadius: 2,

      }
      const individalDataset = profits.filter((profit)=> profit.data.length > 0).map(item => ({
        label: `No. ${item.id+1}`,
        backgroundColor: item.backgroundColor,
        borderColor: item.borderColor,
        data: item.data,
        breakEvenPoint: item.breakEvenPoint,
        fill: false,
        tension: 0.2,
        pointRadius: 2,
      }));

      this.chartData = {
        labels: prices,
        datasets: [...individalDataset, totalDataset],
      }
      this.options = {
        responsive: true,
        maintainAspectRatio: 1.5,
        scales: {
              x: {
                grid: {
                  display: true,
                  color: 'rgba(211, 211, 211, 1)',
                  lineWidth: 0.5,
                },
                ticks: {
                  color: 'rgba(211, 211, 211, 1)',
                  font: {
                    size: 14,
                    weight: 'bold'
                  },
                  padding: 10,
                  autoSkip: true,
                  maxRotation: 45,
                  minRotation: 0
                }
              },
              y: {
                grid: {
                  display: true,
                  color: 'rgba(211, 211, 211, 1)',
                  lineWidth: 0.5,
                },
                ticks: {
                  color: 'rgba(211, 211, 211, 1)',
                  font: {
                    size: 14,
                    weight: 'bold'
                  },
                  padding: 10,
                  autoSkip: true,
                  maxRotation: 45,
                  minRotation: 0
                }
              }
            }
         } 
      
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
      if (!this.optionsData.filter(option => option.checked).length){
        this.showWarningDialog = true;
      }else{
        this.loaded = true;
        this.fetchedBreakEvenData();
        this.generateChartData();
        this.summary();
      }
    },
    showInstruction(){
      this.showInstructionDialog = true;
    }
    
  },
}
</script>

<style scoped>
.option-profit-calculator{
  text-align: center;
  align-items: center;
  background-color: #000;
}
/** title */
.title-box{
  padding: 2vmin 10vmin;
  margin: auto 30vmin;
  background-color: #fff;
  border-radius: 4vmin;
  color:#66ddf7;
  font-size: 8vmin;
  position: relative;
  overflow: hidden;
  z-index: 10;
}
.title-box::before{
  content: '';
  width: 600%;
  height: 600%;
  background-color: #22292f;
  position: absolute;
  left: -250%;
  top: -250%;
  background-image: conic-gradient(transparent, #66ddf7, transparent 30%);
  z-index: -2;
  animation: rotate 3s linear infinite;
}
@keyframes rotate {
  to{
    transform: rotate(360deg);
  }
}
.title-box::after{
  content: '';
  position: absolute;
  inset: 0.5vmin;
  background-color: #000;
  border-radius: 3vmin;
  z-index: -1;
}
h1 {
  font-size: 4vmin;
}
p {
  font-size: 2vmin
}
.instruction-content{
  text-align: left;
}

.col-lg-12, .col-md-12 {
  position: relative;
  width: 100%;

}

.strike-list{
  align-items: center;
}

.strike-list-para{
  margin-bottom: auto;
}
table{
	font-family: "Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
	font-size: 2vmin;
	margin: 5vmin auto;
	width: 60vmin;
	border-collapse: collapse;
	text-align: center;
  background-color: #282828;
  color:#D3D3D3;
  box-shadow: 0 0 2px #66ddf7,
              0 0 5px #66ddf7;
}

th{
  font-size: 2vmin;
	font-weight: bold;
	color:  #34b7d4;
	padding: 2vmin 5vmin;
	border-bottom: 2px solid  #c9e4ea;
  background-color: #333333;
}
td{
  color: #c9e4ea;
	padding: 9px 8px 0px 8px;
}
tr:nth-child(even){
  background-color: #424242;
}

tr:hover td
{
	color:  #66ddf7;
  background-color: #616161
}

input[type=number]{
  font-size: larger;
  text-align: center;
  color: #D3D3D3;
  background-color: transparent;
  border: none;
  padding: 0;
  margin: 0;
  width: 100%
}

input[type=number]:focus {
  background-color: lightblue;
  color:#000;
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
  margin: 10px;
  border: 1px solid rgb(122, 132, 97);
  box-shadow: 2px 2px 4px 1px rgb(174, 174, 159);
}
label{
  color: #fff;
  font-size: larger;
  font-weight: bold;
  text-shadow: 0 0 15px #66ddf7,
    0 0 45px #66ddf7;
}


.chart-container {
  width:80%;
  display: flex; 
  justify-content: space-between;
  flex-direction: column;
  margin-left: 10%;
}

.chart-container{
  background: linear-gradient(to bottom,  #5b949d, #282828);
  color: #fff;
}

.slider-span {
  display: inline-block;
  text-align: left;
  width: 10%;
  margin-left: 5%;
  color: #D3D3D3;
}

.table-header{
  font: 1em sans-serif;
}

.btn{
  display: flex;
  text-align: center; 
  width: 20vmin;
  padding: 10px;
  position: relative;
  background-color: rgba(45,45,45, 1);
  border-radius: 2rem;
  color: #999;
  font-size: 2rem;
  text-decoration: none;
  transition: .5s;
  overflow: hidden;
  margin: 3vmin auto;
}
.btn:hover{
  color: #66ddf7;
  text-shadow: 0 0 15px #66ddf7,
              0 0 45px #66ddf7;
}
.btn:before{
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  transform: translate(-50%, -50%);
  background: radial-gradient(#66ddf7, transparent, transparent);
  opacity: 0;
  transition: .5s;
}
.btn:hover::before{
  opacity: 1;
}
.btn::after{
  content: '';
  background-color: rgba(45,45,45, 1);;
  position: absolute;
  inset: 2px;
  border-radius: 48px;
}
.btn-span{
  position: relative;
  margin-left: 2vmin;
  z-index: 1;
}
.btn-container{
  margin: 0 45%;
  justify-content: center;
} 

.switch {
  display: inline-block;
  height: 34px;
  position: relative;
  width: 60px;
}

.switch input {
  display:none;
}

.slider {
  background-color: #ccc;
  bottom: 0;
  cursor: pointer;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
  transition: .4s;
}

.slider:before {
  background-color: #fff;
  bottom: 4px;
  content: "";
  height: 26px;
  left: 4px;
  position: absolute;
  transition: .4s;
  width: 26px;
}

input:checked + .slider {
  background-color: #66ddf7;
}

input:checked + .slider:before {
  transform: translateX(26px);
}

.slider.round {
  border-radius: 34px;
}

.slider.round:before {
  border-radius: 50%;
}

.checkbox-container {
  margin: 0 auto;
}
.sliders-container {
  display: flex; 
  justify-content: space-between;
}
.slider-container {
  margin: 2vmin 8vmin;
}
.slider-container {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 0.8vmin 0;
    width: 30vmin;
    background: linear-gradient(to bottom,  #5b949d, rgba(34, 62, 62, 0.736));
    border-radius: 1vmin;
    box-shadow:
        0.5vmin 0.5vmin 0.5vmin rgba(66, 173, 155, 0.1),
        0.5vmin -0.5vmin 2vmin #889696;

}
.slider-container span{
  font-weight: bolder;
}

.summary{
  display: flex; 
  justify-content: center;
  align-items: center;
}

.card-container{
    font-family: 'Lucida Sans', 'Lucida Sans Regular', 'Lucida Grande', 'Lucida Sans Unicode';
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 5vmin 0;
    flex-wrap: wrap;
}

.card-container .card{
    position: relative;
    width: 30vmin;
    height: 20vmin;
    color: #fff;
    background: #111;
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 20px 30px;
    transition: .5s;
} 
.card-container .card:hover{
  transform: translateY(-20px);
}

.card-container .card::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, #d2a52b, #ff0058);
}

.card-container .card::after {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: linear-gradient(45deg, #ffbc00, #ff0058);
    filter: blur(2vmin);
}

.card-container .card:nth-child(2):after,
.card-ontainer .card:nth-child(2):before {
    background: linear-gradient(315deg, #03a9f4, #ff0058);
}

.card-container .card:nth-child(3):after,
.card-container .card:nth-child(3):before {
    background: linear-gradient(315deg, #4dff03, #00d0ff);
}

.card-container .card span{
    position: absolute;
    inset: 0.5vmin;
    background: rgb(0, 0, 0, 0.6);
    z-index: 2;
}

.card-container .card span::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    width: 50%;
    height: 100%;
    background: rgba(255, 255, 255, 0.4);
    pointer-events: none;
}

.card-container .card .card-content {
    position: relative;
    z-index: 10;
    padding: 20px 40px;
}

.card-container .card .card-content h2 {
    font: 2em "";
    color: #fff;
    margin-bottom: 10px;
}

.card-container .card .card-content p {
    font: 1.1em/1.4em "";
    color: #fff;
    margin-bottom: 10px;
}

</style>