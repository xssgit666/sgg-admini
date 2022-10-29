<template>
  <div>
    <!-- 按钮 -->
    <el-button
      type="primary"
      icon="el-icon-plus"
      style="margin: 0 0 20px"
      @click="showDialog"
    >添加</el-button>

    <!-- 表格 -->
    <el-table
      :header-cell-style="{textAlign: 'center'}"
      :cell-style="{ textAlign: 'center' }"
      style="width: 100%"
      border
      :data="list"
    >
      <el-table-column
        type="index"
        label="序号"
        width="80px"
      ></el-table-column>

      <el-table-column
        prop="tmName"
        label="品牌名称"
      ></el-table-column>

      <el-table-column
        prop="logoUrl"
        label="品牌LOGO"
      >
        <template slot-scope="{ row }">
          <el-image
            :src="row.logoUrl"
            class="cover"
            fit="contain"
          >
            <div slot="error">
              <i class="el-icon-picture-outline"></i>
            </div>
          </el-image>
        </template>
      </el-table-column>

      <el-table-column
        prop="prop"
        label="操作"
      >
        <template slot-scope="{ row }">
          <el-button
            type="warning"
            icon="el-icon-edit"
            size="mini"
            @click="updateTradeMark(row)"
          >修改</el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteTradeMark(row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页器 -->
    <el-pagination
      style="margin-top:20px;text-align:center"
      :current-page="page"
      :total="total"
      :page-size="limit"
      :pager-count="7"
      :page-sizes="[3, 5, 10]"
      layout="prev, pager, next, jumper,->,sizes,total"
      @current-change="getPageList"
      @size-change="handleSizeChange"
    >
    </el-pagination>

    <!-- 对话框 -->
    <el-dialog
      :title="tmForm.id ? '修改品牌' : '添加品牌'"
      :visible.sync="dialogFormVisible"
    >
      <el-form
        ref="ruleForm"
        :model="tmForm"
        :rules="rules"
      >
        <el-form-item
          label="品牌名称"
          :label-width="formLabelWidth"
          prop="tmName"
        >
          <el-input
            v-model="tmForm.tmName"
            autocomplete="off"
          ></el-input>
        </el-form-item>
        <el-form-item
          label="品牌LOGO"
          :label-width="formLabelWidth"
          prop="logoUrl"
        >
          <!-- 上传地址需加前缀dev-api才会被服务器转发 -->
          <el-upload
            class="avatar-uploader"
            action="/dev-api/admin/product/fileUpload"
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload"
          >
            <img
              v-if="tmForm.logoUrl"
              :src="tmForm.logoUrl"
              class="avatar"
            >
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdateTradeMark">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'TradeMark',
  data() {
    return {
      page: 1, // 分页器页码
      limit: 5, // 展示数据条数
      total: 0, // 总条数
      list: [], // 列表展示的数据
      dialogFormVisible: false, // 对话框显示与隐藏
      tmForm: {
        tmName: '', // 品牌名
        logoUrl: '' // logo地址
      },
      formLabelWidth: '100px', // 弹框内标题宽
      rules: {
        // 品牌名称的验证规则
        tmName: [
          {
            required: true,
            message: '请输入品牌名称',
            trigger: 'blur'
          },
          { min: 2, max: 10, message: '品牌名称2-10位', trigger: 'blur' }
        ],
        // 品牌的logo验证规则
        logoUrl: [
          {
            required: true,
            message: '请上传品牌的图片'
          }
        ]
      }
    }
  },
  mounted() {
    this.getPageList()
  },
  methods: {
    // 获取品牌列表的数据
    async getPageList(pager = 1) {
      this.page = pager
      const { page, limit } = this // 解构出参数
      const result = await this.$API.trademark.reqTradeMarkList(page, limit)
      if (result.code === 200) {
        this.total = result.data.total
        this.list = result.data.records
      }
    },
    // 展示数据条数发生变化时
    handleSizeChange(limit) {
      this.limit = limit
      this.getPageList()
    },
    // 点击添加品牌的按钮
    showDialog() {
      // 显示对话框
      this.dialogFormVisible = true
      // 清空上一次数据
      this.tmForm = { tmName: '', logoUrl: '' }
    },
    // 修改某一个品牌
    updateTradeMark(row) {
      // 显示对话框
      this.dialogFormVisible = true
      // 不能直接修改tmForm，因为如果修改后取消了，但是页面已经展示出来了。因此，使用浅拷贝
      this.tmForm = { ...row }
    },
    // 图片上传成功
    handleAvatarSuccess(res, file) {
      // res：上传成功之后返回前端数据
      this.tmForm.logoUrl = res.data
    },
    // 图片上传之前
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg' // 判断格式
      const isLt2M = file.size / 1024 / 1024 < 2 // 判断大小2M

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }
      return isJPG && isLt2M
    },
    // 弹框-确定按钮（添加品牌|修改品牌）
    addOrUpdateTradeMark() {
      // 当全部验证字段通过
      this.$refs.ruleForm.validate(async(valid) => {
        if (valid) {
          this.dialogFormVisible = false // 关闭弹框
          // 发送网络请求
          const result = await this.$API.trademark.reqAddOrUpdateTradeMark(this.tmForm)
          if (result.code === 200) {
            // 弹出信息:添加品牌成功、修改品牌成功
            this.$message({
              type: 'success',
              message: this.tmForm.id ? '修改品牌成功' : '添加品牌成功'
            })
            // 如果添加品牌： 停留在第一页，修改品牌应该留在当前页面
            this.getPageList(this.tmForm.id ? this.page : 1)
          }
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    // 删除品牌的操作
    deleteTradeMark(row) {
      this.$confirm(`您确定删除品牌“${row.tmName}”吗?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async() => {
        // 发送网络请求
        const result = await this.$API.trademark.reqDeleteTradeMark(row.id)
        if (result.code === 200) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
        // 刷新页面
        this.getPageList(this.list.length > 1 ? this.page : this.page - 1)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style scoped lang="scss">
// 表格样式
.cover{
  width: 80px;
  height: 80px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  border-radius: 8px;
  margin: 0 auto;
  border: 1px solid #DCDFE6;
  background-color: #F5F7FA;
  .el-icon-picture-outline{
    font-size: 24px;
    display: block;
  }
}
</style>
<style>
/* 弹框样式 */
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>
