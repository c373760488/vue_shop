<template>
  <div class="login_container">
    <div class="login_box">
<!--      头像区-->
      <div class="avatar_box">
        <img src="../assets/avatar.jpg" alt="">
      </div>
<!--      登录表单区-->
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
<!--        用户名-->
        <el-form-item label="" prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
<!--        密码-->
        <el-form-item label="" prop="password" >
          <el-input v-model="loginForm.password" type="password" prefix-icon="iconfont icon-3702mima"></el-input>
        </el-form-item>
<!--        按钮区-->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button @click="resetloginForm" type="info">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 这是登录表单的数据绑定对象
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 18,
            message: '长度在 3 到 18 个字符',
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
            max: 18,
            message: '长度在 6 到 18 个字符',
            trigger: 'blur'
          }
        ]
      }
    }
  },
  methods: {
    // 点击重置按钮，重置登录表单
    resetloginForm () {
      console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      this.$refs.loginFormRef.validate(async (valid) => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginForm)
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')
        // 将登录成功之后的token，保存到客户端的sessionStorage（打开浏览器期间生效）中。（localstorage关闭也还在）
        // 将项目中除了登录之外得其他API接口，必须在登录之后才能访问
        // token值应在当前网站打开期间生效，所以讲token保存在sessionStorage
        window.sessionStorage.setItem('token', res.data.token)
        // 通过编程时导航跳转到后台主页。路由地址是/home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container{
  background:url("../assets/images/background.jpg");
  height: 100%;
}
.login_box{
  width: 450px;
  height: 300px;
  background-color: white;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
  .avatar_box{
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%,-50%);
    background-color: #fff;
    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}
.btns {
  display: flex;
  justify-content: flex-end;
}
.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding:0 20px;
  box-sizing: border-box;
}
</style>
