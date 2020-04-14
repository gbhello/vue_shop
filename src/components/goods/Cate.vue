<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path:'/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-button type="primary" @click="showAddCategoryDialog()">添加分类</el-button>

      <!-- 插件方式实现表格 -->
      <tree-table
        class="treeTable"
        :data="categoryList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag v-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag v-if="scope.row.cat_level===2">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="option">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>

      <!-- 展示商品分类列表 -->
      <!-- <el-table :data="categoryList" border>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="分类名称" prop="cat_name"></el-table-column>
        <el-table-column label="是否有效">
          <template slot-scope="scope">
            <span v-if="scope.row.cat_deleted">
              <el-button type="danger" icon="el-icon-close" size="mini" circle></el-button>
            </span>
            <span v-else>
              <el-button type="success" icon="el-icon-check" size="mini" circle></el-button>
            </span>
          </template>
        </el-table-column>
        <el-table-column label="排序">
          <template slot-scope="scope">
            <span v-if="scope.row.cat_level==0">
              <el-tag>一级</el-tag>
            </span>
            <span v-if="scope.row.cat_level==1">
              <el-tag>二级</el-tag>
            </span>
            <span v-if="scope.row.cat_level==2">
              <el-tag>三级</el-tag>
            </span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
          </template>
        </el-table-column>
      </el-table>-->
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10]"
        :page-size="5"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="30%"
      @close="addCateDialogClosed()"
    >
      <span>
        <!-- 添加分类表单 -->
        <el-form
          ref="addCateFormRef"
          :model="addCateForm"
          label-width="80px"
          :rules="addCateFormRules"
        >
          <el-form-item label="分类名称" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类">
            <el-cascader
              expandTrigger="hover"
              v-model="selectedKeys"
              :options="parentCateList"
              :props="cascaderProps"
              @change="parentCateChanged()"
              clearable
              checkStrictly
            ></el-cascader>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 分类列表数据
      categoryList: [],

      // 获取商品分类列表的参数对象
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 将当前列定义为模板列
          type: 'template',
          // 当前这一列使用的模板名称
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'option'
        }
      ],
      // 添加分类对话框可见性
      addCateDialogVisible: false,
      addCateForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      // 添加分类-分类名称校验规则
      addCateFormRules: {
        cat_name: [
          {
            required: true,
            message: '请输入分类名称',
            trigger: 'blur'
          }
        ]
      },
      // 父级分类数据
      parentCateList: [],
      // 级联选择器属性
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选中的父级分类id
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 商品分类列表数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('请求商品分类数据失败！')
      }
      this.categoryList = res.data.result
      this.total = res.data.total
      console.log(res.data)
    },
    // 每页显示数据量函数
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 页码改变函数
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 展示添加分类对话框
    async showAddCategoryDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取符集分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类信息失败！')
      }
      console.log(res.data)
      this.parentCateList = res.data
    },
    // 级联选择器触发函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      // 如果selectedKeys数组中的length>0，证明选中的是父级分类
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys.pop()
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 添加分类
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        // 校验未通过，直接返回
        if (!valid) return
        // 校验通过，发起请求
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类数据失败！')
        }
        this.$message.success('添加分类数据成功！')
        this.addCateDialogVisible = false
        this.getCateList()
      })
    },
    // 监听添加分类对话框
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
</style>
