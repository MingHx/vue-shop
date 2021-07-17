/* eslint-disable vue/no-unused-vars */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getOrederList">

            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getOrederList"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 订单列表区域 -->
      <el-table :data="orderlist"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number"
                         label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price"
                         label="订单价格">
        </el-table-column>
        <el-table-column prop="pay_status"
                         label="是否付款">
          <template slot-scope="scope">
            <el-tag type="success"
                    v-if="scope.row.pay_status === '1'">已付款</el-tag>
            <el-tag type="danger"
                    v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send"
                         label="是否发货">
        </el-table-column>
        <el-table-column prop="create_time"
                         label="下单时间">
          <template slot-scope="scope">
            {{scope.row.create_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="180px">
          // eslint-disable-next-line vue/no-unused-vars
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="showBox"></el-button>
            <el-button type="success"
                       icon="el-icon-place"
                       size="mini"
                       @click="showProgressBox(scope.row.id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[5, 10, 15, 20]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>

    <!-- 修改地址的对话框 -->
    <el-dialog title="修改地址"
               :visible.sync="addressVisible"
               width="30%"
               @close="addressDialogClosed">
      <el-form :model="addressForm"
               :rules="addressRules"
               ref="addressFormRef"
               label-width="100px">
        <el-form-item label="省市区/县"
                      prop="address1">
          <el-cascader :options="cityData"
                       v-model="addressForm.address1"
                       style="width:100%"
                       :props="{ expandTrigger: 'hover' }"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址"
                      prop="address2">
          <el-input v-model="addressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addressVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addressVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 展示物流进度 -->
    <el-dialog title="物流进度"
               :visible.sync="progressVisible"
               width="50%">
      <!-- timeline时间线 -->
      <el-timeline>
        <el-timeline-item v-for="(activity, index) in progressInfo"
                          :key="index"
                          :timestamp="activity.time">
          {{activity.context}}
        </el-timeline-item>
      </el-timeline>

      <span slot="footer"
            class="dialog-footer">
        <el-button @click="progressVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="progressVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      // 获取订单列表的参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少条数据
        pagesize: 10
      },
      total: 0,
      // 订单列表
      orderlist: [],
      addressVisible: false,
      addressForm: {
        address1: [],
        address2: ''
      },
      addressRules: {
        address1: [
          { required: true, message: '请选择省市区县', trigger: 'blur' }
        ],
        address2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      cityData,
      progressVisible: false,
      progressInfo: []
    }
  },
  created () {
    this.getOrederList()
  },
  methods: {
    // 获取订单列表
    async getOrederList () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表失败')
      }
      this.orderlist = res.data.goods
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrederList()
    },
    // 监听页码改变事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrederList()
    },
    // 展示修改地址的对话框
    showBox () {
      this.addressVisible = true
    },
    // 监听dialog关闭事件
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    async showProgressBox () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')

      if (res.meta.status !== 200) return this.$message.error('获取物流进度失败！')

      this.progressInfo = res.data
      this.progressVisible = true

      console.log(this.progressInfo)
    }
  }
}
</script>
<style lang="less" scoped>
</style>
