<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            clearable
            @clear="getUserList"
            v-model="queryInfo.query"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button
            icon="el-icon-plus"
            @click="addDialogVisible = true"
            type="primary"
            >add user</el-button
          >
        </el-col>
      </el-row>

      <el-table :data="userList" style="width: 100%" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="Name" width="180">
        </el-table-column>
        <el-table-column prop="email" label="Email" width="180">
        </el-table-column>
        <el-table-column prop="mobile" label="Mobile" width="180">
        </el-table-column>
        <el-table-column prop="role_name" label="Role"></el-table-column>
        <el-table-column label="Status">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChange(scope.row)"
            ></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="Function" width="180px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              @click="editUser(scope.row.id)"
              icon="el-icon-edit"
            ></el-button>

            <el-button
              type="danger"
              size="mini"
              @click="deleteUser(scope.row.id)"
              icon="el-icon-delete"
            ></el-button>
            <el-tooltip
              effect="dark"
              content="Assign role"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                size="mini"
                @click="eidt(scope.row.id)"
                icon="el-icon-setting"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <el-dialog
      title="Add User"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="this.$refs.addFormRef.resetFields()"
    >
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item label="user name" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="password" prop="password">
          <el-input type="password" v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="email" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="mobile" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog
      title="Edit user"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="this.$refs.editFormRef.resetFields()"
    >
      <el-form
        :model="editForm"
        ref="editFormRef"
        :rules="editFormRules"
        label-width="100px"
      >
        <input type="hidden" v-model="editForm.id" />
        <el-form-item label="user name">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="email" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="mobile" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitEditUser">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      addDialogVisible: false,
      editDialogVisible: false,

      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      userList: [],
      total: 0,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editForm: {
      },
      addFormRules: {
        username: [
          { required: true, message: 'Please input username', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: 'Please input password', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: 'Please input email', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
        ],
        mobile: [
          { required: true, message: 'Please input mobile', trigger: 'blur' }
        ]
      },
      editFormRules: {
        email: [
          { required: true, message: 'Please input email', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
        ],
        mobile: [
          { required: true, message: 'Please input mobile', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('get users error!')
      this.userList = res.data.users
      this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    async userStateChange(userinfo) {
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('Update user status failed!')
      }
      this.$message.success('Update user status successfully!')
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) return this.$message.error('Add user failed!')
        this.$message.success('Add user success!')
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    async editUser(userId) {
      const { data: res } = await this.$http.get(`users/${userId}`)
      if (res.meta.status !== 200) return this.$message.error('Get user failed!')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    submitEditUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('users/' + this.editForm.id, this.editForm)
        if (res.meta.status !== 200) return this.$message.error('Edit user failed!')
        this.$message.success('Edit user success!')
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    async deleteUser(userId) {
      const confirmRes = await this.$confirm('Do you want to remove this user?', 'Warning',
        {
          confirmButtonText: 'Yes',
          cancelButtonText: 'No',
          type: 'warning'
        }).catch(err => err)
      if (confirmRes === 'confirm') {
        const { data: res } = await this.$http.delete('users/' + userId)
        if (res.meta.status !== 200) return this.$message.error('Delete user failed!')
        this.$message.success('Delete user success!')
        this.getUserList()
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
