<template lang="html">
  <div class="title">
    <h1>{{name}}</h1>
    <div class="legend-wrapper">
      <ul>
        <li v-for="(legend,index) in legendArr" v-on:mouseout="donwplay(index)" v-on:mouseover="highlight(index)" :style="styleArr[index]" @click="legendToggle(legend)">
          {{legend.name}}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    legendArr: {
      type: Array,
      default: []
    },
    myChart: Object,
    name: String
  },
  created() {
    this._init()
  },
  data() {
    return {
      styleArr: [],
      color: this.$store.state.color
    }
  },
  methods: {
    _init() {
      this.color.forEach((color) => {
        this.styleArr.push({
          background: color,
          border: '1px solid' + color
        })
      })
    },
    highlight(index) {
      this.myChart.dispatchAction({
        type: 'highlight',
        seriesIndex: index
      });
    },
    donwplay(index) {
      this.myChart.dispatchAction({
        type: 'downplay',
        seriesIndex: index
      })
    },
    legendToggle(legend) {
      legend.selected = !legend.selected;
      this.myChart.dispatchAction({
        type: 'legendToggleSelect',
        name: legend.name
      });
      this.changeStyle()
    },
    changeStyle() {
      this.legendArr.forEach((data, index) => {
        if (data.selected) {
          this.styleArr[index].background = this.color[index];
          this.styleArr[index].border = '1px solid' + this.color[index]
        } else {
          this.styleArr[index].background = 'transparent';
          this.styleArr[index].border = '1px solid #9C8C84'
        }
      })
    }
  }
}

</script>

<style lang="stylus" scoped>
.title
  position relative
  display flex
  height 50px
  line-height 50px
  background-color rgba(32, 32, 35, 0.2)
  color white
  width 100%
  h1
    flex 0 0 280px
    font-size 21px
    font-weight bold
    padding-left 20px
  ul
    position absolute
    right 0
    padding-right 20px
    margin-top -2px
    li
      display inline-block
      min-width 59px
      padding 2px 10px 2px 10px
      line-height 20px
      text-align center
      font-size 11px
      &:first-child
        border-top-left-radius 5px
        border-bottom-left-radius 5px
      &:last-child
        border-top-right-radius 5px
        border-bottom-right-radius 5px
      &+li
        margin-left: -1px
</style>
