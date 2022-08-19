<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item to="/home">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/goods">订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input
            placeholder="请输入订单编号"
            v-model="queryInfo.query"
            clearable
            @clear="getOrdersList()"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrdersList()"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <el-table :data="ordersList" :border="true" :stripe="true">
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column
          label="订单价格"
          prop="order_price"
          width="120"
        ></el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="120">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.pay_status === '0'"
              >未付款</el-tag
            >
            <el-tag type="primary" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column
          label="是否发货"
          prop="is_send"
          width="120"
        ></el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="180">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template>
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditAddressDialog"
            ></el-button>
            <el-button
              type="success"
              icon="el-icon-location"
              size="mini"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[2, 5, 10, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>
    <el-dialog
      title="修改地址"
      :visible.sync="editAddressDialogVisiable"
      width="30%"
      @close="addressClosed"
    >
      <el-form
        :model="editAddressForm"
        :rules="editAddressFormRules"
        ref="editAddressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区：" prop="province">
          <el-cascader
            :options="cityData"
            :props="selectCityProps"
            v-model="editAddressForm.province"
            :clearable="true"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址：" prop="address">
          <el-input v-model="editAddressForm.address"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editAddressDialogVisiable = false">取 消</el-button>
        <el-button type="primary">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流进度对话框 -->
    <el-dialog
      title="物流进度"
      :visible.sync="progressDialogVisiable"
      width="30%"
    >
      <el-timeline>
        <el-timeline-item
          v-for="(activity, index) in activities"
          :key="index"
          :icon="activity.icon"
          :type="activity.type"
          :color="activity.color"
          :size="activity.size"
          :timestamp="activity.timestamp"
        >
          {{ activity.content }}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  created() {
    this.getOrdersList()
  },
  data() {
    return {
      selectCityProps: {
        expandTrigger: 'hover',
        value: 'value',
        label: 'label',
        children: 'children'
      },
      editAddressDialogVisiable: false,
      editAddressForm: {
        address: '',
        province: []
      },
      editAddressFormRules: {
        province: [
          { required: true, message: '请选择省市区', trigger: 'blur' }
        ],
        address: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressDialogVisiable: false,
      queryInfo: {
        query: '',
        pagesize: 10,
        pagenum: 1
      },
      ordersList: [],
      total: 0,
      activities: [
        {
          content: '支持使用图标',
          timestamp: '2018-04-12 20:46',
          size: 'large',
          type: 'primary',
          icon: 'el-icon-more'
        },
        {
          content: '支持自定义颜色',
          timestamp: '2018-04-03 20:46',
          color: '#0bbd87'
        },
        {
          content: '支持自定义尺寸',
          timestamp: '2018-04-03 20:46',
          size: 'large'
        },
        {
          content: '默认样式的节点',
          timestamp: '2018-04-03 20:46'
        }
      ]
    }
  },
  methods: {
    async getOrdersList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.ordersList = res.data.goods
      this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrdersList()
    },
    // 监听页码值改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrdersList()
    },
    showEditAddressDialog() {
      this.editAddressDialogVisiable = true
    },
    addressClosed() {
      this.$refs.editAddressFormRef.resetFields()
    },
    async showProgressBox() {
      this.progressDialogVisiable = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
