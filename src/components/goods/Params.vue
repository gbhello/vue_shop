<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path : '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 商品分类级联选择框 -->
          <el-cascader
            v-model="selectedKeys"
            :options="cateList"
            :props="props"
            clearable
            @change="handleChange()"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleClick()">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible=true"
          >添加参数</el-button>
          <el-table :data="manyTableData" style="width: 100%" border>
            <el-table-column type="expand">
              <!-- 循环渲染参数标签 -->
              <template slot-scope="scope">
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteParam(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible=true"
          >添加属性</el-button>
          <el-table :data="onlyTableData" style="width: 100%" border>
            <el-table-column type="expand">
              <!-- 循环渲染参数标签 -->
              <template slot-scope="scope">
                <el-tag
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(i,scope.row)"
                >{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-edit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                >编辑</el-button>
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="deleteParam(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加动态参数对话框 -->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="30%"
      @close="addDialogClosed()"
    >
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParam()">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑参数对话框 -->
    <el-dialog
      :title="'编辑' + titleText"
      :visible.sync="editDialogVisible"
      width="30%"
      @close="editDialogClosed()"
    >
      <el-form ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="80px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParam()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      selectedKeys: [],
      cateList: [],
      props: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      activeName: 'many',
      paramsList: [],
      // tab页数据对象
      manyTableData: [],
      onlyTableData: [],
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }
        ]
      },
      editDialogVisible: false,
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          {
            required: true,
            message: '请输入参数名称',
            trigger: 'blur'
          }
        ]
      },
      // 分类id
      cateId: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 3 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取分类数据失败！')
      }
      this.cateList = res.data
    },
    // 级联选择框变化会触发
    async handleChange() {
      this.getParamData()
    },
    // 获取参数数据
    async getParamData() {
      // 证明选中的不是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 选中的是三级分类,获取参数列表
      this.cateId = this.selectedKeys[this.selectedKeys.length - 1]
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('请求分类信息失败！')
      }
      console.log(res.data)
      // 转换属性为数组,属性为空时，使用split分割会产生包含一个空格的数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制标签按钮的显示和隐藏
        item.inputVisible = false
        item.inputValue = ''
      })
      console.log(res.data)
      // 根据不同请求，装载数据到不同的数据对象
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // tab页点击函数
    handleClick() {
      this.getParamData()
    },
    // 监听添加参数对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数
    async addParam() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return
        }
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功!')
        this.addDialogVisible = false
        this.getParamData()
      })
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 展示编辑对话框，根据分类id请求参数数据
    async showEditDialog(attrId) {
      this.editDialogVisible = true
      // 根据id获取数据
      const {
        data: res
      } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrId}`,
        { params: { attr_sel: this.activeName } }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('请求参数信息失败（ID）！')
      }
      this.editForm = res.data
    },
    // 保存编辑信息
    editParam() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 校验通过，发起请求
        const {
          data: res
        } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑参数失败！')
        }
        this.editDialogVisible = false
        this.$message.success('编辑参数成功！')
        this.getParamData()
      })
    },

    // 删除参数
    async deleteParam(attrId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功！')
      this.getParamData()
    },

    // 处理新增标签按下回车动作
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      var inputValue = row.inputValue.trim()
      if (inputValue) {
        row.attr_vals.push(inputValue)
      }
      row.inputVisible = false
      row.inputValue = ''
      this.saveAttrVals(row)
    },
    showInput(row) {
      row.inputVisible = true
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 将对attr_vals的操作，保存到数据库
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: this.activeName,
          attr_vals: row.attr_vals.join(' ')
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('添加参数失败！')
      }
    },
    // 删除对应的参数可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      if (this.selectedKeys.length !== 3) {
        return true
      }
      return false
    },
    // 动态计算标题的文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin-top: 15px;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 90px;
  margin: 10px;
}
.button-new-tag {
  margin: 10px;
}
</style>
