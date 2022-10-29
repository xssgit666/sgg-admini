<template>
  <div>
    <el-card>
      <div slot="header" class="header">
        <span>线上热门搜索</span>
        <el-dropdown>
          <span>
            <i class="el-icon-more"></i>
          </span>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>黄金糕</el-dropdown-item>
            <el-dropdown-item>狮子头</el-dropdown-item>
            <el-dropdown-item>螺蛳粉</el-dropdown-item>
            <el-dropdown-item>双皮奶</el-dropdown-item>
            <el-dropdown-item>蚵仔煎</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
      <div>
        <el-row :gutter="10">
          <el-col :span="12">
            <LineCharts></LineCharts>
          </el-col>
          <el-col :span="12">
            <LineCharts></LineCharts>
          </el-col>
        </el-row>
        <el-table
          :data="tableData"
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          style="width: 100%;margin-top: 10px;"
          border
        >
          <el-table-column
            prop="date"
            label="排名"
            type="index"
            width="80"
          >
          </el-table-column>
          <el-table-column
            prop="name"
            label="关键字"
          >
          </el-table-column>
          <el-table-column
            prop="address"
            label="用户数"
            sortable
          >
          </el-table-column>
          <el-table-column
            prop="address"
            label="周涨幅"
            sortable
          >
          </el-table-column>
        </el-table>
        <el-pagination
          style="text-align:center;margin-top:10px"
          background
          layout="prev, pager, next"
          :total="1000"
        >
        </el-pagination>
      </div>
    </el-card>
  </div>
</template>

<script>
import LineCharts from './lineCharts'
export default {
  name: 'Search',
  components: {
    LineCharts
  },
  props: [],
  data() {
    return {
      tableData: []
    }
  },
  computed: {},
  watch: {},
  created() {},
  mounted() {},
  methods: {}
}
</script>

<style scoped lang='scss'>
.header {
  display: flex;
  justify-content: space-between;
}
</style>
