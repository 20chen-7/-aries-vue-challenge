<template>
  <div class="option-profit-calculator">
    <!-- Here is the title and one information icon for instructions -->
    <div class="title-box">
      <h1>Options Profit Calculator
        <!-- this is the information icon, click and warning dia model pop up -->
      <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" class="bi bi-info-circle intro" viewBox="0 0 16 16" @click="showInstruction"> <path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z"/> <path d="m8.93 6.588-2.29.287-.082.38.45.083c.294.07.352.176.288.469l-.738 3.468c-.194.897.105 1.319.808 1.319.545 0 1.178-.252 1.465-.598l.088-.416c-.2.176-.492.246-.686.246-.275 0-.375-.193-.304-.533L8.93 6.588zM9 4.5a1 1 0 1 1-2 0 1 1 0 0 1 2 0z"/> </svg>
      </h1>
          <WarningDia v-bind:visible.sync="showInstructionDialog" title="Warning">
            <template v-slot:header>
              <h3>Options Profit Calculator Instructions</h3>
            </template>
            <template>
                <p>1. Based on the details of four type of strike, you can check up tp four strike and choose the number of Contract for each option. </p>
                <p>2. The <strong>Min Price</strong> and <strong>Max Price</strong> are the lower boundary and upper boundary of current ASSET PRICE range, and <strong>Step Change</strong> is the increment 
                between the two range.</p>
                <p>3. After that, click the <strong>Generate</strong> button, you will get the chart based on your choices, and the <strong>Max Profit</strong>, <strong>Max Loss</strong> and <strong>Break Even Points</strong>
                        for each strike and total options profits will be shown in the bottom. </p>
                <p>4. Finially, there are dashboard, showing the Max Profit, Max Loss, Break Even Points for each strike, and the total Results.</p>
            </template>
            <template v-slot:footer>
              <button @click="showInstructionDialog = false">Dismiss</button>
            </template>
          </WarningDia>
      <div class="strike-table-intro">
          <p class="strike-intro-para">Here is the list of Strike and visualization of the option profit.</p>      
        </div>
    </div>
    <!-- Here is the content of the calculator -->
    <div class="content">
      <div class="strike-list">
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
            <tr v-for="(option, index) in localOptionsData" :key="index" @click="toggleCheckbox(index)">
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
      <div class="strike-visualization">
        <div class="sliders-container">
          <div class="slider-container">
              <input
              type="range"
              class="slider-range"
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
              class="slider-range"
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
              class="slider-range"
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
          <line-chart v-if="loaded" :data="chartData" :options="options"></line-chart>
        </div>
      </div>
      <div v-if="loaded" class="summary">
        <div class="card-container">
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
    const localOptionsData = [...this.optionsData];//make a local copy
    localOptionsData.forEach((option)=>{
      option.number = 1; //initial number of contract
      option.checked = false; 
      datasets.push({});
      });
    return {
      localOptionsData: localOptionsData,
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
      this.localOptionsData[index].checked = !this.localOptionsData[index].checked;
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
      
      this.localOptionsData.forEach((option, index)=>{
        profits.push({id: index, data:[],backgroundColor:backgroundColors[index],borderColor:borderColors[index], breakEvenPoint:this.breakEvenPoints[index]})
      });
      const totalProfits = [];
      const prices = [];
      const totalBreakEvenPoint = [];
      this.localOptionsData.filter(option=>option.checked).forEach((index)=>{
        totalBreakEvenPoint.push(this.breakEvenPoints[index]);
      })

      for (let price = this.minPriceVal; price <= this.maxPriceVal; price+=this.stepVal){
        prices.push(price);
        let totalProfit = 0;
        for(let index = 0; index < this.localOptionsData.length; index++){
          let option = this.localOptionsData[index];
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
          
        }
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
        },
        plugins: {
          legend: {
            labels: {
              color: 'white',
            }
          },
          title: {
            display: true,
            text: 'Risk vs Reward Graph',
            color: 'white', 
            font: {
              size: '20vmin',
            }
          }
         } 
      }
      
    },
    summary(){
      if (!this.chartData.datasets.length) {
        return [];
      }
      this.summaryArray = this.chartData.datasets.map((dataset) => {
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
      if (!this.localOptionsData.filter(option => option.checked).length){
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
    },
  },
  
}
</script>

<style scoped>
.option-profit-calculator{
  text-align: center;
  background-color: #000000c6;
}
/** title */
.title-box{
  padding: 2vh 10vw;
  margin: auto 20vw;
  background-color: #fff;
  border-radius: 4vmin;
  color:#66ddf7;
  position: relative;
  overflow: hidden;
  z-index: 10;
}
.title-box::before{
  content: '';
  width: 200vw;
  height: 200vh;
  background-color: #424242;;
  position: absolute;
  left: -70vw;
  top: -91vh;
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
  background-color:#424242;;
  border-radius: 4vmin;
  z-index: -1;
}
h1 {
  font-size: 3vmin;
}
p {
  font-size: 2vmin
}
.intro{
  width: 2vmin;
  height: 2vmin;
}
.strike-intro-para{
  margin-bottom: auto;
}

/* strike details */

.strike-list{
  width: 100%;
  overflow-x: auto;
  margin: 2vh auto;
  text-align: center;
}

table{
	font-family: "Lucida Sans Unicode", "Lucida Grande", Sans-Serif;
	font-size: 2vmin;
  background-color: #282828;
  color:#D3D3D3;
  box-shadow: 0 0 2px #66ddf7,
              0 0 5px #66ddf7;
  border: 1px solid #ccc;
  width: 90%;
  padding: 5vmin;
  border-collapse: collapse;
  border-spacing: 0;
  margin: 0 auto;
}

table th{
  font-size: 3vmin;
	font-weight: bold;
	color:  #34b7d4;
	padding: 2vmin 5vmin;
	border-bottom: 2px solid  #c9e4ea;
  background-color: #333333;
}
table td{
  color: #c9e4ea;
	padding: 3px 2px 0px 2px;
}
table tr:nth-child(even){
  background-color: #424242;
}
table tr{
  border: 0.2vmin solid #ddd;
  padding: 1vmin;
}
table th {
    text-transform: uppercase;
    font-size: 2vmin;
    letter-spacing: 0.1vmin;
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
  width: 100%;
}

input[type=number]:focus {
  background-color: lightblue;
  color:#000;
}

/* switch */
.switch {
  display: inline-block;
  height: 3.8vmin;
  position: relative;
  width: 7vmin;
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
  transition: .1s;
}

.slider:before {
  background-color: #fff;
  bottom: 0.4vmin;
  content: "";
  height: 3vmin;
  width: 3vmin;
  left: 0.3vw;
  position: absolute;
  transition: .4s;
}

input:checked + .slider {
  background-color: #66ddf7;
}

input:checked + .slider:before {
  transform: translateX(2.6vmin);
}

.slider.round {
  border-radius: 4vmin;
}

.slider.round:before {
  border-radius: 50%;
}


.checkbox-container {
  margin: 0 auto;
}

/* slider-for x */
.sliders-container {
  display: flex; 
  align-items: center;
}

.slider-container {
  height:10%;
  width: 20%;
  margin: auto 8vw;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0.8vmin 0;
  background:#d4dbdb;
  border-radius: 3vmin;
  box-shadow:
    0.2vmin 0.2vmin 0.5vmin rgba(66, 173, 155, 0.1),
    0.2vmin -0.2vmin 2vmin #889696;
}

.slider-span {
  font-weight: bold;
  font-size: 1vw;
  display: inline-block;
  text-align: left;
  width: 3vw;
  margin-left: 1vw;
  color: #472b2bf9;;
}

.slider-range{
  -webkit-appearance: none;
  height: 0.2vmin;
  width: 10vw;
  background: #ddd;
  background-color: #472b2b83;
  outline: none;
  border: none;
  -webkit-transition: .2s;
  transition: opacity .2s;
  opacity: 0.7;
}
.slider-range::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 2.5vmin;
  height: 2.5vmin;
  background: radial-gradient(#64c6dc, #38775e, #afd550);
  border-radius: 50%;
  cursor: pointer;
  }

.slider-range::-moz-range-thumb {
  cursor: pointer;
}
.slider-range:hover {
  opacity: 1;
}

/* button */
.generate-btn-container{
  display: block;
  align-items: center;
}
.btn{
  text-align: center; 
  width: 20vmin;
  padding: 10px;
  position: relative;
  background-color: rgba(243, 251, 244, 0.882);
  border-radius: 1vmin;
  border: 2 px solid cadetblue;
  color: rgb(37, 134, 137);
  font-size: 3vmin;
  text-decoration: none;
  transition: all 1s;
  overflow: hidden;
  margin: 3vmin auto;
}

.btn:before{
  content: '';
  position: absolute;
  display: block;
  height: 100%;
  width: 0%;
  background-color: rgb(27, 95, 95);
  top: 0;
  left: -2vmin;
  transform: skewX(45deg);
  transition: all 1s;
  z-index: -1;
}
.btn:hover::before{
  width: 180%;
}
.btn:hover{
  font-weight: bold;
  color: rgb(220, 240, 239);
  background-color: #0f453a;
  text-shadow: 0 0 15px #66ddf7,
              0 0 45px #66ddf7;
}


/* visualization  */
.strike-visualization{
  align-items: center;
  display: block;
  position: relative;
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
    height: 100%;
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

/* small screen */
@media screen and (max-width: 768px) {

table {
  border: 0;
}


table thead {
  display: none;
}


table tr {
  margin-bottom: 10px;
  display: block;
  border-bottom: 2px solid #ddd;
}


table td {
  display: block;
  font-size: 13px;
  border-bottom: 1px dotted #ccc;
}


table td:last-child {
  border-bottom: 0;
}


table td:before {
  content: attr(data-label);
  float: left;
  text-transform: uppercase;
  font-weight: bold;
}

.sliders-container {
  display: block;
}
.slider-container {
  width: 80%;
  margin: 2vmin auto;
  height: 2vmin;
}
input[type=range]{
  width: 50vw;
  height: 0.5vmin;
  
}
.slider-span {
  font-weight: bold;
  font-size: 1.8vw;
  display: inline-block;
  text-align: left;
  width: 18vw;
  margin-left: 1vw;
  color: #472b2bf9;;
}

.card-container .card{
  flex: 1 1 100%;
  margin: 2vh 20vw;

} 


}

</style>