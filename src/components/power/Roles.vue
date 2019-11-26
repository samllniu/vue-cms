<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" round>添加角色</el-button>
        </el-col>
      </el-row>
      <el-table :data="rolelist" stripe style="width: 100%">
        <el-table-column type="expand">
          <template slot-scope="scope">
             <el-row :class="['bdbottom', i1 ===0 ? 'bdtop' : '']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag closable @close="removeRoleItem(scope.row, item1.id)">{{item1.authName}}</el-tag>
                 <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <el-row :class="[i2 === 0 ? '' : 'bdtop']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="8">
                    <el-tag type="warning" closable @close="removeRoleItem(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="16">
                    <el-tag type="success" v-for="item3 in item2.children" :key="item3.id" closable @close="removeRoleItem(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
             </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" circle></el-button>
            <el-button type="warning" icon="el-icon-star-off" circle @click="settingRoles(scope.row)"></el-button>
            <el-button type="danger" icon="el-icon-delete" circle></el-button>
          </template>
        </el-table-column>
      </el-table>
       <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @closed="setDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightslist"  :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色的对话框 -->
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      rolelist: [],
      rightslist: [],
      setRightDialogVisible: false,
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: ''
    }
  },
  created() {
    this.getrolelist()
  },
  methods: {
    // 获取所有角色的数据
    async getrolelist() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.rolelist = res.data
      console.log(this.rolelist)
    },
    // 删除角色
    async removeRoleItem(role, rightId) {
      const result = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (result !== 'confirm') {
        return this.$message.info('取消权限删除操作')
      }
      const { data: res } = this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限操作失败')
      this.roleslist = res.data
    },
    // 获取权限数据
    async settingRoles(roles) {
      this.roleId = roles.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      this.getKey(roles, this.defKeys)
      this.rightslist = res.data
      this.setRightDialogVisible = true
    },
    // 使用地柜获取三级权限的ID
    getKey(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(element => {
        this.getKey(element, arr)
      })
    },
    // 关闭权限树的时候清空权限树的数据
    setDialogClosed() {
      this.defKeys = []
    },
    // 在树形结构权限列表中增加用户的权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]

      const idStr = keys.join(',')

      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }

      this.$message.success('分配权限成功！')
      this.getrolelist()
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
.vcenter {
  display: flex;
  align-items: center;
}

</style>
