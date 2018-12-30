<template>
  <div class="users">
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item to="/users">用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索框 -->
    <el-input placeholder="请输入内容" v-model="query" class="input-with-select">
      <el-button slot="append" icon="el-icon-search" @click="searchUser"></el-button>
    </el-input>
    <el-button type="success" plain @click="showAdd">用户添加</el-button>
    <!-- 表格 -->
    <el-table :data="userList" style="width: 100%">
      <el-table-column label="姓名" prop="username" width="200"></el-table-column>
      <el-table-column label="邮箱" prop="email" width="200"></el-table-column>
      <el-table-column label="电话" prop="mobile" width="200"></el-table-column>
      <el-table-column label="状态" prop="mg_state" width="200">
        <template slot-scope="scope">
          <el-switch
            @change="changeStatus(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949"
          ></el-switch>
        </template>
      </el-table-column>
      <el-table-column label="操作" prop="date">
        <template slot-scope="scope">
          <el-button
            type="primary"
            icon="el-icon-edit"
            @click="showEdit(scope.row)"
            size="mini"
            circle
            plain
          ></el-button>
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            circle
            plain
            @click="delUser(scope.row.id)"
          ></el-button>
          <el-button type="success" icon="el-icon-check" size="mini" round plain>分配角色</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <el-pagination
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    ></el-pagination>
    <!-- 添加的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="40%">
      <el-form status-icon :model="addForm" :rules="rules" ref="addForm" label-width="80px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" placeholder="请输入密码" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile" placeholder="请输入电话"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改的对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="40%">
      <el-form status-icon :model="editForm" :rules="rules" ref="editForm" label-width="80px">
        <el-form-item label="用户名">
          <el-tag type="info">{{editForm.username}}</el-tag>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="editForm.mobile" placeholder="请输入电话"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateUser">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// import axios from 'axios'
export default {
  data() {
    return {
      userList: [],
      query: '',
      currentPage: 1,
      pageSize: 2,
      total: 0,
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      rules: {
        username: [
          { required: true, message: '用户名不能为空', trigger: 'blur' },
          { min: 3, max: 9, message: '长度在 3 到 9 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '密码不能为空', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12 个字符', trigger: 'blur' }
        ],
        email: [
          { type: 'email', message: '请输入一个合法的邮箱', trigger: 'blur' }
        ],
        mobile: [
          {
            pattern: /^1[3-9]\d{9}$/,
            message: '请输入一个合法的手机号',
            trigger: 'blur'
          }
        ]
      },
      editDialogVisible: false,
      editForm: {
        email: '',
        mobile: '',
        id: '',
        username: ''
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      let res = await this.axios({
        method: 'get',
        url: 'users',
        params: {
          query: this.query,
          pagenum: this.currentPage,
          pagesize: this.pageSize
        }
      })
      let {
        meta: { status },
        data: { users, total }
      } = res
      if (status === 200) {
        this.userList = users
        this.total = total
      }
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.getUserList()
    },
    searchUser() {
      this.currentPage = 1
      this.getUserList()
    },
    async delUser(id) {
      try {
        await this.$confirm('你确定要删除这个用户吗？', '温馨提示', {
          type: 'warning'
        })
        let res = await this.axios({
          method: 'delete',
          url: `users/${id}`
        })

        let {
          meta: { status }
        } = res
        if (status === 200) {
          if (this.userList.length <= 1 && this.currentPage > 1) {
            this.currentPage--
          }
          this.getUserList()
          this.$message.success('删除成功')
        } else {
          this.$message.error('删除失败')
        }
      } catch (e) {
        this.$message.info('已取消删除')
      }
    },
    async changeStatus({ id, mg_state: mgState }) {
      let res = await this.axios({
        method: 'put',
        url: `users/${id}/state/${mgState}`
      })
      let {
        meta: { status }
      } = res
      if (status === 200) {
        this.$message.success('状态修改成功')
        this.getUserList()
      }
    },
    showAdd() {
      this.addDialogVisible = true
    },
    async addUser() {
      try {
        await this.$refs.addForm.validate()
        let res = await this.axios({
          method: 'post',
          url: 'users',
          data: this.addForm
        })
        if (res.meta.status === 201) {
          this.$refs.addForm.resetFields()
          this.addDialogVisible = false
          this.total++
          this.currentPage = Math.ceil(this.total / this.pageSize)
          this.getUserList()
          this.$message.success('用户添加成功')
        } else {
          this.$message.error(res.meta.msg)
        }
      } catch (e) {
        return false
      }
    },
    showEdit(user) {
      this.editDialogVisible = true
      // console.log(user)
      this.editForm.id = user.id
      this.editForm.email = user.email
      this.editForm.mobile = user.mobile
      this.editForm.username = user.username
    },
    async updateUser() {
      try {
        await this.$refs.editForm.validate()
        let res = await this.axios({
          method: 'put',
          url: `users/${this.editForm.id}`,
          data: this.editForm
        })
        if (res.meta.status === 200) {
          this.$refs.editForm.resetFields()
          this.editDialogVisible = false
          this.getUserList()
          this.$message.success('修改用户信息成功')
        } else {
          this.$message.error(res.meta.msg)
        }
      } catch (e) {
        return false
      }
    }
  }
}
</script>

<style lang="less" scoped>
.el-breadcrumb {
  height: 40px;
  line-height: 40px;
}
.el-input {
  width: 400px;
  margin-bottom: 5px;
}
</style>
