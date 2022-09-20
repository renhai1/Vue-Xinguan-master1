<template>
  <div>
    <el-card class="box-card">
      <el-form ref="loginForm" :model="userLoginForm" :rules="loginFormRules">
        <div style="text-align: center">
          <i
            class="icon iconfont icon-apphoutaiguanli"
            style="font-size: 100px; color: #e9232c"
          ></i>
        </div>
        <h2 style="text-align: center">抗疫物资溯源管控平台</h2>
        <el-form-item prop="username">
          <el-input
            v-model="userLoginForm.username"
            placeholder="请输入用户名"
            prefix-icon="icon iconfont icon-icon"
          ></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input
            v-model="userLoginForm.password"
            placeholder="请输入密码"
            type="password"
          >
            <span slot="prefix">
              <i class="icon iconfont icon-mima"></i>
            </span>
            <span slot="suffix">
              <i class="icon iconfont icon-icon-test1"></i>
            </span>
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="danger" style="width: 45%" @click="login"
            >登录</el-button
          >
          <el-button type="danger" style="width: 45%" @click="reset"
            >重置</el-button
          >
        </el-form-item>
      </el-form>
    </el-card>
    <img
      src="../assets/images/login_center_bg.png"
      alt=""
      class="login-bg-img"
    />
  </div>
</template>

<script>
export default {
  data() {
    return {
      userLoginForm: {
        username: "admin",
        password: "123456",
      },

      //验证规则
      loginFormRules: {
        username: [
          { required: true, message: "请输入登录的用户名", trigger: "blur" },
          {
            min: 3,
            max: 15,
            message: "长度在 3 到 15 个字符",
            trigger: "blur",
          },
        ],
        password: [
          { required: true, message: "请输入登录的密码", trigger: "blur" },
          {
            min: 3,
            max: 15,
            message: "长度在 3 到 15 个字符",
            trigger: "blur",
          },
        ],
      },
    };
  },

  methods: {
    //登入提交
    login: function () {
      this.$refs.loginForm.validate(async (valid) => {
        if (!valid) {
          return;
        } else {
          //发起登入请求
          const { data: res } = await this.$http.post(
            "user/login?username=" +
              this.userLoginForm.username +
              "&password=" +
              this.userLoginForm.password
          );
          if (res.code == 200) {
            this.$message({
              title: "登入成功",
              message: "欢迎你进入系统",
              type: "success",
            });
            //保存token
            localStorage.setItem("JWT_TOKEN", res.data);
            this.getUserInfo();
          } else {
            this.$message.error({
              title: "登入失败",
              message: res.msg,
              type: "error",
            });
          }
        }
      });
    },
    //重置表单
    reset: function () {
      this.$refs.loginForm.resetFields();
    },
    /**
      获取用户信息
     */
    async getUserInfo() {
      const { data: res } = await this.$http.get("user/info");
      if (res.code !== 200)
        return this.$message.error("获取用户信息失败:" + res.msg);
      this.userInfo = res.data;
      //保存用户
      this.$store.commit("setUserInfo", res.data);
      //跳转到home
      this.$router.push("/home");
    },
    close() {
      this.isShow = false;
    },
  },
};
</script>

<style scoped lang="less">
.box-card {
  width: 360px;
  position: absolute;
  left: 50%;
  right: 50%;
  margin-left: -180px;
  border-top: 10px solid #e9232c;
  margin-top: 120px;
}
.login-bg-img {
  background: #e9232c;
  margin-top: 205px;
  width: auto;
  height: auto;
  max-height: 100%;
  max-width: 100%;
}
</style>


