<template>
  <div>
    <el-table
      v-loading="loading"
      style="width: 100%"
      :header-cell-style="{textAlign: 'center'}"
      :cell-style="{ textAlign: 'center' }"
      :data="records"
      border
    >
      <el-table-column
        type="index"
        label="序号"
        width="80"
      >
      </el-table-column>
      <el-table-column
        prop="skuName"
        label="名称"
      >
      </el-table-column>
      <el-table-column
        prop="skuDesc"
        label="描述"
      >
      </el-table-column>
      <el-table-column label="默认图片">
        <template slot-scope="{ row }">
          <el-image
            class="cover"
            :src="row.skuDefaultImg"
            fit="contain"
          >
            <div slot="error" class="image-slot">
              <i class="el-icon-picture-outline"></i>
            </div>
          </el-image>
        </template>
      </el-table-column>
      <el-table-column
        prop="weight"
        label="重量"
      >
      </el-table-column>
      <el-table-column
        prop="price"
        label="价格"
      >
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="{ row }">
          <el-tooltip
            v-if="row.isSale == 0"
            class="item"
            effect="dark"
            content="已下架，点击上架"
            placement="top"
          >
            <el-button
              type="info"
              icon="el-icon-sort-down"
              size="mini"
              @click="sale(row)"
            ></el-button>
          </el-tooltip>
          <el-tooltip
            v-else
            class="item"
            effect="dark"
            content="已上架，点击下架"
            placement="top"
          >
            <el-button
              type="success"
              icon="el-icon-sort-up"
              size="mini"
              @click="cancel(row)"
            ></el-button>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="编辑"
            placement="top"
          >
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="edit"
            ></el-button>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="查看"
            placement="top"
          >
            <el-button
              type="warning"
              icon="el-icon-info"
              size="mini"
              @click="getSkuInfo(row)"
            ></el-button>
          </el-tooltip>
          <el-tooltip
            class="item"
            effect="dark"
            content="删除"
            placement="top"
          >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
            ></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      style="margin-top:20px;text-align:center"
      :current-page="page"
      background
      :total="total"
      :page-size="limit"
      :pager-count="7"
      :page-sizes="[3, 5, 10]"
      layout="prev, pager, next, jumper,->,sizes,total"
      @size-change="handleSizeChange"
      @current-change="getSkuList"
    ></el-pagination>

    <!-- 抽屉效果 -->
    <el-drawer
      :visible.sync="show"
      size="30%"
      @closed="closed"
    >
      <el-form
        v-loading="formLoading"
        label-width="100px"
        style="padding:0 20px 0 10px"
      >
        <el-form-item label="商品名称：">
          {{ skuInfo.skuName }}
        </el-form-item>
        <el-form-item label="商品描述：">
          <span v-if="skuInfo.skuDesc">
            {{ skuInfo.skuDesc }}
          </span>
          <span
            v-else
            style="color:#909399"
          >暂未填写商品描述</span>
        </el-form-item>
        <el-form-item label="商品价格：">
          {{ skuInfo.price }} 元
        </el-form-item>
        <el-form-item label="平台属性：">
          <template v-if="arrtNum">
            <el-tag
              v-for="attr in skuInfo.skuAttrValueList"
              :key="attr.id"
              type="success"
              style="margin-right:10px"
            >{{ attr.attrId }}-{{ attr.valueId }}</el-tag>
          </template>
          <span
            v-else
            style="color:#909399"
          >暂未设置平台属性</span>
        </el-form-item>
        <el-form-item label="商品图片：">
          <el-carousel
            v-if="swiper"
            :interval="2000"
            arrow="always"
          >
            <el-carousel-item
              v-for="item in skuInfo.skuImageList"
              :key="item.id"
            >
              <el-image
                style="height: 300px;width: 100%;"
                :src="item.imgUrl"
                fit="contain"
              >
              </el-image>
            </el-carousel-item>
          </el-carousel>
          <span
            v-else
            style="color:#909399"
          >暂未上传商品图片</span>
        </el-form-item>
      </el-form>
    </el-drawer>
  </div>
</template>

<script>
export default {
  name: 'Sku',
  data() {
    return {
      page: 1, // 当前第几页
      limit: 5, // 当前页面有几条数据
      records: [], // 存储SKU列表的数据
      total: 0, // 存储分页器一共展示的数据
      skuInfo: {}, // 存储SKU信息
      show: false,
      loading: true, // 表格数据
      formLoading: true, // 抽屉表单数据
      swiper: true, // 控制抽屉轮播图显示
      arrtNum: true // 控制平台属性显示
    }
  },
  mounted() {
    // 获取sku列表的方法
    this.getSkuList()
  },
  methods: {
    // 获取sku列表数据的方法
    async getSkuList(pages = 1) {
      this.loading = true // 开启加载
      this.page = pages
      // 解构出默认的参数
      const { page, limit } = this
      const result = await this.$API.sku.reqSkuList(page, limit)
      if (result.code === 200) {
        this.total = result.data.total
        this.records = result.data.records
        this.loading = false // loading隐藏
      }
    },

    // 改变页面条数
    handleSizeChange(limit) {
      // 修改参数
      this.limit = limit
      this.getSkuList()
    },

    // 上架
    async sale(row) {
      const result = await this.$API.sku.reqSale(row.id)
      if (result.code === 200) {
        // eslint-disable-next-line require-atomic-updates
        row.isSale = 1
        this.$message({ type: 'success', message: '上架成功' })
      }
    },

    // 下架
    async cancel(row) {
      const result = await this.$API.sku.reqCancel(row.id)
      if (result.code === 200) {
        // eslint-disable-next-line require-atomic-updates
        row.isSale = 0
        this.$message({ type: 'success', message: '下架成功' })
      }
    },

    // 编辑
    edit() {
      this.$message('正在开发中')
    },

    // 获取SKU详情的方法
    async getSkuInfo(sku) {
      // 展示抽屉
      this.show = true
      // 开启加载
      this.formLoading = true
      // 获取SKU数据
      const result = await this.$API.sku.reqSkuById(sku.id)
      if (result.code === 200) {
        this.skuInfo = result.data
        this.formLoading = false // 关闭加载
      }

      // 判断抽屉轮播图是否显示
      if (this.skuInfo.skuImageList.length > 0) {
        this.swiper = true
      } else {
        this.swiper = false
      }

      // 判断平台属性显示
      if (this.skuInfo.skuAttrValueList.length > 0) {
        this.arrtNum = true
      } else {
        this.arrtNum = false
      }
    },

    // 关闭抽屉
    closed() {
      // 清空抽屉数据
      this.skuInfo = {}
    }
  }
}
</script>

<style>
  .cover{
    width: 80px;
    height: 80px;
    margin: 0 auto;
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    border-radius: 8px;
    border: 1px solid #DCDFE6;
    background-color: #F5F7FA;
  }
  .el-icon-picture-outline{
    font-size: 20px;
    display: block;
  }

  .el-carousel__item h3 {
    color: #475669;
    font-size: 14px;
    opacity: 0.75;
    line-height: 150px;
    margin: 0;
  }

  .el-carousel__button{
    width:10px;
    height:10px;
    background:red;
    border-radius:50%;
  }
</style>
