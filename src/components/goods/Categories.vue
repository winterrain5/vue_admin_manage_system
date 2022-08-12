<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialoge"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <zk-table
        :data="catesList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-index="true"
        index-text="#"
        :border="true"
        :stripe="true"
        :show-row-hover="false"
      >
        <template slot="isOk" slot-scope="scope">
          <i v-if="scope.row.cat_deleted === false" class="el-icon-success"></i>
          <i v-if="scope.row.cat_deleted === true" class="el-icon-error"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level == 0" type="primary" size="mini"
            >一级</el-tag
          >
          <el-tag v-if="scope.row.cat_level == 1" type="success" size="mini"
            >二级</el-tag
          >
          <el-tag v-if="scope.row.cat_level == 2" type="warning" size="mini"
            >三级</el-tag
          >
        </template>
        <template slot="operation" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            @click="deleteCate(scope.row)"
            >删除</el-button
          >
        </template>
      </zk-table>

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisiable"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="catname">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            expand-trigger="hover"
            :options="parentCatList"
            :props="addCateProps"
            v-model="selectCateKeys"
            :clearable="true"
            :change-on-select="true"
            @change="parentCateChange"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisiable = false">取 消</el-button>
        <el-button type="primary" @click="addCategories">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  created() {
    this.getCateList()
  },
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      catesList: [],
      parentCatList: [],
      total: 0,
      //   为table指定列表的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          prop: 'cat_deleted',
          type: 'template',
          template: 'isOk'
        },
        {
          label: '排序',
          prop: 'cat_level',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'operation'
        }
      ],
      //   控制添加分类的对话的显示与隐藏
      addCateDialogVisiable: false,
      addCateForm: {
        cat_name: '',
        // 父级分类ID
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        catname: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      addCateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      selectCateKeys: []
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.message)
      this.catesList = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听页码值改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCateDialoge() {
      this.getParentCateList()
      this.addCateDialogVisiable = true
    },
    async addCategories() {
      const { data: res } = await this.$http.post(
        'categories',
        this.addCateForm
      )
      if (res.meta.status !== 201) return this.$message.error(res.meta.message)
      this.$message.success('添加成功')
      this.getCateList()
      this.addCateDialogVisiable = false
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.message)
      this.parentCatList = res.data
    },
    // 级联选择器选项变化事件
    parentCateChange() {
      // 选择了父级ID
      if (this.selectCateKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectCateKeys[
          this.selectCateKeys.length - 1
        ]
        this.addCateForm.cat_level = this.selectCateKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCateDialogClosed() {
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
      this.selectCateKeys = []
      this.$refs.addCateFormRef.resetFields()
    },
    async deleteCate(cateInfo) {
      const result = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).catch(err => err)

      if (result === 'cancel') {
        return
      }

      const { data:res } = await this.$http.delete('categories/' + cateInfo.cat_id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.message)
      this.$message.success('删除成功')
      this.getCateList()
    }
  }
}
</script>
<style lang="less" scoped>
.el-icon-success {
  color: green;
}
.el-icon-error {
  color: red;
}
.zk-table {
  margin-top: 15px;
  margin-bottom: 15px;
}
</style>
