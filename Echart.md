## echart

```html
<template>
  <div class="charts">
     <div id="bottom">
          <div class="date-bottom">
            <div class="text_">
              Energy Profile
            </div>
            <div class="date_">
              <div class="dateSelector" style="display: flex; align-items: center;">
                <img src="@/assets/icons/left.png" @click="getPrev"/>
                <span style="margin: 0 0.1rem; font-size: 16px;">{{searchDate}}</span>
                <img src="@/assets/icons/right.png" @click="getNext"/>
              </div>
              <a-date-picker :allowClear="false" @change="validDateChange" v-model:value="value5">
                <template #suffixIcon>
                  <img src="@/assets/icons/calendar.png">
                </template>  
              </a-date-picker>
            </div>
          </div>
        </div>
    <div id="main"></div>
  </div>
</template>

<script>
import * as echarts from "echarts";
import {energyCurve,baseInfo} from '@/api/user'
import {getYMDHMS } from '@/utils/common';
import { defineComponent, ref } from 'vue';
import dayjs from 'dayjs';

import {mapGetters} from 'vuex'
export default {
  name: "main",
  data() {
    return {
        searchDate:null,
        value5: '',

    };
  },
  mounted() {
    this.baseInfos();
    this.overviewData()
    setInterval(()=>{
        this.overviewData();
      },1000*60*5)
  },
 computed:{
    // ...mapGetters(['chartData',]),
 },

  methods: {
     // 跳转到前一天
    getPrev(){
      let timeChuo = new Date(this.searchDate).getTime()//时间戳
      let dateTime = Number(parseInt(timeChuo/1000))
      dateTime-=86400
      this.searchDate = getYMDHMS(dateTime*1000)
      this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
      this.overviewData()

    },
        // 跳转到后一天
    getNext(){
      let timeChuo = new Date(this.searchDate).getTime()//时间戳
      let dateTime = Number(parseInt(timeChuo/1000))
      dateTime+=86400
      this.searchDate = getYMDHMS(dateTime*1000)
      this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
      this.overviewData()

    },
     validDateChange(moment, dateString) {
      this.searchDate=dateString
      this.overviewData()
    },
     baseInfos() {
        baseInfo({'energySn':'00000031122011987062603503980000'},res=>{
          if(res.errCode=='0') {
            this.searchDate=res.result.date
            this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
          }
        },fail=>{
  
        })
      },
    overviewData(){
        // console.log(this.chartData_,'chartData')
      let params={
        energySn:'00000031122011987062603503980000'
      }
    energyCurve(params,res=>{
        let data= res.result
        let batteryPowers=[]
        let gridPowers=[]
        let homeUsages=[]
        let minutes=[]
        let pvPowers=[]
        for(let i of data) {
          if(Object.keys(i).indexOf('batteryPower')!=-1) {
            let data=i['batteryPower']
            batteryPowers.push(data)
          }
          if(Object.keys(i).indexOf('gridPower')!=-1) {
            let data=i['gridPower']
            gridPowers.push(data)
          }
          if(Object.keys(i).indexOf('homeUsage')!=-1) {
            let data=i['homeUsage']
            homeUsages.push(data)
          }
          if(Object.keys(i).indexOf('minute')!=-1) {
            let data=i['minute']
            minutes.push(data)
          }
          if(Object.keys(i).indexOf('pvPower')!=-1) {
            let data=i['pvPower']
            pvPowers.push(data)
          }
        }
        
       this.sets(batteryPowers,gridPowers,homeUsages,minutes,pvPowers)
    })
    },
    sets(batteryPowers,gridPowers,homeUsages,minutes,pvPowers) {
      var chartDom = document.getElementById("main");
      var myChart = echarts.init(chartDom);
      var option;
      option = {
        backgroundColor:'transparent',
        color: ["#25CF42", "#FF3B30", "#3EC1F5", "#5D49FF"],
        title: {
          text: "",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "cross",
            label: {
              backgroundColor: "#6a7985",
            },
          },
        },
        legend: {
          data: ["PV Power", "Grid Power", "Home Usage", "Battery Power"],
        },
        toolbox: {
          feature: {
            saveAsImage: {},
          },
        },
        grid: {
          left: "1%",
          right: "1.5%",
          bottom: "3%",
          borderColor: "#ccc",
          containLabel: true,
        },
        xAxis: [
          {
            type: "category",
            boundaryGap: false,
            data:minutes,
          },
        ],
        yAxis: [
          {
            type: "value",
            axisLabel: {
              formatter: '{value} KW'
            }
          },
        ],
        dataZoom: [
          {
            type: 'inside',
            start: 0,
            end: 100
          },
          {
            start: 0,
            end: 100
          }
        ],
        
        series: [
          {
            name: "PV Power",
            type: "line",
            stack: "Total",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#DBFFDB'
            },
            emphasis: {
              focus: "series",
            },
            data: pvPowers,
          },
          {
            name: "Grid Power",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#EFA19C'

            },
            emphasis: {
              focus: "series",
            },
            data: gridPowers,
          },
          {
            name: "Home Usage",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#BCF3F8'
            },
            emphasis: {
              focus: "series",
            },
            data: homeUsages,
          },
          {
            name: "Battery Power",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#D5D1f6'
            },
            emphasis: {
              focus: "series",
            },
            data:batteryPowers,
          }
        
        ],
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#999'
            }
          },
          formatter: function (params) {
            var relVal = params[0].name
            for (var i = 0, l = params.length; i < l; i++) {
              relVal += '<br/>' + params[i].marker +params[i].seriesName+'  '+ params[i].value + 'KW'
            }
            return relVal
          }
        }


        
      };

      option && myChart.setOption(option);
    },
  },
  watch: {},
  created() {},
};
</script>
<style scoped lang="less">
.charts {
    box-shadow: 0px 3px 7px 0px rgba(0,0,0,0.3);
    border-radius:10px;
    background:white;
    padding-top:.01rem;
    margin-top:.15rem;
}
#main {
  width: 100%;
  height: 300px;
}
#bottom{
  position: relative;
  width: 100%;
  margin-top: 0.2rem;
  .date-bottom {
    display: flex;
    width: 100%;
    position: absolute;
    align-items: center;
    top:10%;
    .text_ {
      position: absolute;
      font-size:20px;
      z-index: 10;
      left:4%;
    }
    .date_ {
      position: absolute;
      display: flex;
      right:4%;
      .dateSelector{
        margin-right:.15rem; 
        img {
          cursor: pointer;
        }
      } 
  }
  }
 
}
</style>




```

```html
<template>
  <div class="charts">
     <div id="bottom">
          <div class="date-bottom">
            <div class="text_">
              Energy Profile
            </div>
            <div class="date_">
              <div class="dateSelector" style="display: flex; align-items: center;">
                <img src="@/assets/icons/left.png" @click="getPrev"/>
                <span style="margin: 0 0.1rem; font-size: 16px;">{{searchDate}}</span>
                <img src="@/assets/icons/right.png" @click="getNext"/>
              </div>
              <a-date-picker :allowClear="false" @change="validDateChange" v-model:value="value5">
                <template #suffixIcon>
                  <img src="@/assets/icons/calendar.png">
                </template>  
              </a-date-picker>
            </div>
          </div>
        </div>
    <div id="main"></div>
  </div>
</template>

<script>
import * as echarts from "echarts";
import {energyCurve,baseInfo} from '@/api/user'
import {getYMDHMS } from '@/utils/common';
import { defineComponent, ref } from 'vue';
import dayjs from 'dayjs';

import {mapGetters} from 'vuex'
export default {
  name: "main",
  data() {
    return {
        searchDate:null,
        value5: '',

    };
  },
  mounted() {
    this.baseInfos();
    this.overviewData()
    setInterval(()=>{
        this.overviewData();
      },1000*60*5)
  },
 computed:{
    // ...mapGetters(['chartData',]),
 },

  methods: {
     // 跳转到前一天
    getPrev(){
      let timeChuo = new Date(this.searchDate).getTime()//时间戳
      let dateTime = Number(parseInt(timeChuo/1000))
      dateTime-=86400
      this.searchDate = getYMDHMS(dateTime*1000)
      this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
      this.overviewData()

    },
        // 跳转到后一天
    getNext(){
      let timeChuo = new Date(this.searchDate).getTime()//时间戳
      let dateTime = Number(parseInt(timeChuo/1000))
      dateTime+=86400
      this.searchDate = getYMDHMS(dateTime*1000)
      this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
      this.overviewData()

    },
     validDateChange(moment, dateString) {
      this.searchDate=dateString
      this.overviewData()
    },
     baseInfos() {
        baseInfo({'energySn':'00000031122011987062603503980000'},res=>{
          if(res.errCode=='0') {
            this.searchDate=res.result.date
            this.value5=ref(dayjs(this.searchDate, 'YYYY/MM/DD'))
          }
        },fail=>{
  
        })
      },
    overviewData(){
        // console.log(this.chartData_,'chartData')
      let params={
        energySn:'00000031122011987062603503980000'
      }
    energyCurve(params,res=>{
        let data= res.result
        let batteryPowers=[]
        let gridPowers=[]
        let homeUsages=[]
        let minutes=[]
        let pvPowers=[]
        for(let i of data) {
          if(Object.keys(i).indexOf('batteryPower')!=-1) {
            let data=i['batteryPower']
            batteryPowers.push(data)
          }
          if(Object.keys(i).indexOf('gridPower')!=-1) {
            let data=i['gridPower']
            gridPowers.push(data)
          }
          if(Object.keys(i).indexOf('homeUsage')!=-1) {
            let data=i['homeUsage']
            homeUsages.push(data)
          }
          if(Object.keys(i).indexOf('minute')!=-1) {
            let data=i['minute']
            minutes.push(data)
          }
          if(Object.keys(i).indexOf('pvPower')!=-1) {
            let data=i['pvPower']
            pvPowers.push(data)
          }
        }
        
       this.sets(batteryPowers,gridPowers,homeUsages,minutes,pvPowers)
    })
    },
    sets(batteryPowers,gridPowers,homeUsages,minutes,pvPowers) {
      var chartDom = document.getElementById("main");
      var myChart = echarts.init(chartDom);
      var option;
      option = {
        backgroundColor:'transparent',
        color: ["#25CF42", "#FF3B30", "#3EC1F5", "#5D49FF"],
        title: {
          text: "",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "cross",
            label: {
              backgroundColor: "#6a7985",
            },
          },
        },
        legend: {
          data: ["PV Power", "Grid Power", "Home Usage", "Battery Power"],
        },
        toolbox: {
          feature: {
            saveAsImage: {},
          },
        },
        grid: {
          left: "1%",
          right: "1.5%",
          bottom: "3%",
          borderColor: "#ccc",
          containLabel: true,
        },
        xAxis: [
          {
            type: "category",
            boundaryGap: false,
            data:minutes,
          },
        ],
        yAxis: [
          {
            type: "value",
            axisLabel: {
              formatter: '{value} KW'
            }
          },
        ],
        dataZoom: [
          {
            type: 'inside',
            start: 0,
            end: 100
          },
          {
            start: 0,
            end: 100
          }
        ],
        
        series: [
          {
            name: "PV Power",
            type: "line",
            stack: "Total",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#DBFFDB'
            },
            emphasis: {
              focus: "series",
            },
            data: pvPowers,
          },
          {
            name: "Grid Power",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#EFA19C'

            },
            emphasis: {
              focus: "series",
            },
            data: gridPowers,
          },
          {
            name: "Home Usage",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#BCF3F8'
            },
            emphasis: {
              focus: "series",
            },
            data: homeUsages,
          },
          {
            name: "Battery Power",
            type: "line",
            stack: "Total",
            // symbol: "none",
            showSymbol: false,
            symbolSize:[10,10],
            smooth: true,
            areaStyle: {
              color:'#D5D1f6'
            },
            emphasis: {
              focus: "series",
            },
            data:batteryPowers,
          }
        
        ],
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            crossStyle: {
              color: '#999'
            }
          },
          formatter: function (params) {
            var relVal = params[0].name
            for (var i = 0, l = params.length; i < l; i++) {
              relVal += '<br/>' + params[i].marker +params[i].seriesName+'  '+ params[i].value + 'KW'
            }
            return relVal
          }
        }


        
      };

      option && myChart.setOption(option);
    },
  },
  watch: {},
  created() {},
};
</script>
<style scoped lang="less">
.charts {
    box-shadow: 0px 3px 7px 0px rgba(0,0,0,0.3);
    border-radius:10px;
    background:white;
    padding-top:.01rem;
    margin-top:.15rem;
}
#main {
  width: 100%;
  height: 300px;
}
#bottom{
  position: relative;
  width: 100%;
  margin-top: 0.2rem;
  .date-bottom {
    display: flex;
    width: 100%;
    position: absolute;
    align-items: center;
    top:10%;
    .text_ {
      position: absolute;
      font-size:20px;
      z-index: 10;
      left:4%;
    }
    .date_ {
      position: absolute;
      display: flex;
      right:4%;
      .dateSelector{
        margin-right:.15rem; 
        img {
          cursor: pointer;
        }
      } 
  }
  }
 
}
</style>




```

