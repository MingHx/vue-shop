<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    clearable
                    v-model="queryInfo.query"
                    @clear="getGoodsList">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="goAddPage">添加商品</el-button>
        </el-col>
      </el-row>

      <!-- table表格区域 -->
      <el-table :data="goodsList"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label=" 商品名称"
                         prop="goods_name"></el-table-column>
        <el-table-column label=" 商品价格(元)"
                         prop="goods_price"
                         width="100px"></el-table-column>
        <el-table-column label=" 商品重量"
                         prop="goods_weight"
                         width="70px"></el-table-column>
        <el-table-column label=" 商品数量"
                         prop="goods_number"
                         width="70px"></el-table-column>
        <el-table-column label=" 创建时间"
                         prop="add_time"
                         width="140px">
          <template slot-scope="scope">
            {{scope.row.add_time | dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label=" 操作"
                         width="130px">
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"></el-button>
            <!-- @click="showEditDialog(scope.row.goods_id)" -->
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeGoodsById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[10, 30, 50, 100]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total"
                     background>
      </el-pagination>

      <!-- 商品修改对话框 -->
      <!-- <el-dialog title="商品修改"
                 :visible.sync="editDialogVisible"
                 width="50%"
                 @close="editDialogClosed"> -->

      <!-- 修改表单区域 -->
      <!-- <el-form :model="editForm"
                 :rules="editFormRules"
                 ref="editFormRef"
                 label-width="100px">
          <el-form-item label="商品名称:"
                        prop="goods_name">
            <el-input v-model="editForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格:"
                        prop="goods_price">
            <el-input v-model="editForm.goods_price"
                      type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品重量:"
                        prop="goods_weight">
            <el-input v-model="editForm.goods_weight"
                      type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品数量:"
                        prop="goods_number">
            <el-input v-model="editForm.goods_number"
                      type="number"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer"
              class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary"
                     @click="editGoodsInfo()">确 定</el-button>
        </span>
      </el-dialog> -->

    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 商品列表数组
      goodsList: [],
      // 商品总数
      total: 0,
      editDialogVisible: false,
      editForm: {
        goods_cat: ''
      },
      editFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }

        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ]
      }

    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    // 根据分页获取对应商品列表
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败')
      }
      this.goodsList = res.data.goods
      console.log(this.goodsList)
      this.total = res.data.total
    },
    // 监听pagesize改变事件
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 监听页码改变事件
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    // 根据id删除商品
    async removeGoodsById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      // 如果用户确认删除，返回字符串confirm
      // 如果用户取消删除，返回字符串cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }

      const { data: res } = await this.$http.delete('goods/' + id)

      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败！')
      }

      this.$message.success('删除商品成功！')
      // 重新获取用户列表
      this.getGoodsList()
    },
    // 跳转到添加商品页面
    goAddPage () {
      this.$router.push('/goods/add')
    }
    /*
    // 展示编辑用户信息的对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品信息失败！')
      }
      this.editForm = res.data

      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    editGoodsInfo (id) {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('goods/' + this.editForm.goods_id,
          {
            goods_name: this.editForm.goods_name,
            goods_price: this.editForm.goods_price,
            goods_weight: this.editForm.goods_weight,
            goods_number: this.editForm.goods_number

          })

        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败！')
        }

        this.$message.success('修改用户成功！')
        // 隐藏用户对话框
        this.editDialogVisible = false
        // 重新获取用户列表
        this.getGoodsList()
      })
    } */

  },
  computed: {

  }
}
</script>

<style lang="less" scoped>
</style>
