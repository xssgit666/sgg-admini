<template>
  <div>
    <el-form
      ref="ruleForm"
      :rules="rules"
      :model="skuInfo"
      label-width="80px"
    >
      <el-form-item label="SPU名称">
        {{ spu.spuName }}
      </el-form-item>
      <el-form-item label="SKU名称" prop="skuName">
        <el-input
          v-model="skuInfo.skuName"
          placeholder="sku名称"
        ></el-input>
      </el-form-item>
      <el-form-item label="价格(元)" prop="price">
        <el-input
          v-model="skuInfo.price"
          placeholder="请输入价格(元)"
          type="number"
        ></el-input>
      </el-form-item>
      <el-form-item label="重量(kg)" prop="weight">
        <el-input
          v-model="skuInfo.weight"
          placeholder="请输入重量(kg)"
          type="number"
        ></el-input>
      </el-form-item>
      <el-form-item label="规格描述">
        <el-input
          v-model="skuInfo.skuDesc"
          type="textarea"
          rows="4"
        ></el-input>
      </el-form-item>
      <el-form-item label="平台属性">
        <el-form
          ref="form"
          :inline="true"
          label-width="80px"
        >
          <el-form-item
            v-for="attr in attrInfoList"
            :key="attr.id"
            :label="attr.attrName"
          >
            <el-select
              v-model="attr.attrIdAndValueId"
              placeholder="请选择"
            >
              <el-option
                v-for="attrValue in attr.attrValueList"
                :key="attrValue.id"
                :label="attrValue.valueName"
                :value="`${attr.id}:${attrValue.id}`"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="销售属性">
        <el-form
          ref="form"
          :inline="true"
          label-width="80px"
        >
          <el-form-item
            v-for="saleAttr in spuSaleAttrList"
            :key="saleAttr.id"
            :label="saleAttr.saleAttrName"
          >
            <el-select
              v-model="saleAttr.attrIdAndValueId"
              placeholder="请选择"
            >
              <el-option
                v-for="saleAttrValue in saleAttr.spuSaleAttrValueList"
                :key="saleAttrValue.id"
                :label="saleAttrValue.saleAttrValueName"
                :value="`${saleAttr.id}:${saleAttrValue.id}`"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </el-form-item>
      <el-form-item label="图片列表">
        <el-table
          style="width: 100%"
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          border
          :data="spuImageList"
          @selection-change="handleSelectionChange"
        >
          <el-table-column
            type="selection"
            prop="prop"
            width="80"
          >
          </el-table-column>
          <el-table-column
            prop="prop"
            label="图片"
            width="width"
          >
            <template slot-scope="{row}">
              <img
                :src="row.imgUrl"
                style="width:100px;height:100px"
              >
            </template>
          </el-table-column>
          <el-table-column
            prop="imgName"
            label="名称"
            width="width"
          >
          </el-table-column>
          <el-table-column
            prop="prop"
            label="操作"
            width="width"
          >
            <template slot-scope="{row}">
              <el-button
                v-if="row.isDefault == 0"
                type="primary"
                @click="changeDefault(row)"
              >设置默认</el-button>
              <el-button v-else>默认</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          @click="save"
        >保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>

    <el-backtop></el-backtop>
  </div>
</template>

<script>
export default {
  name: 'SkuForm',
  data() {
    return {
      spuImageList: [], // 存储图片的信息
      spuSaleAttrList: [], // 存储销售是属性
      attrInfoList: [], // 存储平台属性的数据
      skuInfo: { // 收集sku数据的字段
        // 第一类收集的数据：父组件给的数据
        category3Id: 0,
        spuId: 0,
        tmId: 0,
        // 第二类：需要通过数据双向绑定v-model收集
        skuName: '',
        price: '',
        weight: '',
        skuDesc: '',
        // 第三类：需要自己书写代码
        skuDefaultImg: '', // 默认图片
        skuImageList: [], // 收集图片的字段
        skuAttrValueList: [], // 平台属性
        skuSaleAttrValueList: [] // 销售属性
      },
      spu: {},
      imageList: [], // 收集图片的数据字段
      rules: { // 表单验证
        skuName: [
          { required: true, message: '请输入SKU名称', trigger: 'blur' }
        ],
        price: [
          { required: true, message: '请输入价格', trigger: 'blur' }
        ],
        weight: [
          { required: true, message: '请输入重量', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 获取SkuForm数据
    async getData(category1Id, category2Id, spu) {
      // 收集父组件给予的数据
      this.skuInfo.category3Id = spu.category3Id
      this.skuInfo.spuId = spu.id
      this.skuInfo.tmId = spu.tmId
      this.spu = spu
      // 获取图片的数据
      const result = await this.$API.spu.reqSpuImageLIst(spu.id)
      if (result.code === 200) {
        const list = result.data
        list.forEach(item => {
          item.isDefault = 0
        })
        this.spuImageList = list
      }
      // 获取销售属性的数据
      const result1 = await this.$API.spu.reqSpuSaleAttrList(spu.id)
      if (result1.code === 200) {
        this.spuSaleAttrList = result1.data
      }
      // 获取平台属性的数据
      const result2 = await this.$API.spu.reqAttrInfoList(
        category1Id,
        category2Id,
        spu.category3Id
      )
      if (result2.code === 200) {
        this.attrInfoList = result2.data
      }
    },

    // table表格复选框按钮
    handleSelectionChange(params) {
      // 获取到用户选中图片的信息数据，缺少isDefault字段
      this.imageList = params
    },

    // 设置默认操作
    changeDefault(row) {
      // 图片列表的数据的isDefault字段全部变为 0
      this.spuImageList.forEach(item => {
        item.isDefault = 0
      })
      // 点击的那个图片的数据变为1
      row.isDefault = 1
      // 收集默认图片的地址
      this.skuInfo.skuDefaultImg = row.imgUrl
    },

    // 取消按钮
    cancel() {
      // 自定义事件，让父组件切换场景0
      this.$emit('changeScenes', 0)
      // 清除数据
      Object.assign(this._data, this.$options.data())
    },

    // 保存按钮的事件
    save() {
      // 表单验证
      this.$refs.ruleForm.validate((valid) => {
        if (valid) {
          this.submit()
        } else {
          this.$message({
            message: '警告哦，先填完必填项目才能保存！',
            type: 'warning'
          })
          return false
        }
      })
    },
    // 验证通过
    async submit() {
      // 解构出后续要用到的数据
      const { attrInfoList, skuInfo, spuSaleAttrList, imageList } = this

      // 整理平台属的数据
      skuInfo.skuAttrValueList = attrInfoList.reduce((prev, item) => {
        if (item.attrIdAndValueId) {
          const [attrId, valueId] = item.attrIdAndValueId.split(':')
          prev.push({ attrId, valueId })
        }
        return prev
      }, [])

      // 整理销售属性
      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((prev, item) => {
        if (item.attrIdAndValueId) {
          const [saleAttrId, saleAttrValueId] = item.attrIdAndValueId.split(':')
          prev.push({ saleAttrId, saleAttrValueId })
        }
        return prev
      }, [])

      // 整理图片的数据
      skuInfo.skuImageList = imageList.map(item => {
        return {
          imgName: item.imgName,
          imgUrl: item.imgUrl,
          isDefault: item.isDefault,
          spuImgId: item.id
        }
      })

      // 发请求
      const result = await this.$API.spu.reqAddSku(skuInfo)
      if (result.code === 200) {
        this.$message({
          type: 'success',
          message: '添加SKU成功'
        })
        this.$emit('changeScenes', 0)
      }

      // 清除数据
      Object.assign(this._data, this.$options.data())
    }
  }
}
</script>

<style scoped lang='less'>

</style>
