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

  const USER_NAME = 'elastic'
  const PSW = 'elasticl@ethical.cn'
  const AUTH_TOKEN = "Basic " + btoa(USER_NAME + ":" + PSW)


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
        name: '散点图'
      }
    },
    methods: {
      _init(options) {
        this.myChart = echarts.init(document.querySelector('.point .main'))
        this.myChart.setOption(options)
        this.legendArr = options.series
        this.legendArr.forEach((data) => {
          data.selected = true;
        })
        this.$root.charts.push(this.myChart)
        window.addEventListener('resize', function () {
          this.myChart.resize()
        }.bind(this))
      },
      _getCityData() {
        axios.get('static/data/citys.json').then((res) => {
          this.geoCoordMap = res.data
          this.$nextTick(() => {
            this._getMyChart()
          })
        })
      },
      /* 海门-9-121.15,31.89,9  鄂尔多斯-12-109.78,39.6,12 */
      convertData(data) {
        let res = [];
        /*
         * 这里是从 json 中随机取 4 个城市出来
         * 随机方式是先获取数据总长度，然后乘[0,1)随机数，取整x，选第x个数据
         */
        /* 在这里 i<4 限定了每个标签只有 4 个城市 */
        for (let i = 0; i < data.length; i++) {
          // let l = data.length
          /* Math.random() [0,1) */
          // let x = parseInt(Math.random() * l)
          let geoCoord = this.geoCoordMap[data[i].name]   // 地理坐标
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
                return params.name + ' : ' + params.value[2];
              }
            },
            legend: {
              show: false
            },
            visualMap: {
              min: 0,
              max: 200,
              bottom: 50,
              splitNumber: 5,
              inRange: {
                color: ['#255B78', '#2A7484', '#2F9696', '#3BBCB0', '#51D4EB']
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
              name: '人数',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function (val) {
                let size = val[2]
                if (size < 100) {
                  return size / 3;
                } else if (size < 400) {
                  return size / 5;
                } else if (size < 1000) {
                  return size / 11;
                } else if (size < 5000) {
                  return size / 55;
                } else if (size < 10000) {
                  return size / 110;
                } else if (size < 20000) {
                  return size / 220;
                } else {
                  // document.write(size + "\n")
                  return size / 550;
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
              data: this.convertData(res.data)
            }, {
              name: '标签2',
              type: 'scatter',
              coordinateSystem: 'geo',
              symbolSize: function(val) {
                return val[2] / 6;
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
              data: this.convertData(res.data)
            }]
          }
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

<!--
, {
            name: '标签2',
            type: 'scatter',
            coordinateSystem: 'geo',
            symbolSize: function(val) {
              return val[2] / 6;
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
            data: this.convertData(res.data)
          }, {
            name: '第3个标签！',
            type: 'scatter',
            coordinateSystem: 'geo',
            symbolSize: function(val) {
              return val[2] / 6;
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
            data: this.convertData(res.data)
          }
-->
