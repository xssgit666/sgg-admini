<template>
  <div>
    <el-card>
      <div slot="header">
        <div class="category-header">
          <span>销售额类别占比</span>
          <el-radio-group
            v-model="value"
            size="mini"
          >
            <el-radio-button label="全部"></el-radio-button>
            <el-radio-button label="线上"></el-radio-button>
            <el-radio-button label="门店"></el-radio-button>
          </el-radio-group>
        </div>
      </div>
      <div>
        <div ref="charts" class="charts"></div>
      </div>
    </el-card>
  </div>
</template>

<script>
import * as echarts from 'echarts'
export default {
  name: 'Category',
  data() {
    return {
      value: '全部'
    }
  },
  mounted() {
    const mychart = echarts.init(this.$refs.charts)
    mychart.setOption({
      title: {
        text: '搜索引擎',
        subtext: 1048,
        left: 'center',
        top: '43%'
      },
      tooltip: {
        trigger: 'item'
      },
      series: [
        {
          name: '销售额类别占比',
          type: 'pie',
          radius: ['40%', '70%'],
          avoidLabelOverlap: false,
          itemStyle: {
            borderColor: '#fff',
            borderWidth: 2
          },
          labelLine: {
            show: true
          },
          data: [
            { value: 1048, name: '搜索引擎' },
            { value: 735, name: '直接访问' },
            { value: 580, name: '邮件营销' },
            { value: 484, name: '联盟广告' },
            { value: 300, name: '视频广告' }
          ]
        }
      ]
    })
    mychart.on('mouseover', (params) => {
    // 获取鼠标移上去那条数据
      const { name, value } = params.data
      // 重新设置标题
      mychart.setOption({
        title: {
          text: name,
          subtext: value
        }
      })
    })
  }
}
</script>

<style scoped lang='scss'>
::v-deep .el-card__header{
  padding: 13px 20px;
}
.category-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.charts {
  width: 100%;
  height: 280px;
}
</style>
