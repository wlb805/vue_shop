<template>
  <div>
    <!-- 导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片试图 -->
    <el-card class="box-card">
      <!-- 搜索添加区 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input @clear="getGoodsList" v-model="queryInfo.query" placeholder="请输入内容" clearable>
            <el-button @click="getGoodsList" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary">添加商品</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- 用户列表区域 -->
    <el-table :data="goodslist" border stripe="true">
      <el-table-column type="index"></el-table-column>
      <el-table-column label="商品名称" prop="goods_name"></el-table-column>
      <el-table-column label="商品价格(元)" prop="goods_price"></el-table-column>
      <el-table-column label="创建总量" prop="goods_weight"></el-table-column>
      <el-table-column label="创建时间" prop="add_time">
        <template slot-scope="scope">
          {{scope.row.add_time | dateFormat}}
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <!-- 修改 -->
          <el-tooltip class="item" effect="dark" content="编辑商品" placement="top" :enterable="false">
            <el-button @click="showEditDialog(scope.row.id)" type="primary" icon="el-icon-edit" size="mini"></el-button>
          </el-tooltip>
          <!-- 删除 -->
          <el-tooltip class="item" effect="dark" content="删除商品" placement="top" :enterable="false">
            <el-button @click="removeById(scope.row.goods_id)" type="danger" icon="el-icon-delete" size="mini"></el-button>
          </el-tooltip>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页区域 -->
    <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[1, 2, 5, 10]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total">
    </el-pagination>
  </div>
</template>
<script>
export default {
  data () {
    return {
      goodslist: [],
      total: '',
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      }
    }
  },
  created () {
    this.getGoodsList()
  },
  methods: {
    async getGoodsList () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      console.log(res)
      if (res.meta.status !== 200) { return this.$message.error('获取商品列表失败') }

      this.goodslist = res.data.goods
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
    async removeById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败!')
      }
      this.$message.success('删除商品成功!')
      this.getGoodsList()
    }
  }
}
</script>
<style lang="less" scoped>
</style>
