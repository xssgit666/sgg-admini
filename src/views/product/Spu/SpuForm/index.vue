<template>
  <div>
    <el-form
      ref="form"
      :model="spu"
      label-width="80px"
    >
      <el-form-item label="SPU名称">
        <el-input
          v-model="spu.spuName"
          placeholder="请输入SPU名称"
        ></el-input>
      </el-form-item>

      <el-form-item label="品牌">
        <el-select
          v-model="spu.tmId"
          placeholder="请选择品牌"
        >
          <el-option
            v-for="tm in tradeMarkList"
            :key="tm.id"
            :label="tm.tmName"
            :value="tm.id"
          ></el-option>
        </el-select>
      </el-form-item>

      <el-form-item label="SPU描述">
        <el-input
          v-model="spu.description"
          type="textarea"
          :rows="4"
          placeholder="请输入SPU描述"
        ></el-input>
      </el-form-item>

      <el-form-item label="SPU图片">
        <el-upload
          action="/dev-api/admin/product/fileUpload"
          list-type="picture-card"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handlerSuccess"
          :file-list="spuImageList"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
          <img width="100%" :src="dialogImageUrl" alt="">
        </el-dialog>
      </el-form-item>

      <el-form-item label="销售属性">
        <el-select
          v-model="attrIdAndAttrName"
          :placeholder="`还有${unSelectSaleAttr.length}项未选择`"
        >
          <el-option
            v-for="unselect in unSelectSaleAttr"
            :key="unselect.id"
            :label="unselect.name"
            :value="`${unselect.id}:${unselect.name}`"
          ></el-option>
        </el-select>
        <el-button
          type="primary"
          icon="el-icon-plus"
          style="margin-left: 10px"
          :disabled="!attrIdAndAttrName"
          @click="addSaleAttr"
        >添加销售属性</el-button>
        <!-- 表格:展示平台属性 -->
        <el-table
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          style="width: 100%;margin-top: 10px;"
          border
          :data="spu.spuSaleAttrList"
        >
          <el-table-column
            type="index"
            label="序号"
            width="80"
          ></el-table-column>

          <el-table-column
            prop="saleAttrName"
            label="属性名"
          ></el-table-column>

          <el-table-column
            label="属性值名称列表"
          >
            <template slot-scope="{ row }">
              <el-tag
                v-for="(tag, index) in row.spuSaleAttrValueList"
                :key="tag.id"
                closable
                :disable-transitions="false"
                @close="row.spuSaleAttrValueList.splice(index, 1)"
              >
                {{ tag.saleAttrValueName }}
              </el-tag>
              <el-input
                v-if="row.inputVisible"
                ref="saveTagInput"
                v-model="row.inputValue"
                class="input-new-tag"
                size="small"
                @keyup.enter.native="handleInputConfirm(row)"
                @blur="handleInputConfirm(row)"
              >
              </el-input>
              <el-button
                v-else
                class="button-new-tag"
                size="small"
                @click="addSaleAttrValue(row)"
              >+ 添加</el-button>
            </template>
          </el-table-column>

          <el-table-column
            label="操作"
          >
            <template slot-scope="{ $index }">
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
                @click="spu.spuSaleAttrList.splice($index, 1)"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>

      <el-form-item>
        <el-button
          type="primary"
          @click="addOrUpdateSpu"
        >保存</el-button>
        <el-button @click="cancel">取消</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
export default {
  name: 'SpuForm',
  data() {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      spu: {
        category3Id: 0, // 三级分类的id
        description: '', // 描述
        spuName: '', // spu名称
        tmId: '', // 品牌的id
        spuImageList: [], // 收集SPU图片的信息
        spuSaleAttrList: [] // 品牌属性与属性值收集
      },
      tradeMarkList: [], // 存储品牌信息
      spuImageList: [], // 存储SPU图片的数据
      saleAttrList: [], // 销售属性的数据（项目全部的销售属性）
      attrIdAndAttrName: '' // 收集未选择的销售属性的id
    }
  },
  computed: {
    // 计算出还未选择的销售属性
    unSelectSaleAttr() {
      // 整个平台的销售属性一共三个：尺寸、颜色、版本
      // saleAttrList每个元素拿出来与spuSaleAttrList比较，满足every条件true的item会被返回出来
      const result = this.saleAttrList.filter((item) => {
        // every数组的方法，会返回一个布尔值【真，假的】
        return this.spu.spuSaleAttrList.every((item1) => {
          // 真正的判断条件
          return item.name !== item1.saleAttrName
        })
      })
      return result
    }
  },
  methods: {
    // 初始化SpuForm数据
    async initSpuData(spu) {
      // 获取SPU信息的数据
      const spuResult = await this.$API.spu.reqSpu(spu.id)
      if (spuResult.code === 200) {
        this.spu = spuResult.data
      }

      // 获取品牌的信息
      const tradeMarkResult = await this.$API.spu.reqTradeMarkList()
      if (tradeMarkResult.code === 200) {
        this.tradeMarkList = tradeMarkResult.data
      }

      // 获取spu图片的数据
      const spuImageResult = await this.$API.spu.reqSpuImageList(spu.id)
      if (spuImageResult.code === 200) {
        const listArr = spuImageResult.data
        // 照片墙显示图片的数据需要是数组格式，且数组里的元素要有name与url字段，把服务器返回的数据进行修改
        listArr.forEach((item) => {
          item.name = item.imgName
          item.url = item.imgUrl
        })
        // 把整理好的数据赋值给
        this.spuImageList = listArr
      }

      // 获取平台全部的销售属性
      const saleResult = await this.$API.spu.reqBaseSaleAttrList()
      if (saleResult.code === 200) {
        this.saleAttrList = saleResult.data
      }
    },

    // 照片墙-文件列表移除文件时的钩子
    handleRemove(file, fileList) {
      // file参数:代表的是删除的那个张图片
      // fileList:照片墙删除某一张图片以后，剩余的其他的图片
      this.spuImageList = fileList
    },
    // 照片墙-点击文件列表中已上传的文件时的钩子
    handlePictureCardPreview(file) {
      // 将图片地址赋值给这个属性
      this.dialogImageUrl = file.url
      // 对话框显示
      this.dialogVisible = true
    },
    // 照片墙-文件上传成功时的钩子
    handlerSuccess(response, file, fileList) {
      // 收集图片的信息
      this.spuImageList = fileList
    },

    // 添加新的销售属性
    addSaleAttr() {
      // 把收集到的销售属性数据进行分割
      const [baseSaleAttrId, saleAttrName] = this.attrIdAndAttrName.split(':')
      // 向SPU对象的spuSaleAttrList属性里面添加新的销售属性
      const newSaleAttr = {
        baseSaleAttrId,
        saleAttrName,
        spuSaleAttrValueList: []
      }
      // 添加新的销售属性
      this.spu.spuSaleAttrList.push(newSaleAttr)
      // 清空数据
      this.attrIdAndAttrName = ''
    },

    // 添加按钮的回调
    addSaleAttrValue(row) {
      // 点击销售属性值当中添加按钮的时候，需要有button变为input,通过当前销售属性的inputVisible控制
      // 挂载在销售属性身上的响应式数据inputVisible，控制button与input切换
      this.$set(row, 'inputVisible', true)
      // 通过响应式数据inputValue字段收集新增的销售属性值
      this.$set(row, 'inputValue', '')
      // 自动获取焦点
      this.$nextTick(() => {
        this.$refs.saveTagInput.focus()
      })
    },

    // el-input失却焦点
    handleInputConfirm(row) {
      // 解构出销售属性当中收集数据
      const { baseSaleAttrId, inputValue } = row
      // 新增的销售属性值为空即为取消添加
      if (inputValue.trim() === '') {
        row.inputVisible = false
        return
      }
      // 属性值不能重复,这里也可以用some
      const result = row.spuSaleAttrValueList.every(
        (item) => item.saleAttrValueName !== inputValue
      )
      if (!result) {
        this.$message({
          message: '警告哦，列表中已有该属性值！',
          type: 'warning'
        })
        // 获取焦点
        this.$nextTick(() => {
          this.$refs.saveTagInput.focus()
        })
        return
      }
      // 新增的销售属性值
      const newSaleAttrValue = {
        baseSaleAttrId,
        saleAttrValueName: inputValue
      }
      // 新增
      row.spuSaleAttrValueList.push(newSaleAttrValue)
      // 修改inputVisible为false，不就显示button
      row.inputVisible = false
    },

    // 保存按钮
    async addOrUpdateSpu() {
      // 整理参数：需要整理照片墙的数据
      // 携带参数：对于图片，需要携带imageName与imageUrl字段
      this.spu.spuImageList = this.spuImageList.map((item) => {
        return {
          imageName: item.name,
          // 判断item.response然后使用item.response.data
          imageUrl: (item.response && item.response.data) || item.url
        }
      })
      // 发请求
      const result = await this.$API.spu.reqAddOrUpdateSpu(this.spu)
      if (result.code === 200) {
        // 提示
        this.$message({
          type: 'success',
          message: '保存成功'
        })
        // 通知父组件回到场景0
        this.$emit('changeScene', {
          scene: 0,
          flag: this.spu.id ? '修改' : '添加'
        })
      }
      // 清除数据
      Object.assign(this._data, this.$options.data())
    },

    // 点击添加SPU按钮的时候，发请求的函数
    async addSpuData(category3Id) {
      // 收集三级分类id
      this.spu.category3Id = category3Id
      // 获取品牌的信息
      const tradeMarkResult = await this.$API.spu.reqTradeMarkList()
      if (tradeMarkResult.code === 200) {
        this.tradeMarkList = tradeMarkResult.data
      }
      // 获取平台全部的销售属性
      const saleResult = await this.$API.spu.reqBaseSaleAttrList()
      if (saleResult.code === 200) {
        this.saleAttrList = saleResult.data
      }
    },

    // 取消按钮
    cancel() {
      // 取消按钮的回调，通知父亲切换场景为0
      this.$emit('changeScene', { scene: 0, flag: '' })
      // 清理数据
      // 组件实例this._data：data当中的响应式数据
      // this.$options：可以获取配置对象配置对象的data函数执行，data对象返回的响应式数据都是空的
      // Object.assign(es6中新增的方法可以合并对象。目标项a, 用来替换的内容b)，即用b替换a，即完成清空功能
      Object.assign(this._data, this.$options.data())
    }
  }
}
</script>

<style>
  .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }
</style>
