<template>
  <div>
    <!--    面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-alert
        :closable="false"
        title="注意：只允许为第三级分类设置相关参数"
        type="warning"
        show-icon>
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
<!--          选择分类商品的级联选择框-->
          <el-cascader v-model="selectedKeys"
           :options="CateList"
           :props="cascaderProps"
           @change="HandleChanged"
           size="small"
           :filterable="true"
           clearable>
          </el-cascader>
        </el-col>
      </el-row>
<!--      tab页签区域-->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialogVisible">添加参数</el-button>
<!--          动态参数表格-->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
<!--                循环渲染tag标签-->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleCloses(i,scope.row)">
                  {{item}}
                </el-tag>
<!--                输入文本框-->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  clearable
                  autosize
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
<!--                添加按钮-->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialogVisible(scope.row.attr_id)">操作</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="showAddDialogVisible">添加属性</el-button>
          <!--          静态属性表格-->
          <el-table :data="onlyTableData">
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!--                循环渲染tag标签-->
                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleCloses(i,scope.row)">
                  {{item}}
                </el-tag>
                <!--                输入文本框-->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  clearable
                  autosize
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!--                添加按钮-->
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index"></el-table-column>
            <el-table-column label="属性名称" prop="attr_name"></el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditDialogVisible(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!--添加参数或属性对话框-->
    <el-dialog
      :title="'添加' + titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="addDialogClose">
<!--      添加参数的对话框-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name">}</el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
    </el-dialog>
<!--    修改参数或属性对话框-->
    <el-dialog
      :title="'修改' + titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      :close-on-click-modal="false"
      @close="editDialogClose">
      <!--      修改参数的对话框-->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name">}</el-input>
        </el-form-item>

      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editParams">确 定</el-button>
        </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  created () {
    this.getCateList()
  },
  data () {
    return {
      CateList: [],
      selectedKeys: [],
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      activeName: 'many',
      manyTableData: [],
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      editDialogVisible: false,
      // 修改的数据表单对象
      addForm: {
        attr_name: ''
      },
      editForm: {
        // Todo
        attr_name: ''
      },
      addFormRules: {
        attr_name: [{
          required: true, message: '请输入参数名称', trigger: 'blur'
        }]
      },
      editFormRules: {
        attr_name: [{
          required: true, message: '请输入参数名称', trigger: 'blur'
        }]
      },
      // 控制按钮与文本框的切换显示
      inputVisible: false,
      // 文本框中输入的内容
      inputValue: ''
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败')
      this.CateList = res.data
    },
    HandleChanged() {
      this.getParamsData()
    },
    async getParamsData() {
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return null
      }
      // 根据所选分类的id，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        { params: { sel: this.activeName } })
      if (res.meta.status !== 200) return this.$message.error('获取参数或属性失败')
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 添加一个字段为文本框提示隐藏的属性
        item.inputVisible = false
        item.inputValue = ''
      })
      // console.log(res.data)
      if (this.activeName === 'many') {
        this.manyTableData = res.data
        // console.log(this.manyTableData)
      } else {
        this.onlyTableData = res.data
        // console.log(this.onlyTableData)
      }
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      // console.log(this.activeName)
      this.getParamsData()
    },
    // 打开添加对话框操作
    showAddDialogVisible() {
      this.addDialogVisible = !this.addDialogVisible
    },
    async showEditDialogVisible(attrid) {
      console.log(attrid)
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attrid}`,
        { params: { attr_sel: this.activeName } })
      console.log(res)
      console.log(res.meta.status)
      if (res.meta.status !== 200) return this.$message.error('查询失败')
      this.editForm = res.data
      console.log(res.data)
      this.editDialogVisible = !this.editDialogVisible
    },
    // 添加对话框的关闭操作
    addDialogClose() {
      console.log(this.addForm)
      this.$refs.addFormRef.resetFields()
    },
    // 修改对框框的关闭操作
    editDialogClose() {
      console.log(this.addForm)
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮添加参数
    async addParams () {
      this.$refs.addFormRef.validate(valid => {
        if (!valid) return null
      })
      const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`,
        { attr_name: this.addForm.attr_name, attr_sel: this.activeName })
      if (res.meta.status !== 201) return this.$message.error('添加失败')
      this.$message.success('添加成功')
      this.getParamsData()
      this.addDialogVisible = false
    },
    async editParams() {
      this.$refs.editFormRef.validate(valid => {
        if (!valid) return null
      })
      console.log(this.editForm)
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
        { attr_name: this.editForm.attr_name, attr_sel: this.activeName, attr_vals: 'test' })
      console.log(res)
      if (res.meta.status !== 200) return this.$message.error('修改参数失败')
      this.$message.success('修改成功')
      this.getParamsData()
      this.editDialogVisible = false
    },
    async removeParams(attrid) {
      const confirmResult = await this.$confirm('此操作将永久删除该参数，是否继续？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attrid}`)
      if (res.meta.status !== 200) return this.$message.error('删除参数失败')
      this.$message.success('删除参数成功')
      this.getParamsData()
    },
    // 文本框失去焦点或按下了enter建都会触发
    async handleInputConfirm(row) {
      console.log(row)
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return null
      }
      if (row.inputValue.trim().length > 20) {
        return this.$message.error('输入的文字数需要小于等于20')
      }
      row.attr_vals.push(row.inputValue.trim())
      const { data: res } = await this.$http.put(`categories/${row.cat_id}/attributes/${row.attr_id}`,
        { attr_name: row.attr_name, attr_vals: row.attr_vals.join(' '), attr_sel: row.attr_sel })
      if (res.meta.status !== 200) return this.$message.error('添加tag失败')
      this.$message.success('添加tag成功')
      row.inputValue = ''
      row.inputVisible = false
    },
    // 展示文本输入框
    showInput(row) {
      row.inputVisible = true
      // 让文本框自动获得焦点
      // $nextTick 方法的作用，就是当页面上的元素被重新渲染之后，才会指定回调函数中的代码，如果不等渲染可能会找不到ref对象
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 关闭tag操作
    async handleCloses(i, row) {
      const v = row.attr_vals.splice(i, 1)
      const { data: res } = await this.$http.put(`categories/${row.cat_id}/attributes/${row.attr_id}`,
        { attr_name: row.attr_name, attr_vals: row.attr_vals.join(' '), attr_sel: row.attr_sel })
      if (res.meta.status !== 200) {
        row.attr_vals.splice(i, 0, v[0])
        console.log(row.attr_vals)
        return this.$message.error('删除tag失败')
      }
      this.$message.success('删除tag成功')
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true，否则false
    isBtnDisabled() {
      return this.selectedKeys.length !== 3
    },
    // 当前选中的三级分类的id
    cateId() {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      }
      return null
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt{
  margin: 15px;
}
.el-tag{
  margin: 10px;
}
.input-new-tag{
  width: 200px;
}
</style>
