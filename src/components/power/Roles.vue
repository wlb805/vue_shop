<template>
  <div>
    <!-- 导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 搜索添加区 -->
      <el-row :gutter="20">
        <el-col :span="4">
          <el-button type="primary" @click="addrDialogVisible=true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bdbottom',i1===0?'bdtop' :'']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
              </el-col>
              <el-col :span="19">
                <el-row :class="i2===0?'' :'bdtop'" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row,item3.id)" v-for="(item3) in item2.children" :key="item3.id" type="warning">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button @click="showrEditDialog(scope.row.id)" size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button @click="removeRoleById(scope.row.id)" size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button @click="showSetRightDialog(scope.row)" size="mini" type="warning" icon="el-icon-setting">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加角色" :visible.sync="addrDialogVisible" width="50%" @close="addrDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="addrForm" status-icon :rules="addrFormRules" ref="addrFormRef" label-width="70px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addrForm.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addrForm.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addrDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色" :visible.sync="editrDialogVisible" width="50%" @close="editrDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="editrForm" status-icon :rules="editrFormRules" ref="editrFormRef" label-width="70px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editrForm.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editrForm.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editrDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog @close="setRightDialogClosed" title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
      <el-tree ref="treeRef" :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all="false" :default-checked-keys="defKeys"></el-tree>

      <!-- 底部按钮区 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      addrFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' }, { message: '角色名不为空!', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }, { message: '角色描述不为空!', trigger: 'blur' }
        ]
      },
      editrFormRules: {
        roleName: [
          { required: true, message: '请输入角色名', trigger: 'blur' }, { message: '角色名不为空!', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }, { message: '角色描述不为空!', trigger: 'blur' }
        ]
      },
      addrDialogVisible: false,
      editrDialogVisible: false,
      addrForm: {
        roleName: '',
        roleDesc: ''
      },
      editrForm: {
      },
      setRightDialogVisible: false,
      rightslist: [],
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys: [],
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败!')
      }
      this.rolesList = res.data
    },
    addrDialogClosed () {
      this.$refs.addrFormRef.resetFields()
    },
    addRole () {
      this.$refs.addrFormRef.validate(async vaild => {
        if (!vaild) { return }
        const { data: res } = await this.$http.post('roles', this.addrForm)
        if (res.meta.status !== 201) { return this.$message.error('添加用户失败!') }
        this.$message.success('添加用户成功!')
        this.addrDialogVisible = false
        this.getRolesList()
      })
    },
    // 展示编辑用户的对话框
    async showrEditDialog (id) {
      // console.log(id)
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败!')
      }
      this.editrForm = res.data
      this.editrDialogVisible = true
    },
    editrDialogClosed () {
      this.$refs.editrFormRef.resetFields()
    },
    editRolesInfo () {
      this.$refs.editrFormRef.validate(async vaild => {
        console.log(this.editrForm)
        if (!vaild) { return }
        const { data: res } = await this.$http.put('roles/' + this.editrForm.roleId, {
          roleName: this.editrForm.roleName,
          roleDesc: this.editrForm.roleDesc
        })
        if (res.meta.status !== 200) { return this.$message.error('修改角色失败!') }
        this.$message.success('修改角色成功!')
        this.editrDialogVisible = false
        this.getRolesList()
      })
    },
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除角色失败!')
      }
      this.$message.success('删除角色成功!')
      this.getRolesList()
    },
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + rightId)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败!')
      }
      this.$message.success('删除权限成功!')
      role.children = res.data
    },
    // 展示分配权限对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      // 获取所有权限数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败!')
      }
      this.rightslist = res.data
      // console.log(this.rightslist)

      // 递归获取三级节点id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    setRightDialogClosed () {
      this.defKeys = []
    },
    async allotRights () {
      const keys = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      // console.log(keys)
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta !== 200) {
        this.$message.error('分配失败!')
      }
      this.$message.success('分配成功!')
      this.getRolesList()
      this.setRightDialogVisible = false
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
</style>
