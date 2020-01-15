<template>
  <div id="root"></div>
</template>
<script>
export default {
  data () {
    return {
      myChart: null
    }
  },
  mounted () {
    const barOption = {
      title: {
        text: 'ECharts 入门示例'
      },
      tooltip: {},
      xAxis: {
        data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
      },
      yAxis: {},
      series: [{
        name: '销量',
        type: 'bar',
        data: [5, 20, 36, 10, 10, 20]
      }]
    }
    console.log(barOption)
    const pieOption = {
      backgroundColor: '#2c343c',
      textStyle: {
        color: 'rgba(255, 255, 255, 0.3)'
      },
      visualMap: {
        show: false,
        min: 80,
        max: 600,
        inRange: {
          colorLightness: [0, 1]
        }
      },
      series: [
        {
          name: '这是什么？',
          type: 'pie',
          radius: '55%',
          roseType: 'angle',
          data: [
            { value: 222, name: '是个图' },
            { value: 333, name: '是个表' },
            { value: 555, name: '是个图表' }
          ],
          itemStyle: {
            color: '#c23531',
            shadowBlur: 200,
            shadowOffsetX: 0,
            shadowOffsetY: 0,
            shadowColor: 'rgba(0,0,0,0.5)'
          },
          labelLine: {
            lineStyle: {
              color: 'rgba(255, 255, 255, 0.3)'
            }
          }
        }
      ]
    }
    this.myChart = this.$echarts.init(document.getElementById('root'))
    this.myChart.setOption(pieOption)
  },
  methods: {
  }
}
</script>
<style scoped>
  #root{
    width: 600px;
    height: 400px;
  }
</style>
