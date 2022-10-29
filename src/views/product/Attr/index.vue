<template>
  <div>
    <el-card style="margin: 0 0 20px">
      <CategorySelect
        :show="!isShowTable"
        @getCategoryId="getCategoryId"
      ></CategorySelect>
    </el-card>
    <el-card>
      <!-- 表格:展示平台属性 -->
      <div v-show="isShowTable">
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!category3Id"
          style="margin-bottom: 20px"
          @click="addAttr"
        >添加属性</el-button>

        <!-- 表格:展示平台属性 -->
        <el-table
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          style="width: 100%"
          border
          :data="attrList"
        >
          <el-table-column
            type="index"
            label="序号"
            width="80"
          ></el-table-column>

          <el-table-column
            prop="attrName"
            label="属性名称"
          ></el-table-column>

          <el-table-column
            prop="attrName"
            label="属性值列表"
          >
            <template slot-scope="{ row }">
              <el-tag
                v-for="tag in row.attrValueList"
                :key="tag.id"
                style="margin:0 5px"
                size="mini"
              >{{ tag.valueName }}</el-tag>
            </template>
          </el-table-column>

          <el-table-column
            label="操作"
          >
            <template slot-scope="{ row }">
              <el-button
                type="warning"
                icon="el-icon-edit"
                size="mini"
                @click="updateAttr(row)"
              >修改</el-button>
              <el-button
                type="danger"
                icon="el-icon-delete"
                size="mini"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <!-- 添加属性|修改属性的结构 -->
      <div v-show="!isShowTable">
        <!-- 请输入属性名 -->
        <el-form
          ref="form"
          :inline="true"
          :model="attrInfo"
        >
          <el-form-item label="属性名">
            <el-input
              v-model="attrInfo.attrName"
              placeholder="请输入属性名"
            ></el-input>
          </el-form-item>
        </el-form>

        <!-- 添加属性值 -->
        <el-button
          type="primary"
          icon="el-icon-plus"
          :disabled="!attrInfo.attrName"
          @click="addAttrValue"
        >添加属性值</el-button>
        <el-button @click="isShowTable=true">取消</el-button>

        <!-- 表格 -->
        <el-table
          :header-cell-style="{textAlign: 'center'}"
          :cell-style="{ textAlign: 'center' }"
          :data="attrInfo.attrValueList"
          style="width: 100%;margin: 20px 0"
          border
        >
          <el-table-column
            type="index"
            label="序号"
            width="80px"
          ></el-table-column>

          <el-table-column
            prop="tmName"
            label="属性值名称"
          >
            <template slot-scope="{ row, $index }">
              <el-input
                v-if="row.flag"
                :ref="$index"
                v-model="row.valueName"
                placeholder="请输入属性值名称"
                @blur="toLook(row)"
                @keyup.native.enter="toLook(row)"
              ></el-input>
              <span
                v-else
                style="display: block"
                @click="toEdit(row, $index)"
              >{{ row.valueName }}</span>
            </template>
          </el-table-column>

          <el-table-column
            label="操作"
            width="300px"
          >
            <template slot-scope="{ row, $index }">
              <el-popconfirm
                :title="`确定删除属性值：${row.valueName} 吗?`"
                @onConfirm="deleteAttrValue($index)"
              >
                <el-button
                  slot="reference"
                  type="danger"
                  icon="el-icon-delete"
                >删除</el-button>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>

        <el-button
          type="primary"
          :disabled="attrInfo.attrValueList.length<1"
          @click="addOrUpdateAttr"
        >保存</el-button>
        <el-button @click="isShowTable=true">取消</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
// 按需引入lodash当中的深拷贝
import cloneDeep from 'lodash/cloneDeep'
export default {
  name: 'Atrr',
  data() {
    return {
      category1Id: '',
      category2Id: '',
      category3Id: '',
      attrList: [], // 接收平台属性的数据
      isShowTable: true, // 数据表显示
      attrInfo: { // 新增属性|修改属性
        attrName: '', // 属性名
        attrValueList: [ // 属性值
          // {
          //   attrId: 0, // 属性的Id,此时还没有属性值Id
          //   valueName: ''
          // }
        ],
        categoryId: 0, // 三级分类id(data中不能获取category3Id)
        categoryLevel: 3 // 服务器区分几级id
      }
    }
  },
  methods: {
    // 自定义事件的回调
    getCategoryId({ categoryId, level }) {
      if (level === 1) {
        this.category1Id = categoryId
        this.category2Id = ''
        this.category3Id = ''
      } else if (level === 2) {
        this.category2Id = categoryId
        this.category3Id = ''
      } else {
        // 代表三级分类的id有了
        this.category3Id = categoryId
        // 发请求获取平台的属性数据
        this.getAttrList()
      }
    },
    // 获取平台属性的数据
    async getAttrList() {
      // 解构获取分类的ID
      const { category1Id, category2Id, category3Id } = this
      // 获取属性列表的数据
      const result = await this.$API.attr.reqAttrList(
        category1Id,
        category2Id,
        category3Id
      )
      if (result.code === 200) {
        this.attrList = result.data
      }
    },
    // 添加属性值
    addAttrValue() {
      // 向属性值的数组里面添加元素
      this.attrInfo.attrValueList.push({
        // 新增属性值时，需要把已有的属性的id带上
        attrId: this.attrInfo.id, // 属性的id
        // 此时还没有属性值Id
        valueName: '', // 属性名
        flag: true // 用户切换查看模式与编辑模式
      })
      this.$nextTick(() => {
        this.$refs[this.attrInfo.attrValueList.length - 1].focus()
      })
    },
    // 添加属性
    addAttr() {
      // 切换table显示与隐藏
      this.isShowTable = false
      // 清除数据
      this.attrInfo = {
        attrName: '', // 属性名
        attrValueList: [], // 属性值
        categoryId: this.category3Id, // 三级分类的id
        categoryLevel: 3 // 因为服务器也需要区分几级id
      }
    },
    // 修改某一个属性
    updateAttr(row) {
      this.isShowTable = false
      // 将选中的属性赋值给attrInfo
      // 由于数据结构当中存在对象里面套数组，数组里面有套对象，因此需要使用深拷贝解决这类问题
      this.attrInfo = cloneDeep(row)
      // 修改属性时，将相应属性值元素添加上flag标记，必须$set
      this.attrInfo.attrValueList.forEach((item) => {
        // （对象，属性名，属性值）
        this.$set(item, 'flag', false)
      })
    },
    // 失却焦点的事件---切换为查看模式，展示span
    toLook(row) {
      // 如果属性值为空,用户提示
      if (row.valueName.trim() === '') {
        this.$message.error('请你输入一个正常的属性值')
        return
      }
      // 新增的属性值不能与已有的属性值重复
      // some( )方法用于监测数组中的元素是否满足指定条件，方法会依次执行数组的每一个元素，如果有一个元素满足条件，表达式返回true，剩余的元素不会再执行监测，如果没有满足条件的元素，则返回false
      const isRepat = this.attrInfo.attrValueList.some(item => {
        // row：形参是当前用户添加的最新的属性值
        if (row !== item) { // 与非己项比较(item中包含row)
          return row.valueName === item.valueName
        }
      })
      if (isRepat) {
        this.$message.error('属性中已有该属性值')
        return
      }
      // 当前编辑模式变为查看模式【让input消失，显示span】
      row.flag = false
    },
    // 点击span的回调，变为编辑模式
    toEdit(row, index) {
      row.flag = true
      // $nextTick,当节点渲染完毕了，会执行一次
      this.$nextTick(() => {
        // 获取相应的input表单元素实现聚焦
        this.$refs[index].focus()
      })
    },
    // 删除属性值(当前删除属性值的操作是不需要发请求的)
    deleteAttrValue(index) {
      // arr.splice(index,n,add)
      // index : 开始操作的位置，数组的下标
      // n : 要删除的个数
      // add : 要添加的元素（可以是多个）
      this.attrInfo.attrValueList.splice(index, 1)
    },
    // 保存按钮：进行添加属性或者修改属性的操作
    async addOrUpdateAttr() {
      // 提交给服务器数据当中不应该出现flag字段
      // filter 将数组中不符合的元素去掉，返回一个新数组
      this.attrInfo.attrValueList = this.attrInfo.attrValueList.filter(item => {
        // 过滤掉属性值不是空的
        if (item.valueName !== '') {
          // 删除掉flag属性
          delete item.flag
          return true
        }
      })
      try {
        // 发请求
        await this.$API.attr.reqAddOrUpdateAttr(this.attrInfo)
        // 展示平台属性的信号量进行切换
        this.isShowTable = true
        // 提示消失
        this.$message({ type: 'success', message: '保存成功' })
        // 保存成功以后需要再次获取平台属性进行展示
        this.getAttrList()
      } catch (error) {
        // this.$message('保存失败，该数据无权限修改')
      }
    }
  }
}
</script>

<style scoped lang='less'>

</style>
