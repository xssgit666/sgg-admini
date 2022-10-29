<template>
  <div>
    <el-card>
      <CategorySelect
        :show="scene !== 0"
        @getCategoryId="getCategoryId"
      ></CategorySelect>
    </el-card>

    <el-card style="margin: 20px 0">
      <!-- 展示SPU列表的结构 -->
      <div v-show="scene == 0">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          style="margin: 0 0 20px"
          @click="addSpu"
        >添加SPU</el-button>

        <!-- 表格 -->
        <el-table
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          :data="records"
          style="width: 100%"
          border
        >
          <el-table-column
            type="index"
            label="序号"
            width="80"
          ></el-table-column>

          <el-table-column
            prop="spuName"
            label="SPU名称"
          ></el-table-column>

          <el-table-column
            prop="description"
            label="SPU描述"
          ></el-table-column>

          <el-table-column
            label="操作"
          >
            <template slot-scope="{ row }">
              <el-tooltip
                class="item"
                effect="dark"
                content="添加"
                placement="top"
              >
                <el-button
                  type="success"
                  icon="el-icon-plus"
                  size="mini"
                  @click="addSku(row)"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="修改"
                placement="top"
              >
                <el-button
                  type="warning"
                  icon="el-icon-edit"
                  size="mini"
                  @click="updateSpu(row)"
                ></el-button>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="查看列表"
                placement="top"
              >
                <el-button
                  type="info"
                  icon="el-icon-info"
                  size="mini"
                  style="margin-right: 10px;"
                  @click="handler(row)"
                ></el-button>
              </el-tooltip>
              <!-- 需注意tooltip包裹popconfirm -->
              <el-tooltip
                class="item"
                effect="dark"
                content="删除"
                placement="top"
              >
                <el-popconfirm
                  title="确定要删除本条SPU吗？"
                  @onConfirm="deleteSpu(row)"
                >
                  <el-button
                    slot="reference"
                    type="danger"
                    size="mini"
                    icon="el-icon-delete"
                  ></el-button>
                </el-popconfirm>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>

        <el-pagination
          style="margin-top:20px;text-align:center"
          :current-page="page"
          :total="total"
          :page-size="limit"
          :pager-count="7"
          :page-sizes="[3, 5, 10]"
          layout="prev, pager, next, jumper,->,sizes,total"
          @current-change="getSpuList"
          @size-change="handleSizeChange"
        ></el-pagination>
      </div>

      <SpuForm
        v-show="scene == 1"
        ref="spu"
        @changeScene="changeScene"
      ></SpuForm>

      <SkuForm
        v-show="scene == 2"
        ref="sku"
        @changeScenes="changeScenes"
      ></SkuForm>
    </el-card>

    <el-dialog
      :title="`${spu.spuName}的sku列表`"
      :visible.sync="dialogTableVisible"
      :before-close="close"
    >
      <el-table
        v-loading="loading"
        :data="skuList"
        :header-cell-style="{textAlign: 'center'}"
        :cell-style="{ textAlign: 'center' }"
        style="width: 100%"
        border
      >
        <el-table-column
          property="skuName"
          label="名称"
        ></el-table-column>
        <el-table-column
          property="price"
          label="价格"
        ></el-table-column>
        <el-table-column
          property="weight"
          label="重量"
        ></el-table-column>
        <el-table-column label="默认图片" width="width">
          <template slot-scope="{ row }">
            <img
              :src="row.skuDefaultImg"
              alt=""
              style="width: 100px; height: 100px"
            >
          </template>
        </el-table-column>
      </el-table>
    </el-dialog>
  </div>
</template>

<script>
// 引入子组件
import SpuForm from './SpuForm'
import SkuForm from './SkuForm'
export default {
  name: 'TradeMark',
  components: { SpuForm, SkuForm },
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      page: 1, // 页码
      limit: 3, // 展示条数
      records: [], // spu列表的数据
      total: 0, // 总条数
      scene: 0, // 0展示SPU列表数据  1添加SPU|修改SPU  2添加SKU
      dialogTableVisible: false, // 控制对话框的显示与隐藏
      spu: {},
      skuList: [], // SKU列表数据
      loading: true
    }
  },
  methods: {
    // 三级联动的自定义事件
    getCategoryId({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId
        // 清除2、3级分类的id
        this.category2Id = ''
        this.category3Id = ''
      } else if (level === 2) {
        this.category2Id = categoryId
        // 清除3级id
        this.category3Id = ''
      } else {
        this.category3Id = categoryId
        // 获取SPU列表数据
        this.getSpuList()
      }
    },

    // 获取SPU列表数据
    async getSpuList(pages = 1) {
      this.page = pages
      const { page, limit, category3Id } = this
      const result = await this.$API.spu.reqSpuList(page, limit, category3Id)
      if (result.code === 200) {
        this.total = result.data.total
        this.records = result.data.records
      }
    },

    // 分页器的条数变化
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit
      // 再发请求
      this.getSpuList()
    },

    // 添加SPU按钮的回调
    addSpu() {
      // 切换为场景为1
      this.scene = 1
      // 通知子组件SpuForm发请求---两个
      this.$refs.spu.addSpuData(this.category3Id)
    },

    // 修改某一个SPU
    updateSpu(row) {
      this.scene = 1
      // 父组件执行子组件方法
      this.$refs.spu.initSpuData(row)
    },

    // 自定义事件回调（SpuForm）
    changeScene({ scene, flag }) {
      // flag,区分保存按钮是添加还是修改
      // 切换结构（场景）
      this.scene = scene
      // 子组件通知父组件切换场景，需要再次获取SPU列表的数据进行展示
      if (flag === '修改') {
        this.getSpuList(this.page)
      } else {
        this.getSpuList()
      }
    },

    // 删除SPU
    async deleteSpu(row) {
      const result = await this.$API.spu.reqDeleteSpu(row.id)
      if (result.code === 200) {
        this.$message({
          type: 'success',
          message: '删除成功'
        })
        // SPU个数大于1删除的时候停留在当前页，如果SPU个数小于1 回到上一页
        this.getSpuList(this.records.length > 1 ? this.page : this.page - 1)
      }
    },

    // 添加SKU按钮
    addSku(row) {
      // 切换场景为2
      this.scene = 2
      // 父组件调用子组件的方法，让子组件发请求------三个请求
      this.$refs.sku.getData(this.category1Id, this.category2Id, row)
    },

    // skuform通知父组件修改场景
    changeScenes(scene) {
      this.scene = scene
    },

    // 查看SKU的按钮的回调
    async handler(spu) {
      // 点击这个按钮的时候，对话框可见的
      this.dialogTableVisible = true
      // 保存spu信息
      this.spu = spu
      // 获取sku列表的数据进行展示
      const result = await this.$API.spu.reqSkuList(spu.id)
      if (result.code === 200) {
        this.skuList = result.data
        // loading隐藏
        this.loading = false
      }
    },

    // 关闭对话框的回调
    close(done) {
      // loading属性再次变为真
      this.loading = true
      // 清除sku列表的数据
      this.skuList = []
      // 管理对话框
      done()
    }
  }
}
</script>

<style scoped lang='less'>

</style>
