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
      <el-table :data="roleslist" stripe style="width: 100%">
        <el-table-column type="expand">
          <template slot-scope="scope">
             <el-row :class="['bdbottom', i1 ===0 ? 'bdtop' : '']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag>
              </el-col>
              <el-col :span="19">
                <i class="el-icon-caret-right"></i>
              </el-col>
             </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template>
            <el-button type="primary" icon="el-icon-edit" circle></el-button>
            <el-button type="warning" icon="el-icon-star-off" circle></el-button>
            <el-button type="danger" icon="el-icon-delete" circle></el-button>
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
      roleslist: []
    }
  },
  created() {
    this.getrolelist()
  },
  methods: {
    async getrolelist() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.console.error(res.meta.msg)
      }
      console.log(res)
      this.roleslist = res.data
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
