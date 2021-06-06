<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表数据 -->
      <el-table :data="orderlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="订单编号" prop="order_number"></el-table-column>
        <el-table-column label="订单价格" prop="order_price"></el-table-column>
        <el-table-column label="是否付款" prop="pay_status">
          <template slot-scope="scope">
            <el-tag v-if="scope.pay_status === '1'">已付款</el-tag>
            <el-tag v-else type="success">未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send">
          <template slot-scope="scope">{{ scope.row.is_send }}</template>
        </el-table-column>
        <el-table-column label="下单时间" prop="create_time">
          <template slot-scope="scope">{{
            scope.row.create_time | dateFormat
          }}</template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showBox"
            ></el-button>
            <el-button
              type="success"
              size="mini"
              icon="el-icon-location"
              @click="showProgressBox"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 地址对话框 -->
      <el-dialog
        title="修改地址"
        @close="addressDialogClosed"
        :visible.sync="addressVisible"
        width="50%"
      >
        <el-form
          :model="addressForm"
          :rules="addressFormRules"
          ref="addressFormRefs"
          label-width="100px"
        >
          <el-form-item label="省市区/县" prop="address1">
            <el-cascader
              v-model="addressForm.address1"
              :options="cityData"
            ></el-cascader>
          </el-form-item>
          <el-form-item label="详细地址" prop="address2">
            <el-input v-model="addressForm.address2"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addressVisible = false">取 消</el-button>
          <el-button type="primary" @click="addressVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
      <!-- 物流对话框 -->
      <el-dialog title="物流信息" :visible.sync="progressVisible" width="50%">
        <span>这是一段信息</span>
        <span slot="footer" class="dialog-footer">
          <el-button @click="progressVisible = false">取 消</el-button>
          <el-button type="primary" @click="progressVisible = false"
            >确 定</el-button
          >
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      total: 0,
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: '',
        address2: ''
      },
      addressFormRules: {},
      cityData,
      progressVisible: false,
      progressInfo: {}
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        this.$message.error('获取订单列表失败！')
      }
      this.total = res.data.total
      this.orderlist = res.data.goods
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 展示修改地址的对话框
    showBox() {
      this.addressVisible = true
    },
    addressDialogClosed() {
      this.$refs.addressFormRefs.resetFields()
    },
    async showProgressBox() {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流进度失败！')
      }
      this.progressInfo = res.data
      this.progressVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-cascader {
  height: 100%;
}
</style>
