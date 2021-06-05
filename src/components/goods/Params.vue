<template>
  <div>
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card class="box-card">
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类:</span>
          <!-- 商品级联框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="attrDialogVisible = true"
            >添加属性</el-button
          >
          <el-table :data="manyTableData" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  @close="handleClose(i, scope.row)"
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            :disabled="isBtnDisabled"
            @click="attrDialogVisible = true"
            >添加属性</el-button
          >
          <el-table :data="onlyTableData" style="width: 100%">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  closable
                  @close="handleClose(i, scope.row)"
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index"> </el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 新增属性对话框 -->
    <el-dialog
      :title="'添加' + dialogTitle"
      :visible.sync="attrDialogVisible"
      width="50%"
      @close="addAttrClosed"
    >
      <el-form
        :model="addAttrForm"
        :rules="addAttrFormRules"
        ref="addAttrFormRefs"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="addAttrForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="attrDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改属性对话框 -->
    <el-dialog
      :title="'修改' + dialogTitle"
      :visible.sync="attrDialogEditVisible"
      width="50%"
      @close="editAttrClosed"
    >
      <el-form
        :model="editAttrForm"
        :rules="editAttrFormRules"
        ref="editAttrFormRefs"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item :label="dialogTitle" prop="attr_name">
          <el-input v-model="editAttrForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="attrDialogEditVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateList: [],
      // 级联选择框的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框选中的对象
      selectedCateKeys: [],
      // 页签名称
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态参数数据
      onlyTableData: [],
      // 属性对话框的展示或隐藏的控制
      attrDialogVisible: false,
      // 添加属性对话框内容
      addAttrForm: {
        attr_name: ''
      },
      // 属性对话框规则
      addAttrFormRules: {
        attr_name: [
          { required: true, message: '请输入属性名称', trigger: 'blur' }
        ]
      },
      // 修改属性对话框
      attrDialogEditVisible: false,
      editAttrForm: {
        attr_name: ''
      },
      editAttrFormRules: {
        attr_name: [
          { required: true, message: '请输入属性名称', trigger: 'blur' }
        ]
      },
      // 添加标签显示
      inputVisible: false,
      // 标签输入框
      inputValue: ''
    }
  },
  computed: {
    // 设置添加属性按钮是否灰显
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 获取选中的三级分类id
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 添加属性对话框的标题
    dialogTitle() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态参数'
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.cateList = res.data
    },
    // 级联选择框选中值变化触发的事件
    handleChange() {
      this.getParamsData()
    },
    // tab页签点击事件
    handleClick() {
      this.getParamsData()
    },
    // 获取三级分类数据
    async getParamsData() {
      // 选中的不是三级分类
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 根据所选分类的id，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      res.data.forEach((ele) => {
        // 控制文本框的显示与隐藏
        ele.inputVisible = false
        // 文本框输入值
        ele.inputVisible = ''
        ele.attr_vals = ele.attr_vals ? ele.attr_vals.split(',') : []
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    // 编辑属性对话框
    async showEditDialog(attrId) {
      this.attrDialogEditVisible = true
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attrId}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }
      this.editAttrForm = res.data
    },
    // 关闭对话框重置表单事件
    addAttrClosed() {
      this.$refs.addAttrFormRefs.resetFields()
    },
    // 添加属性对话框确认
    addParams() {
      this.$refs.addAttrFormRefs.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addAttrForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.getParamsData()
        this.attrDialogVisible = false
      })
    },
    // 关闭编辑对话框重置表单
    editAttrClosed() {
      this.$refs.editAttrFormRefs.resetFields()
    },
    // 编辑对话框确认
    editParams() {
      this.$refs.editAttrFormRefs.validate(async (valid) => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editAttrForm.attr_id}`,
          {
            attr_name: this.editAttrForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.getParamsData()
        this.attrDialogEditVisible = false
      })
    },
    // 根据id删除参数
    async removeParams(attrId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该参数, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch((err) => err)
      if (confirmResult !== 'confirm') {
        return this.$message.error('已取消删除!')
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attrId}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功！')
      this.getParamsData()
    },
    // 文本框失去焦点或者摁下enter键
    async handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        this.inputVisible = false
        return
      }
      // 如果有输入内容 进行保存
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      this.inputVisible = false
      this.saveAttrVals(row)
    },
    // 将修改的参数可选项保存到数据库
    async saveAttrVals(row) {
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(',')
        }
      )
      if (res.meta.status !== 200) {
        this.$message.error('修改参数失败！')
      }
      this.$message.success('修改参数成功！')
    },
    // 点击按钮，展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // nextTick当页面元素被重新渲染后 才会执行回调函数中的代码
      this.$nextTick((_) => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除对应的参数可选项
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 100px;
  margin-left: 10px;
  vertical-align: center;
}
</style>
