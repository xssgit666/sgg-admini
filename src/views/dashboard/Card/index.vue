<template>
  <div>
    <el-row :gutter="10">
      <el-col :span="6">
        <el-card>
          <Detail title="总销售量" count="¥ 126560">
            <template slot="charts">
              <span style="margin-right:20px">
                周同比&nbsp;&nbsp;
                <span style="color:#67C23A">56.67%</span>
                <i
                  class="el-icon-caret-top"
                  style="color:#67C23A"
                ></i>
              </span>
              <span>
                日同比&nbsp;&nbsp;
                <span style="color:#F56C6C">19.96%</span>
                <i
                  class="el-icon-caret-bottom"
                  style="color:#F56C6C"
                ></i>
              </span>
            </template>
            <template slot="footer">
              <span>日销售额：
                <span style="color:#409EFF">¥12423</span>
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="访问量" count="88460">
            <template slot="charts">
              <lineCharts></lineCharts>
            </template>
            <template slot="footer">
              <span>日访问量：
                <span style="color:#409EFF">1234</span>
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="支付笔数" count="88460">
            <template slot="charts">
              <barCharts></barCharts>
            </template>
            <template slot="footer">
              <span>转化率：
                <span style="color:#409EFF">64%</span>
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
      <el-col :span="6">
        <el-card>
          <Detail title="运营活动效果" count="78%">
            <template slot="charts">
              <progressCharts></progressCharts>
            </template>
            <template slot="footer">
              <span style="margin-right:20px">
                周同比&nbsp;&nbsp;
                <span style="color:#67C23A">56.67%</span>
                <i
                  class="el-icon-caret-top"
                  style="color:#67C23A"
                ></i>
              </span>
              <span>
                日同比&nbsp;&nbsp;
                <span style="color:#F56C6C">19.96%</span>
                <i
                  class="el-icon-caret-bottom"
                  style="color:#F56C6C"
                ></i>
              </span>
            </template>
          </Detail>
        </el-card>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Detail from './Detail'
import lineCharts from './lineChart'
import barCharts from './barCharts'
import progressCharts from './progressCharts'
export default {
  name: 'Card',
  components: {
    Detail,
    lineCharts,
    barCharts,
    progressCharts
  }
}
</script>

<style>

</style>
