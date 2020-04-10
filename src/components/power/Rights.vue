<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <el-table :data="rightList" border style="width: 100%">
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="authName" label="权限名称"></el-table-column>
        <el-table-column prop="path" label="路径"></el-table-column>
        <el-table-column label="等级">
          <template slot-scope="scope">
            <span v-if="scope.row.level=='0'">
              <el-tag type="success">一级</el-tag>
            </span>
            <span v-if="scope.row.level=='1'">
              <el-tag type="info">二级</el-tag>
            </span>
            <span v-if="scope.row.level=='2'">
              <el-tag type="warning">三级</el-tag>
            </span>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rightList: []
    }
  },
  created() {
    this.getRightList()
  },
  methods: {
    async getRightList() {
      // 请求权限数据
      const { data: res } = await this.$http.get('rights/' + 'list')
      if (res.meta.status !== 200) {
        return this.$message.error('请求数据失败！')
      }
      this.rightList = res.data
    }
  }
}
</script>

<style lang="less" scoped>
</style>
