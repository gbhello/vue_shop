<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <div id="main" style="width: 600px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
import echarts from 'echarts'
import _ from 'lodash'

export default {
  data() {
    return {
      reportData: {},
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created() {
    this.getReportData()
  },
  methods: {
    async getReportData() {
      const { data: res } = await this.$http.get('reports/type/1')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取统计数据失败！')
      }
      this.reportData = res.data
      var myChart = echarts.init(document.getElementById('main'))

      const result = _.merge(res.data, this.options)
      myChart.setOption(result)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
