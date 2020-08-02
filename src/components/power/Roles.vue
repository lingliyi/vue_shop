<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-button type="primary" @click="addRolesUser">添加角色</el-button>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', 'vcenter', index1 === 0 ? 'bdtop' : '']"
              v-for="(item1, index1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限列表 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="remobeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-edit el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限列表 -->
              <el-col :span="19">
                <el-row
                  :class="['vcenter', index2 === 0 ? '' : 'bdtop']"
                  v-for="(item2, index2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="remobeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-edit el-icon-caret-right"></i>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="remobeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column label="#" type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showRolesDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeRolesById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="setRightDialogVisible(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="rightDialogVisible"
      width="50%"
      @close="setRightDialogClose"
    >
      <el-tree
        :data="rightList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRight">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editdialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <el-form :model="editForm" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRolesInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolelist: [],
      rightDialogVisible: false,
      rightList: [],
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      defKeys: [],
      roleId: '',
      //添加用户对话
      addDialogVisible: false,
      //添加用户数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      //修改用户对话框
      editdialogVisible: false,
      editForm: {
        roleName: '',
        roleDesc: ''
      },
      //添加分类验证规则
      addFormRules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }]
      },
      //编辑的id
      editId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    // 获取角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')

      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 删除角色权限
    async remobeRightById(roles, rightId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') return this.$message.info('取消删除')

      const { data: res } = await this.$http.delete(
        `roles/${roles.id}/rights/${rightId}`
      )
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('删除失败')

      roles.children = res.data
    },
    //分配权限
    async setRightDialogVisible(roles) {
      this.roleId = roles.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200)
        return this.$message.error('获取角色权限失败')
      // 把获取到的数据保存到data中
      this.rightList = res.data
      console.log(this.rightList)
      //递归获取三级节点的id
      this.getleafKeys(roles, this.defKeys)
      this.rightDialogVisible = true
    },
    //通过递归形式，获取三级权限下的id，并保存到defkeys数组中
    getleafKeys(node, arr) {
      //如果当前node节点不包含children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item => {
        this.getleafKeys(item, arr)
      })
    },
    //关闭分配权限窗口清空数组
    setRightDialogClose() {
      this.defKeys = []
    },
    async allotRight() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')

      this.$message.success('分配权限成功')
      this.getRolesList()
      this.rightDialogVisible = false
    },
    //删除角色
    async removeRolesById(id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') return this.$message.info('取消失败')

      const { data: res } = await this.$http.delete('roles/' + id)
      // console.log(res)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功')
      this.getRolesList()
    },
    //编辑角色
    async showRolesDialog(id) {
      this.editId = id
      const { data: res } = await this.$http.get('roles/' + id)
      this.editForm.roleName = res.data.roleName
      this.editForm.roleDesc = res.data.roleDesc
      this.editdialogVisible = true
    },
    //提交编辑角色
    async editRolesInfo() {
      const { data: res } = await this.$http.put(
        'roles/' + this.editId,
        this.editForm
      )
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('提交失败')
      }
      this.$message.success('提交成功')
      //更新列表
      this.getRolesList()
      this.editdialogVisible = false
    },
    //监听编辑角色关闭窗口事件
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    //添加角色
    addRolesUser() {
      this.addDialogVisible = true
    },
    //添加角色确定按钮
    async addRolesInfo() {
      const { data: res } = await this.$http.post('roles', this.addForm)
      console.log(res)
      if (res.meta.status !== 201) {
        return this.$message.error('添加失败')
      }
      this.$message.success('添加成功')
      this.getRolesList()
      this.addDialogVisible = false
    },

    //添加角色消息框关闭
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    }
    //
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.el-button {
  margin-bottom: 20px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
