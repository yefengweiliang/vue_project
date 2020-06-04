<template>
  <!--面包屑导航-->
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>人员管理</el-breadcrumb-item>
      <el-breadcrumb-item>人员信息</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!--搜索与添加区-->
          <el-input placeholder="请输入内容" v-model="queryInfo.userName" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区 -->
      <el-table :data="userlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="userName"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="性别" prop="gender">
          <template slot-scope="scope">
            <div v-if="scope.row.gender === 0 ">女</div>
            <div v-if="scope.row.gender === 1 ">男</div>
          </template>
        </el-table-column>
        <el-table-column label="昵称" prop="nickName"></el-table-column>
        <el-table-column label="电话" prop="phone"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.state" @change="userStateChane(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="230px">
          <template slot-scope="scope">
            <!--修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            ></el-button>
            <!--删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeUserById(scope.row.id)"
            ></el-button>
            <!--分配权限 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="this.queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="this.queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加用户 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="30%" @close="addDialogClosed">
      <!--内容主题区-->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="用户名称" prop="userName">
          <el-input v-model="addForm.userName"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="gender">
          <el-input v-model="addForm.gender"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="nickName">
          <el-input v-model="addForm.nickName"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="phone">
          <el-input v-model="addForm.phone"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区域-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="30%" @close="editDialogClosed">
      <!--内容主题区-->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="用户名称">
          <el-input v-model="editForm.userName" disabled></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="editForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="性别" prop="gender">
          <el-input v-model="editForm.gender"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="nickName">
          <el-input v-model="editForm.nickName"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="phone">
          <el-input v-model="editForm.phone"></el-input>
        </el-form-item>
      </el-form>
      <!--底部区域-->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    //验证邮箱的规则
    var checkEmail = (rule, value, callback) => {
      const mailReg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/
      if (mailReg.test(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的邮箱格式'))
      }
    }
    //验证手机号的规则
    var checkMobile = (rule, value, callback) => {
      const mobileReg = /^1[3456789]\d{9}$/
      if (mobileReg.test(value)) {
        callback()
      } else {
        callback(new Error('请输入正确的手机号'))
      }
    }
    return {
      queryInfo: {
        userName: '',
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      userlist: [],
      total: 0,
      //控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      //修改对话框的显示和隐藏
      editDialogVisible: false,
      //添加用户的表单数据
      addForm: {
        userName: '',
        password: '',
        email: '',
        gender: '',
        nickName: '',
        phone: ''
      },
      //查询到的用户信息对象
      editForm: {},
      //修改表单验证规则对象
      editFormRules: {
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 16,
            message: '密码的长度在6~16个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: checkEmail
          }
        ],
        gender: [
          {
            required: true,
            message: '请输入性别',
            trigger: 'blur'
          }
        ],
        nickName: [
          {
            required: true,
            message: '请输入昵称',
            trigger: 'blur'
          }
        ],
        phone: [
          {
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          },
          {
            validator: checkMobile
          }
        ]
      },
      //添加表单验证的规则对象
      addFormRules: {
        userName: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 10,
            message: '用户名称的长度在3~10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 16,
            message: '密码的长度在6~16个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          },
          {
            validator: checkEmail
          }
        ],
        gender: [
          {
            required: true,
            message: '请输入性别',
            trigger: 'blur'
          }
        ],
        nickName: [
          {
            required: true,
            message: '请输入昵称',
            trigger: 'blur'
          }
        ],
        phone: [
          {
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          },
          {
            validator: checkMobile
          }
        ]
      }
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      //获取用户列表的参数对象
      var params = new URLSearchParams()
      params.append('page', this.queryInfo.pagenum)
      params.append('rows', this.queryInfo.pagesize)
      params.append('userName', this.queryInfo.userName)
      const { data: res } = await this.$http.post('getUser', params)
      console.log(res)
      this.userlist = res.respVo[0].rows
      this.total = res.respVo[0].total
    },
    //监听pagesize改变事件
    handleSizeChange(newSize) {
      console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    //监听页码值改变事件
    handleCurrentChange(newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    //监听swich开关的改变事件
    async userStateChane(userInfo) {
      console.log(userInfo)
      var params = new URLSearchParams()
      params.append('id', userInfo.id)
      params.append('state', userInfo.state)
      const { data: res } = await this.$http.post('updateUser', params)
      if (res.respCode !== 100) {
        userInfo.state = !userInfo.state
        this.$message.error('更新用户状态失败！')
      } else {
        this.$message.success('更新用户状态成功！')
      }
    },
    //监听添加用户表单关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //点击按钮天机新用户
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        //添加用户的请求
        var params = new URLSearchParams()
        params.append('userName', this.addForm.userName)
        params.append('password', this.addForm.password)
        params.append('email', this.addForm.email)
        if (this.addForm.gender === '男') {
          this.addForm.gender = 1
        } else {
          this.addForm.gender = 0
        }
        params.append('gender', this.addForm.gender)
        params.append('nickName', this.addForm.nickName)
        params.append('phone', this.addForm.phone)
        const { data: res } = await this.$http.post('addUser', params)
        if (res.respCode !== 100) {
          this.$message.error('添加用户失败！')
        } else {
          this.$message.success('添加用户成功！')
        }
        this.addDialogVisible = false
        this.getUserList()
      })
    },
    //展示用户编辑用户的对话框
    async showEditDialog(id) {
      this.editDialogVisible = true
      var params = new URLSearchParams()
      params.append('id', id)
      const { data: res } = await this.$http.post('getUser', params)
      if (res.respCode !== 100) {
        this.$message.error('获取修改用户信息失败！')
      } else {
        this.editForm = res.respVo[0].rows[0]
      }
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //修改用户信息并提交
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        //修改用户请求
        var params = new URLSearchParams()
        params.append('id', this.editForm.id)
        params.append('userName', this.editForm.userName)
        params.append('password', this.editForm.password)
        params.append('email', this.editForm.email)
        if (this.editForm.gender === '男') {
          this.editForm.gender = 1
        } else {
          this.editForm.gender = 0
        }
        params.append('gender', this.editForm.gender)
        params.append('nickName', this.editForm.nickName)
        params.append('phone', this.editForm.phone)
        const { data: res } = await this.$http.post('updateUser', params)
        if (res.respCode !== 100) {
          this.$message.error('更新用户信息失败！')
        } else {
          //关闭对话框
          this.editDialogVisible = false
          //刷新数据列表
          this.getUserList()
          //提示修改成功
          this.$message.success('更新用户信息成功！')
        }
      })
    },
    //根据id删除用户信息
    async removeUserById(id) {
      console.log(id)
      //询问用户是否删除数据
      const res = await this.$confirm(
        '此操作将永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      //如果用户确认删除则返回字符串confirm
      //如果用户取消删除则返回字符串cancel
      // console.log(res)
      if (res !== 'confirm') {
        return this.$message.info('已取消删除')
      } else {
        // console.log('确认了删除')
        //删除用户
        var params = new URLSearchParams()
        params.append('id', id)
        const { data: res } = await this.$http.post('deleteUser', params)
        if (res.respCode !== 100) {
          this.$message.error('用户信息删除失败！')
        } else {
          this.$message.success('用户信息删除成功！')
          this.getUserList()
        }
      }
    }
  }
}
</script>
<style lang="less" scoped>
</style>
