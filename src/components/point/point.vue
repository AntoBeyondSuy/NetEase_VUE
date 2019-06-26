<!-- 散点图 -->
<!-- 对应项目中的 地图表 -->
<style lang="stylus" scoped>
  .point
    background url('../../assets/bg.jpg') no-repeat
    background-size 100% 100%

    .main
      height calc(100% - 120px)
      width 100%
      transition all 0.5s linear
</style>

<template lang="html">
  <div class="point">
    <v-header :name="name" :legendArr="legendArr" :myChart="myChart"></v-header>
    <v-filter :myChart="myChart" v-if="myChart._dom"></v-filter>
    <div class="main"></div>
  </div>
</template>

<script>
  import axios from 'axios'
  import echarts from 'echarts'
  import china from 'echarts/map/js/china'
  import header from 'components/header/header'
  import filter from 'components/filter/filter'

  const USER_NAME = 'elastic';
  const PSW = 'elasticl@ethical.cn';
  const AUTH_TOKEN = "Basic " + btoa(USER_NAME + ":" + PSW);


  export default {
    created() {
      this._getCityData()
    },
    data() {
      return {
        legendArr: [],
        color: this.$store.state.color,
        myChart: {},
        geoCoordMap: {},
        name: '网易云音乐 - 用户地域分布'
      }
    },
    methods: {
      _init(options) {
        this.myChart = echarts.init(document.querySelector('.point .main'));
        this.myChart.setOption(options);
        this.legendArr = options.series;
        this.legendArr.forEach((data) => {
          data.selected = true;
        });
        this.$root.charts.push(this.myChart);
        window.addEventListener('resize', function () {
          this.myChart.resize()
        }.bind(this))
      },
      _getCityData() {
        axios.get('static/data/citys.json').then((res) => {
          this.geoCoordMap = res.data;
          this.$nextTick(() => {
            this._getMyChart()
          })
        })
      },
      convertData(data, begin, end) {
        // document.write(begin + "  " + end)
        let res = [];
        for (let i = 0; i < data.length; i++) {
          if (data[i].value > begin && data[i].value <= end) {
            // let l = data.length
            /* Math.random() [0,1) */
            // let x = parseInt(Math.random() * l)
            let geoCoord = this.geoCoordMap[data[i].name];   // 地理坐标
            /* data[i].name 是地名：海门 鄂尔多斯 南京 南充 天津 富阳 泰安 诸暨 郑州 哈尔滨 上海 */
            /* geoCoord 是搜索 cityData.json 获取的对应坐标 */
            // document.write(data[i].name + "-" + data[i].value + "-" + geoCoord.concat(data[i].value) + " ")
            if (geoCoord) {
              res.push({
                name: data[i].name,
                // name: data[x].name,
                // value: geoCoord.concat(Math.random() * 200)
                value: geoCoord.concat(data[i].value)
              });
            }
          }
        }
        return res;
      },
      _getMyChart() {
        axios.get('static/data/point/citys_value.json').then((res) => {
          let options = {
            // backgroundColor: '#404a59',
            title: {
              show: false
            },
            tooltip: {
              trigger: 'item',
              formatter: function (params) {
                /* 天津：105 没错！！ */
                return params.name + ' : ' + params.value[2] + "人";
              }
            },
            legend: {
              show: false
            },
            visualMap: {
              bottom: 30,
              left: 200,
              itemSymbol: 'roundRect',
              pieces: [
                {gt: 10000},            // (1500, Infinity]
                {gt: 2000, lte: 10000},  // (2000, 10000]
                {gt: 500, lte: 2000},  // (500, 2000]
                {gt: 200, lte: 500},  // (200, 500]
                {gt: 100, lte: 200},  // (100, 200]
                {gt: 50, lte: 100},   // (50, 100]
                {gt: 20, lte: 50},       // (20, 50]
                {gt: 0, lte: 20}                 // (-Infinity, 5)
              ],
              splitNumber: 8,
              inRange: {
                color: [
                  '#90b9cc', '#b0becc', '#ccb7bf',
                  '#cc9495', '#cc7673', '#cc5a56',
                  '#cc332b', '#cc1b12']
              },
              textStyle: {
                color: '#fff'
              }
            },
            geo: {
              map: 'china',
              label: {
                emphasis: {
                  show: false
                }
              },
              zoom: 1,
              top: 50,
              itemStyle: {
                normal: {
                  color: '#3c4247',
                  opacity: 0.6,
                  borderColor: 'rgba(255, 255, 255, 0.35)'
                },
                emphasis: {
                  color: '#2a333d'
                }
              }
            },
            series: [{
              name: '10k+',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 10000 +
                let size = val[2];
                if (size > 10000) {
                  return size * 0.01;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 10000, 50000)
            }, {
              name: '2k~10k',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 2000 ~ 10000
                let size = val[2];
                if (size > 2000 && size <= 10000) {
                  return size * 0.02;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 2000, 10000)
            }, {
              name: '500~2k',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 500 ~ 2000
                let size = val[2];
                if (size > 500 && size <= 2000) {
                  return size * 0.04;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 500, 2000)
            }, {
              name: '200~500',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 200 ~ 500
                let size = val[2];
                if (size > 200 && size <= 500) {
                  return size * 0.055;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 200, 500)
            }, {
              name: '100~200',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 100 ~ 200
                let size = val[2];
                if (size > 100 && size <= 200) {
                  return size * 0.07;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 100, 200)
            }, {
              name: '50~100',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 50 ~ 100
                let size = val[2];
                if (size > 50 && size <= 100) {
                  return size * 0.09;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 50, 100)
            }, {
              name: '20~50',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 20 ~ 50
                let size = val[2];
                if (size > 20 && size <= 50) {
                  return size * 0.1;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 20, 50)
            }, {
              name: '20~0',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                // 20 ~ 50
                let size = val[2];
                if (size <= 20) {
                  return size * 0.4;
                }
              },
              label: {
                normal: {
                  show: false
                },
                emphasis: {
                  show: false
                }
              },
              itemStyle: {
                emphasis: {
                  borderColor: '#fff',
                  borderWidth: 1
                }
              },
              data: this.convertData(res.data, 0, 20)
            }]
          };
          this._init(options)
        });
      }
    },
    components: {
      'v-header': header,
      'v-filter': filter
    }
  }

</script>
