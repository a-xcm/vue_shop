<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-row>
          <el-button type="primary" @click="addRoleDialogVisable = true">添加角色</el-button>
        </el-row>
      </el-row>

      <!-- 角色列表区域 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable
                @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <!-- 通过for 循环 嵌套二级权限 -->
                <el-row :class="[i2 === 0 ? '' : 'bdtop','vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable
                    @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag type="warning" v-for="(item3) in item2.children" :key="item3.id" closable
                    @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>

      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisable" width="30%"
    @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist" show-checkbox node-key="id"
      :props="treeProps" default-expand-all
      :default-checked-keys="defKeys"
      ref="treeRef">
      </el-tree>
      <span slot="footer" class="dialog-footer" >
        <el-button @click="setRightDialogVisable = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 认</el-button>
      </span>
    </el-dialog>
    <!-- 添加角色的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addRoleDialogVisable" width="30%"
    @close="addRoleDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisable = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 认</el-button>
      </span>
    </el-dialog>

     <!-- 编辑角色的对话框 -->
    <el-dialog title="添加用户" :visible.sync="editRoleDialogVisable" width="30%"
    @close="editRoleDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisable = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 认</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolelist: [],
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisable: false,
      // 所有权限的数据
      rightslist: [],
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的id
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 控制添加角色对话框的显示与隐藏
      addRoleDialogVisable: false,
      // 添加角色的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加角色表单的验证规则
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { max: 10, message: '长度最大 10 个字符', trigger: 'blur' }
        ]
      },
      // 查询角色的对象
      editForm: {},
      // 控制编辑对话框的显示与隐藏
      editRoleDialogVisable: false,
      // 编辑角色表单的验证规则
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { max: 10, message: '长度最大 10 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolelist = res.data
    },
    // 根据Id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      // 如果用户确认删除 则返回值为字符串 confirm
      // 如果用户取消删除 则返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 发起删除请求
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败！')
      }
      this.$message.success('删除权限成功！')
      role.children = res.data
    },

    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败')
      }
      // 把获取到的权限数据保存到 data 中
      this.rightslist = res.data
      // 递归获取三级节点的Id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisable = true
    },
    // 通过递归的形式，获取角色下所有三级权限的Id，并保存到 defkeys 数组中
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }

      node.children.forEach(item =>
        this.getLeafKeys(item, arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击确认为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`,
        { rids: idStr })

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisable = false
    },
    // 点击确认，添加角色
    addRole() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        // 发送添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败！')
        }
        this.addRoleDialogVisable = false
        this.getRolesList()
        this.$message.success('添加角色成功')
      })
    },
    // 监听添加角色对话框的关闭事件
    addRoleDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 展示 编辑对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询角色信息失败！')
      }
      this.editForm = res.data
      this.editRoleDialogVisable = true
    },
    // 点击确认,编辑角色信息
    editRoleInfo() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return false
        }
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          // 判断输入的值是否为多个空格或者为空 直接取一个空字符串
          roleDesc: this.editForm.roleDesc.split(' ').join('').length === 0 ? ' ' : this.editForm.roleDesc
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新角色信息失败！')
        }
        console.log(res.data)
        // 数据同步最新
        this.editForm = res.data
        // 关闭对话框
        this.editRoleDialogVisable = false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新角色信息成功')
      })
    },
    // 监听编辑角色对话框关闭事件
    editRoleDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 根据角色id删除 角色
    async removeRoleById(id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // console.log(confirmResult)
      // 如果用户确认删除 则返回值为字符串 confirm
      // 如果用户取消删除 则返回值为字符串 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 发起删除请求
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功！')
      this.getRolesList()
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
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
