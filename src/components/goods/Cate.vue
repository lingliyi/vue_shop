<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:green"
          ></i>
          <i class="el-icon-serror" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一级</el-tag>
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            type="success"
            size="mini"
            >二级</el-tag
          >
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
            @click="editCate(scope.row.cat_id)"
          ></el-button>
          <el-button
            type="primary"
            icon="el-icon-delete"
            size="mini"
            @click="removeCate(scope.row.cat_id)"
          ></el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 30]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        change="parentCateChanged"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类消息框 -->
    <el-dialog
      title="提示"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader
            class="formItem"
            v-model="selectedKeys"
            expand-trigger="hover"
            :options="parentCateList"
            :props="casCaderProps"
            @change="parentCateChanged"
            clearable
            change-on-select="true"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCade">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改分类 -->
    <el-dialog
      title="提示"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editCateDialogClosed"
    >
      <el-form ref="form" :model="editCateForm" label-width="80px">
        <el-form-item label="分类名称">
          <el-input v-model="editCateForm.title"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCade">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //   商品分类数据
      cateList: [],
      //   总数据条数
      total: 0,
      //为table指定属性
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        {
          label: '是否有效',
          //表示将当列定义为模板列
          type: 'template',
          //当前这一列使用的名称
          template: 'isok'
        },
        {
          label: '排序',
          //表示将当列定义为模板列
          type: 'template',
          //当前这一列使用的名称
          template: 'order'
        },
        {
          label: '操作',
          //表示将当列定义为模板列
          type: 'template',
          //当前这一列使用的名称
          template: 'opt'
        }
      ],
      //添加分类消息框
      addDialogVisible: false,
      //添加分类表单数据
      addCateForm: {
        //将要添加的分类的名称
        cat_name: '',
        //父级分类id
        cat_pid: 0,
        //分类的等级，默认要添加的是一级分类
        cat_level: 0
      }, //添加分类验证规则
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      //父级分类数据列表
      parentCateList: [],
      //指定级联选择器对象
      casCaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      //选中的父级分类的id数组
      selectedKeys: [],
      //修改分类数据
      editCateForm: {
        title: ''
      },
      editDialogVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      //   console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      //   把商品分类数据赋值给catelist
      this.cateList = res.data.result
      this.total = res.data.total
      //   console.log(this.cateList)
    },
    //条数
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    //页数
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    //添加分类消息框
    addDialog() {
      this.getParentCateList()
      this.addDialogVisible = true
    },
    //获取父级分类数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败')
      }

      //   console.log(res)
      //   console.log(res.data)

      this.parentCateList = res.data
    },
    //选择项发生变化触发这个函数
    parentCateChanged() {
      //   console.log(this.selectedKeys)
      //如果 selectedkey数组中的length大于0，证明选中的父级分类
      //反之就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的id
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        //父级分类的id
        this.addCateForm.cat_pid = 0
        //为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    //点击按钮，添加新的分类
    addCade() {
      //   console.log(this.addCateForm)
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )

        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addDialogVisible = false
      })
    },
    //监听对话框的关闭时间，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    //删除分类
    async removeCate(id) {
      const { data: res } = await this.$http.delete('categories/' + id)
      //   console.log(res)
      this.getCateList()
    },
    //修改分类
    async editCate(id) {
      this.editId = id
      this.editDialogVisible = true
      const { data: res } = await this.$http.get('categories/' + id)
      //   console.log(res)
      if (res.meta.status !== 200) return this.$message.error('获取失败')
      this.editCateForm.title = res.data.cat_name
    },
    //确定修改
    async editCade() {
      const { data: res } = await this.$http.put(`categories/${this.editId}`, {
        cat_name: this.editCateForm.title
      })
      //   console.log(res)
    },
    //修改窗口关闭
    editCateDialogClosed() {}
  }
}
</script>

<style lang="less" scoped>
.formItem {
  width: 100%;
}
</style>
